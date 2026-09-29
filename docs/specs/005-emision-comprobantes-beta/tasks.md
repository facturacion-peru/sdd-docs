# Tareas 005: Emisión y consulta de comprobantes en beta

**Estado:** Aprobado (2026-09-29) · **Fecha:** 2026-09-29 · **Plan:** [plan.md](./plan.md) (aprobado)

> Mismas convenciones que las specs anteriores: tareas pequeñas con su verificación; `[P]` = en paralelo con la anterior; 🔒 = confirmación del usuario al ejecutarla.
> **Pruebas primero:** las tareas «Escribir pruebas» terminan en rojo y la siguiente las pone en verde.
> Antes de cada commit se verifica con el **código de salida** de las suites de la API (`php artisan test`, `composer test:pgsql`, `vendor/bin/pint --test`) o del frontend.

## Fase 1 · Preparación y *spike*

- [ ] **T001** Crear la rama `005-emision-comprobantes` desde `main` en la API y en el frontend. — Verificación: rama en ambos.
- [ ] **T002** *Spike* contra SUNAT beta real (R-1, R-3). Un script en `tests/Beta/` (no es código de producción) arma con Greenter los casos de CE-001 con la regla de cálculo del plan y los envía a beta:
  - Gravado, exonerado, inafecto y mixto.
  - Descuento por línea y redondeos límite (p. ej. 3 × 0.33, importes que no se dividen exacto entre 1.18).
  - Boleta a «Cliente varios» (tipo `0`, número `-`), boleta con DNI y factura con RUC.
  - Reenvío del mismo comprobante (respuesta «duplicado») y si existe consulta de CDR en beta.
  - Verificación: resultado por caso anotado en `plan.md` (sección Cálculo y R-3); si algún caso falla, la regla corregida queda en el plan antes de seguir.

## Fase 2 · Fundamentos

- [ ] **T010** Enums `SalesDocumentStatus` (con etiquetas), `CustomerDocumentType` (`1`, `4`, `6`, con validación de formato) y `SubmissionResult`, con pruebas. — Verificación: pruebas en verde.
- [ ] **T011** Migraciones `customers`, `sales_documents`, `sales_document_lines` y `sunat_submissions`, según el plan. — Verificación: `test` y `test:pgsql` en verde; `php artisan migrate` en local.
- [ ] **T012** Modelos `Customer`, `SalesDocument`, `SalesDocumentLine` y `SunatSubmission` con `BelongsToCompany`, casts decimales y *factories*. `SunatSubmission` es inmutable. — Verificación: pruebas de modelo en verde.

## Fase 3 · Clientes (HU-5)

- [ ] **T020** Escribir pruebas `Sales/CustomersTest`:
  - HU-5.1: búsqueda por documento o nombre, solo de la propia empresa.
  - HU-5.2: alta por cualquier rol; DNI de 8 dígitos, RUC con dígito verificador válido, carné de extranjería; documento duplicado rechazado.
  - HU-5.3: solo el administrador edita.
  - No hay ruta para borrar.
  - Auditoría del alta y la edición.
  - Verificación: en rojo.
- [ ] **T021** Implementar `CustomerService`, `GET/POST /customers` y `PATCH /customers/{customer}`, con requests y recurso. — Verificación: T020 en verde en ambas bases.

## Fase 4 · Cálculo y XML (HU-3)

- [ ] **T030** Escribir pruebas `Unit/Sunat/TaxCalculatorTest`: la tabla de casos de T002 con sus importes esperados, totales por afectación, determinismo (HU-3.4) y que el total coincide con el de `TicketService` para las mismas líneas. — Verificación: en rojo.
- [ ] **T031** Implementar `TaxCalculator`. — Verificación: T030 en verde.
- [ ] **T032** [P] Escribir e implementar `Unit/Sunat/AmountInWordsTest` y `AmountInWords` (caracterización del número en letras heredado: 0, 1, 21, 100, 101, 1 000, 1 000 000, céntimos). — Verificación: en verde.
- [ ] **T033** Escribir pruebas `Unit/Sunat/UblBuilderTest`: del comprobante guardado al `Invoice` de Greenter y al XML firmado con un certificado de prueba. Comprobar emisor, cliente o «Cliente varios», líneas con código de afectación y tributo, descuento de línea, totales, leyenda 1000, forma de pago contado y `hash`. — Verificación: en rojo.
- [ ] **T034** Implementar `UblBuilder` y el firmado con el certificado vigente descifrado en memoria. — Verificación: T033 en verde.

## Fase 5 · Emisión (HU-1, HU-2)

- [ ] **T040** Interfaz `SunatSender` con `GreenterSender` (endpoint beta de `SunatEndpoints`, credenciales de `config('services.sunat')`, tiempo límite) y `FakeSunatSender` para pruebas (acepta, observa, rechaza, no responde, «duplicado»). — Verificación: prueba unitaria del *fake*; `GreenterSender` se prueba en T090.
- [ ] **T041** Escribir pruebas `Sales/IssueSalesDocumentTest` (con `FakeSunatSender`):
  - HU-1.1: boleta con gravados y exonerados: correlativo, stock descontado con el comprobante como origen, XML firmado y estado tras el primer envío.
  - HU-1.2: «Cliente varios» hasta S/ 700; por encima, 422 sin documento.
  - HU-2.1–2.3: factura con RUC; factura sin cliente o sin RUC → 422 que sugiere boleta; Nuevo RUS → 422.
  - RF-002: configuración no validada, serie inactiva o de otro tipo → 422; varias series activas sin `series_id` → 422 con las opciones.
  - RF-005: sin stock o con la firma fallando → nada guardado y el correlativo intacto.
  - RF-006: copia de emisor, cliente y productos.
  - Idempotencia: la misma clave devuelve el mismo comprobante (200).
  - HU-4.1: SUNAT sin responder → 201 con `pending`.
  - Auditoría `sales_document.issued`.
  - Verificación: en rojo.
- [ ] **T042** Implementar `SalesDocumentService::issue()`, `POST /sales-documents`, request y recurso. — Verificación: T041 en verde en ambas bases.
- [ ] **T043** [P] Escribir e implementar `Integration/Sales/ConcurrentIssueTest` (solo PostgreSQL): varios procesos emiten en la misma serie → números consecutivos sin repetir. Comprobar con una mutación (sin `nextNumber()` bloqueante) que la prueba falla. — Verificación: en verde, y en rojo con la mutación.

## Fase 6 · Envío y reintentos (HU-4)

- [ ] **T050** Escribir pruebas `Sales/SunatDispatchTest`:
  - Aceptado (CDR guardado), observado (notas), rechazado (código y motivo, sin reintento).
  - Sin respuesta → `pending` con `next_attempt_at` según la espera creciente del plan; tras 24 h sigue pendiente sin más reintentos automáticos.
  - HU-4.5: «duplicado» → `accepted` con la nota de R-3 (o el estado real, si T002 encontró la consulta de CDR en beta).
  - HU-4.3: `POST /sales-documents/{id}/retry` reenvía; 409 con envío en curso; 422 si es definitivo.
  - `sunat:send-pending` toma los vencidos y los `sent` con *lease* caducado.
  - Cada intento queda en `sunat_submissions`; auditoría de cambios de estado y reintentos (RF-022).
  - Verificación: en rojo.
- [ ] **T051** Implementar `SunatDispatcher`, el comando `sunat:send-pending` en el *scheduler* (cada minuto), el primer intento tras el *commit* en `issue()` y la ruta de reintento. — Verificación: T050 en verde en ambas bases.
- [ ] **T052** [P] Escribir e implementar `Integration/Sales/ConcurrentDispatchTest` (solo PostgreSQL): reintento manual y comando a la vez → un solo envío. Comprobar con una mutación (sin *lease*) que la prueba falla. — Verificación: en verde, y en rojo con la mutación.
- [ ] **T053** Escribir e implementar `Integration/Sales/OutageRecoveryTest` (CE-002): N comprobantes con SUNAT caída; al volver, todos definitivos y exactamente un envío aceptado por comprobante. — Verificación: en verde.

## Fase 7 · Consulta y descargas (HU-6, RF-007, RF-021)

- [ ] **T060** Escribir pruebas `Sales/ListSalesDocumentsTest`: filtros por fecha, tipo, estado y cliente; el vendedor ve todos los de su empresa (A-32); detalle con líneas, totales por afectación, mensaje de SUNAT e intentos. — Verificación: en rojo.
- [ ] **T061** Implementar `GET /sales-documents` y `GET /sales-documents/{id}` con request de filtros y recursos. — Verificación: T060 en verde.
- [ ] **T062** Escribir pruebas `Sales/SalesDocumentPdfTest` y de descargas: PDF A4 y 80 mm (contenido, QR con los campos de SUNAT, marca «PRUEBAS — sin valor legal»), XML firmado y CDR (404 si no existe). — Verificación: en rojo.
- [ ] **T063** Implementar `DocumentPdf` con las plantillas heredadas reescritas sobre un DTO del comprobante, y las rutas `/pdf`, `/xml` y `/cdr`. Retirar de `resources/views/pdf` las plantillas que no se usen (notas, guías, 50 mm, A5). — Verificación: T062 en verde; PDF revisado a ojo.

## Fase 8 · Seguridad, aislamiento y contrato

- [ ] **T070** Añadir las rutas nuevas a `TenantRoutes.php` y el cliente y el comprobante de la empresa B a `CrossTenantAccessTest` (incluidas las descargas). — Verificación: `TenantRoutesCoverageTest` y `CrossTenantAccessTest` en verde.
- [ ] **T071** [P] Ampliar `NoSunatSecretsLeakTest` al flujo de emisión y envío: ni el PEM, ni la clave privada, ni la contraseña del certificado aparecen en respuestas, XML, logs ni auditoría. — Verificación: en verde, y en rojo con una mutación que registre el PEM en el log.
- [ ] **T072** [P] `Contract/SalesDocumentResourcesTest` (comprobante, línea, intento y cliente). — Verificación: en verde.
- [ ] **T073** `php artisan openapi:generate`. En `DevelopmentSeeder`, clientes demo (uno con DNI y otro con RUC) y, si SUNAT beta responde, una boleta y una factura demo emitidas. README de la API: emisión, estados, `schedule:run` y `schedule:work`. — Verificación: OpenAPI con las rutas nuevas; `migrate:fresh --seed` en local (autorizado).

## Fase 9 · Frontend

- [ ] **T080** `pnpm api:types`; tipos y API de comprobantes, clientes, estado SUNAT y series en `features/sales`, y descarga con token (`Blob`). — Verificación: `pnpm type-check`.
- [ ] **T081** `CustomerPicker`: búsqueda y alta rápida en un diálogo, con errores del documento. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T082** «Vender»: selector Ticket · Boleta · Factura con bloqueos y motivo (SUNAT no disponible, sin serie), selector de serie, cliente, aviso de S/ 700 y factura con RUC. Tras emitir, lleva al detalle con el resultado (aceptado, pendiente o rechazado). Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T083** «Ventas» con pestañas Tickets · Comprobantes (filtros, pendientes y rechazados resaltados) y detalle del comprobante (`EnvironmentBadge`, estado y mensaje de SUNAT, totales por afectación, descargas y «Reintentar»). Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T084** [P] `/clientes` para el administrador (lista y edición) y «Clientes» en «Más». Pruebas incluidas. — Verificación: pruebas en verde.

## Fase 10 · Cierre

- [ ] **T090** Correr `tests/Beta/SunatBetaTest` (grupo `sunat-beta`) contra SUNAT beta real: los casos de CE-001 aceptados. — Verificación: salida adjunta en `verificacion.md`.
- [ ] **T091** API: `php artisan test`, `composer test:pgsql` y `vendor/bin/pint --test` en verde (códigos de salida). — Verificación: salidas adjuntas.
- [ ] **T092** Frontend: `pnpm type-check`, `pnpm lint`, `pnpm test:unit --run` y `pnpm build-only` en verde. — Verificación: salidas adjuntas.
- [ ] **T093** Verificación de extremo a extremo por el proxy de Vite, con `schedule:work` corriendo:
  - Boleta a «Cliente varios», boleta con DNI de más de S/ 700 y factura, aceptadas por SUNAT beta real.
  - Descargas de PDF (A4 y 80 mm), XML y CDR.
  - Caída simulada (endpoint inalcanzable): pendiente, reintento automático al restablecer y reintento manual (CE-002).
  - Vendedor ve todos los comprobantes; otra empresa recibe 404.
  - Búsqueda de secretos en las respuestas.
  - Capturas a 360 px. Resultado en `verificacion.md`.
  - Verificación: `verificacion.md` con resultado por paso.
- [ ] **T094** Documentación: README de la API y del frontend, `AGENTS.md` y estado de la spec en `specs/README.md`. — Verificación: enlaces válidos.
- [ ] **T095** 🔒 Merge local de `005-emision-comprobantes` a `main` en ambos repositorios y commit de la raíz. — Verificación: suites en verde sobre `main`.

## Trazabilidad

| Spec | Tareas |
|---|---|
| HU-1, HU-2, RF-001–009 | T041–T042, T082 |
| HU-3, CE-001 | T002, T030–T034, T090 |
| HU-4, RF-010–013, CE-002 | T050–T053, T083, T093 |
| HU-5, RF-030–031 | T020–T021, T081, T084 |
| HU-6, RF-021 | T060–T061, T083 |
| RF-007, RF-020 | T062–T063, T083 |
| RF-014 | T012, T050 |
| RF-022 | T020, T041, T050 |
| Correlativo y envío concurrentes | T043, T052 |
| CE-003 | T093 (cronometraje pendiente del usuario) |
| Principio IV | T034, T071 |
| Principio VIII | T070 |
| Principio III | T072–T073, T080 |
