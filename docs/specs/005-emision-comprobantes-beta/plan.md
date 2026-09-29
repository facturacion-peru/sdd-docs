# Plan 005: Emisión y consulta de comprobantes en beta

**Estado:** Aprobado (2026-09-29) · **Fecha:** 2026-09-29 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

Sobre las specs 001 a 004 (aislamiento, auditoría, `bcmath`, `InventoryService`, series con `nextNumber()` bloqueante y certificado cifrado):

1. **Clientes** (`customers`) con documento único por empresa.
2. **Comprobantes** (`sales_documents` y sus líneas) emitidos por un `SalesDocumentService` con el mismo patrón que `TicketService`: una transacción con idempotencia, correlativo, copia de datos, cálculo exacto, descuento de stock y **XML firmado**. Si la firma falla, no queda nada, ni el número (RF-005).
3. El **cálculo** vive en una clase pura, `TaxCalculator` (`bcmath`), probada con una tabla de casos y **validada contra SUNAT beta** antes de construir lo demás (CE-001).
4. El **envío** se hace después de confirmar la transacción: un intento inmediato dentro de la misma petición, con tiempo límite. Si SUNAT no responde, el comprobante queda pendiente y un comando programado lo reenvía con espera creciente. Un *lease* en la fila evita envíos simultáneos (RF-012).
5. **XML firmado y CDR en la base de datos**, sin archivos en disco. El **PDF** (A4 y 80 mm, con QR y el distintivo de pruebas) se genera al descargarlo, con las plantillas heredadas adaptadas y cubiertas por pruebas.
6. En el frontend, «Vender» ofrece Ticket, Boleta o Factura con selector de cliente, y «Ventas» gana una pestaña «Comprobantes» con el detalle, las descargas y «Reintentar».

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos | API y frontend |
| Dependencias nuevas | Ninguna. Ya están instalados `greenter/lite` 5.1 (XML UBL 2.1, firma y envío SOAP), `dompdf` y `endroid/qr-code` |
| Código heredado | Se reincorpora **solo lo necesario**, reescrito sobre los modelos nuevos y con pruebas de caracterización: construcción del `Invoice` de Greenter, número en letras y plantillas A4/80 mm. Nada de `DocumentService` (2 263 líneas, con ramas rotas) |
| Reintentos | Comando `sunat:send-pending` en el *scheduler* de Laravel, cada minuto. Sin cola ni worker: basta con `schedule:run` en cron (en desarrollo, `php artisan schedule:work`). Más simple que cola + worker y suficiente para el volumen esperado (A-06) |
| Envío en pruebas | Interfaz `SunatSender` con dos implementaciones: `GreenterSender` (real) y `FakeSunatSender` (pruebas: acepta, observa, rechaza, no responde o responde «duplicado») |
| Credenciales de envío | En beta, las genéricas de `config('services.sunat')` (`{RUC}MODDATOS`), como en la 004 (A-31). El certificado vigente se descifra solo en memoria para firmar |

## Chequeo de constitución (v1.1.2)

| Principio | Cumple | Cómo / justificación |
|---|---|---|
| I. Spec primero | ✅ | Spec 1.0 aprobada; A-22, A-23 y A-32 a A-34 resueltas. |
| II. Corrección fiscal | ✅ ⚖️ | Cálculo solo en el servidor, con `bcmath` y probado caso por caso; aceptación real de SUNAT beta como criterio (CE-001); nunca producción. Los casos tributarios siguen pendientes de revisión con un contador (A-22). |
| III. Contrato de API | ⚠️ | FormRequests para todas las entradas, OpenAPI y tipos regenerados. Las respuestas se protegen con pruebas de contrato (excepción heredada). |
| IV. Seguridad | ✅ | El certificado solo se descifra en memoria; el XML y el CDR no contienen secretos; las descargas pasan por el aislamiento de empresa. Se amplía la prueba de no filtración de la 004 al flujo de emisión. |
| V. Simplicidad | ✅ | Sin cola ni worker; PDF generado al vuelo; sin resumen diario ni notas (A-22). |
| VI. Verificable | ✅ | Tabla de casos del cálculo, XML de caracterización, estados con el emisor simulado, concurrencia real de correlativo y de envío, y un grupo de pruebas contra SUNAT beta real. |
| VII. Personas reales | ✅ | La emisión sale desde «Vender», con el mismo carrito; mensajes de SUNAT en español; «PRUEBAS — sin valor legal» en pantalla y en el PDF. |
| VIII. Aislamiento y auditoría | ✅ | Modelos con `BelongsToCompany`, rutas en `TenantRoutes.php`, auditoría de emisión, reintentos y cambios de estado. |

## Modelo de datos

| Tabla | Columnas relevantes | Restricciones |
|---|---|---|
| `customers` | `company_id`, `document_type` (catálogo 06: `1` DNI, `4` carné de extranjería, `6` RUC), `document_number`, `name`, `address` nulo, `created_by` | único (`company_id`, `document_type`, `document_number`) |
| `sales_documents` | `company_id`, `series_id`, `document_type` (`01`/`03`), `series_code`, `number`, `issued_at`, `seller_id`, `payment_method`, `currency` (`PEN`), `environment` (`beta`); copia del emisor (RUC, razón social, dirección) y del cliente (`customer_id` nulo, tipo, número, nombre, dirección); `op_gravadas`, `op_exoneradas`, `op_inafectas`, `igv`, `discount_total`, `total`; `status`, `sunat_code`, `sunat_message`, `sunat_notes` (json), `xml` (firmado), `hash`, `cdr` (zip en base64), `attempts`, `next_attempt_at`, `locked_until`, `idempotency_key` | único (`company_id`, `series_id`, `number`) y (`company_id`, `idempotency_key`) |
| `sales_document_lines` | `company_id`, `sales_document_id`, `product_id`, `position`, copia de código, nombre y unidad, `igv_affectation` (`10`/`20`/`30`), `quantity`, `unit_price` (con IGV), `unit_value` (sin IGV, 10 decimales), `discount`, `base_amount`, `igv`, `amount` | — |
| `sunat_submissions` | `company_id`, `sales_document_id`, `trigger` (`issue`/`scheduled`/`manual`), `started_at`, `duration_ms`, `result` (`accepted`/`observed`/`rejected`/`unreachable`/`error`), `code`, `message`, `user_id` nulo | inmutable (RF-013) |

**Estados** (RF-010):

| Estado | Significado | Transiciones |
|---|---|---|
| `pending` | Guardado y firmado, sin respuesta definitiva de SUNAT | → `sent` al empezar un envío |
| `sent` | Envío en curso (con *lease*) | → `accepted`, `observed`, `rejected`, o vuelta a `pending` si SUNAT no responde |
| `accepted` / `observed` / `rejected` | Definitivos | Ninguna (RF-014) |

## Cálculo (`App\Sunat\TaxCalculator`)

El precio del catálogo incluye IGV (A-16) y el importe de cada línea se calcula igual que en el ticket: `importe = redondeo(cantidad × precio, 2) − descuento`. Así el total que ve el vendedor en «Vender» no cambia (HU-3.4). A partir del importe de la línea:

| Afectación | Base (`base_amount`) | IGV | Tributo |
|---|---|---|---|
| Gravado (`10`) | `redondeo(importe / 1.18, 2)` | `importe − base` | `1000` IGV |
| Exonerado (`20`) | `importe` | 0 | `9997` EXO |
| Inafecto (`30`) | `importe` | 0 | `9998` INA |

- `unit_value = precio / 1.18` a 10 decimales. El descuento se informa como descuento de línea (código `00`, afecta la base), por su valor sin IGV.
- **Totales:** `op_*` = suma de bases por afectación; `igv` = suma de IGV de las líneas; `total` = suma de importes. Cuadran por construcción y quedan dentro de las tolerancias de ±1 sol de las validaciones de SUNAT.
- **Leyenda `1000`** («SON … CON xx/100 SOLES»): se reincorpora el número en letras de `legacy/PdfService` con su propia tabla de pruebas.
- **Riesgo principal (II):** que SUNAT beta rechace alguna combinación. Por eso la **primera tarea técnica es un *spike*** que envía a beta los casos de CE-001 (gravado, exonerado, inafecto, mixto, descuentos y redondeos límite) con este cálculo, antes de construir el resto. Si algún caso falla, se ajusta la regla y se registra aquí.

## Servicios

| Pieza | Responsabilidad | Spec |
|---|---|---|
| `CustomerService` | Alta (cualquier rol) y edición (administrador); valida DNI de 8 dígitos, RUC de 11 con dígito verificador (regla `Ruc` de la 001) y carné de extranjería | HU-5, RF-030/031 |
| `TaxCalculator` | Importes por línea y totales, puro y determinista | HU-3 |
| `UblBuilder` | Del comprobante guardado al `Invoice` de Greenter (emisor, cliente o «Cliente varios», líneas, leyenda, forma de pago contado) | RF-001 |
| `SalesDocumentService::issue()` | Transacción: idempotencia → comprobaciones (configuración validada, serie activa del tipo, factura solo con RUC y no en Nuevo RUS, boleta de más de S/ 700 con documento) → `nextNumber()` → copia de datos → cálculo → `InventoryService::consume()` con el comprobante como origen → XML firmado y `hash` → auditoría | HU-1, HU-2, RF-002–009 |
| `SunatDispatcher::send()` | Toma el *lease* (`UPDATE … WHERE locked_until IS NULL OR locked_until < now()`), envía, clasifica la respuesta, guarda el CDR y el intento, y audita el cambio de estado | HU-4, RF-011–013 |
| `SunatSender` | Interfaz del envío; `GreenterSender` usa `See` de Greenter con el endpoint beta de `SunatEndpoints` | — |
| `sunat:send-pending` | Cada minuto: pendientes con `next_attempt_at` vencido, y `sent` con *lease* caducado (un proceso que murió a mitad de envío) | HU-4.2 |
| `DocumentPdf` | PDF A4 y 80 mm con Dompdf: datos del comprobante, QR y marca «PRUEBAS — sin valor legal» | RF-007, RF-020 |

**Clasificación de la respuesta de SUNAT:**

| Respuesta | Estado | ¿Reintenta? |
|---|---|---|
| CDR con código 0 sin observaciones | `accepted` | — |
| CDR con observaciones (códigos 4000 o más) | `observed`, con las notas | — |
| Error de 2000 a 3999, o excepción de 1000 a 1999 | `rejected`, con código y motivo | No |
| Tiempo agotado, error de red o de HTTP, o servicio de SUNAT no disponible (códigos 0100–0199) | `pending` | Sí: 1, 2, 5, 10 y 30 min, luego cada hora durante 24 h. Después sigue pendiente para el reintento manual y se avisa en la lista |
| «Registrado previamente» (1033) | Ver riesgo R-3 | — |

- **«Cliente varios»:** boleta de S/ 700 o menos sin cliente → tipo de documento `0` y número `-`, «CLIENTES VARIOS» (se confirma en el *spike*).
- **Boletas:** se envían una por una con `sendBill`, igual que las facturas; el resumen diario queda fuera (A-22).
- **Primer intento:** dentro de la petición de emisión, después del *commit* y con un tiempo límite de 15 s, para que el vendedor vea el resultado al momento. Si falla, la respuesta ya informa «pendiente de envío» (HU-4.1).

## Contrato de API (`/api/v1`, grupo de empresa)

| Método | Ruta | Rol | Descripción | Spec |
|---|---|---|---|---|
| GET | `/customers?search=` | Todos | Búsqueda por documento o nombre (paginada) | HU-5.1 |
| POST | `/customers` | Todos | Alta | HU-5.2 |
| PATCH | `/customers/{customer}` | Admin | Edición (el comprobante conserva su copia) | HU-5.3/5.4 |
| POST | `/sales-documents` | Todos | `{ idempotency_key, document_type, series_id?, customer_id?, payment_method, lines[] }` → 201 con el estado tras el primer intento; 200 si la clave ya existía | HU-1, HU-2 |
| GET | `/sales-documents` | Todos | Filtros: `from`, `to`, `document_type`, `status`, `customer` (paginado). Todos ven todos los de su empresa (A-32) | HU-6 |
| GET | `/sales-documents/{id}` | Todos | Detalle con líneas, totales, estado, mensaje y notas de SUNAT, e intentos | HU-1.3 |
| POST | `/sales-documents/{id}/retry` | Todos | Reintento inmediato; 409 si ya hay un envío en curso; 422 si el estado es definitivo | HU-4.3 |
| GET | `/sales-documents/{id}/pdf?format=a4\|80mm` · `/xml` · `/cdr` | Todos | Descargas (el CDR solo si existe) | RF-021 |

- `series_id` es opcional: si falta, se usa la única serie activa del tipo; si hay varias, es obligatorio (422 con las opciones).
- `InsufficientStock` responde 422 con `meta.available`, igual que en los tickets.

## Frontend

| Dónde | Cambio |
|---|---|
| `sales/views/NewSaleView` | Selector **Ticket · Boleta · Factura** (Boleta y Factura se desactivan con un motivo si la emisión SUNAT no está disponible o no hay serie activa); selector de serie si hay varias; `CustomerPicker` (búsqueda y alta rápida en un diálogo); aviso al pasar de S/ 700 en boleta sin cliente; la factura exige un cliente con RUC. La vista previa de importes no cambia |
| `sales/views/SalesView` | «Ventas» con pestañas **Tickets · Comprobantes**. La de comprobantes tiene filtros y resalta los pendientes y rechazados |
| `sales/views/SalesDocumentView` | Detalle con `EnvironmentBadge`, estado y mensaje de SUNAT, líneas y totales por afectación, descargas (PDF A4 y 80 mm, XML y CDR) y «Reintentar» si está pendiente |
| `sales/views/CustomersView` | `/clientes` (administrador, menú «Más»): lista y edición |

- **Todo va en la feature `sales`** (tickets, comprobantes y clientes son el mismo dominio de venta), para respetar la regla de que una feature no importa de otra. El estado de SUNAT y las series que necesita «Vender» se piden desde `sales/api.ts` a los endpoints de la 004, tipados localmente con lo mínimo.
- **Descargas:** `fetch` con el token y `Blob`, porque un enlace directo no lleva la cabecera `Authorization`.

## Estrategia de pruebas

**API** (SQLite y PostgreSQL):

| Escenario / requisito | Prueba |
|---|---|
| HU-3, CE-001 (cálculo) | `Unit/Sunat/TaxCalculatorTest`: tabla de casos, determinismo y coincidencia con la vista previa de tickets |
| Leyenda | `Unit/Sunat/AmountInWordsTest` (caracterización del heredado) |
| XML | `Unit/Sunat/UblBuilderTest`: estructura y totales del XML firmado con un certificado de prueba (caracterización) |
| HU-1, HU-2, RF-002–009 | `Sales/IssueSalesDocumentTest` (con `FakeSunatSender`): boleta, factura, Cliente varios, tope de S/ 700, factura sin RUC, Nuevo RUS, sin configuración, serie inactiva, sin stock (sin número consumido), firma que falla (sin número consumido) e idempotencia |
| HU-4, RF-010–013 | `Sales/SunatDispatchTest`: aceptado, observado, rechazado, sin respuesta con reintento y espera creciente, duplicado, reintento manual y *lease* caducado |
| HU-5 | `Sales/CustomersTest` |
| HU-6, A-32 | `Sales/ListSalesDocumentsTest` |
| Correlativo y envío concurrentes | `Integration/Sales/ConcurrentIssueTest` y `Integration/Sales/ConcurrentDispatchTest` (solo PostgreSQL): números consecutivos y un solo envío |
| CE-002 | `Integration/Sales/OutageRecoveryTest`: N comprobantes con SUNAT caída → al volver, todos definitivos y un envío aceptado por comprobante |
| CE-001 real | `tests/Beta/SunatBetaTest` (grupo `sunat-beta`, excluido por defecto): envía los casos a SUNAT beta real. Se corre a mano y en la verificación |
| PDF | `Sales/SalesDocumentPdfTest`: A4 y 80 mm, QR con los campos de SUNAT y marca de pruebas |
| Principio VIII / IV | Rutas en `TenantRoutes.php` y cliente y comprobante de B en `CrossTenantAccessTest`; ampliación de `NoSunatSecretsLeakTest` a la emisión |
| Contratos | `Contract/SalesDocumentResourcesTest` |

**Frontend (Vitest):** selector de tipo y sus bloqueos, `CustomerPicker` (búsqueda y alta), tope de S/ 700, factura sin RUC, emisión con resultado aceptado, pendiente y rechazado, pestañas de «Ventas», detalle con descargas y reintento, y clientes.

**Verificación manual:** recorrido de extremo a extremo con emisión real a SUNAT beta de boletas y facturas (CE-001), una caída simulada (CE-002), cronometraje de una boleta de 3 productos en el celular (CE-003) y capturas a 360 px.

## Riesgos

| # | Riesgo | Mitigación |
|---|---|---|
| R-1 | SUNAT beta rechaza alguna combinación del cálculo o de «Cliente varios» | *Spike* contra beta como primera tarea técnica, antes de construir el resto |
| R-2 | SUNAT beta es inestable o lenta | Primer intento con tiempo límite y cola de pendientes; la venta nunca se pierde |
| R-3 | **HU-4.5 («duplicado»)**: el servicio que consulta el CDR de un comprobante ya enviado solo existe en producción, no en beta (se confirma en el *spike*) | Si SUNAT responde 1033, el comprobante se marca `accepted` con la nota «SUNAT ya lo tenía registrado; CDR no disponible en beta», y el detalle lo muestra. En producción (spec futura) se consultará el CDR. **Es una desviación de HU-4.5 que requiere tu aprobación** |
| R-4 | Se pierde el *cron* del *scheduler* y los pendientes no se reenvían | El reintento manual siempre está disponible; README y verificación incluyen `schedule:run`; la lista resalta los pendientes con más de 1 h |
| R-5 | Plantillas heredadas acopladas a los modelos viejos | Se reescriben sobre un DTO del comprobante, cubiertas por `SalesDocumentPdfTest` |
| R-6 | Volumen de XML y CDR en la base de datos | Unos 15 KB por comprobante; con el volumen de A-06 es despreciable. La retención se decide en A-24 |

## Aprobación

Aprobado por el usuario el 2026-09-29, incluidos:

1. Reintentos con el *scheduler* (cron) en lugar de cola y worker.
2. Todo en la feature `sales`, con «Comprobantes» como pestaña de «Ventas» y «Clientes» en «Más».
3. **R-3:** si en beta no existe la consulta de CDR, el «duplicado» (1033) se marca `accepted` con la nota «SUNAT ya lo tenía registrado; CDR no disponible en beta».
