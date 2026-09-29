# Plan 003: Tickets de venta internos

**Estado:** Aprobado (2026-09-28) · **Fecha:** 2026-09-28 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

Sobre la 001 (aislamiento y auditoría) y la 002 (`InventoryService`):

1. Tablas nuevas `tickets`, `ticket_lines` y `ticket_sequences`. La numeración sin huecos sale de una fila por empresa, bloqueada durante la transacción de la venta: si la venta falla, el número no se consume.
2. Un `TicketService` hace todo en **una** transacción: numerar, crear líneas con los precios del catálogo, calcular importes con `bcmath` y descontar el stock con `InventoryService::consume()` (origen = el ticket). Si falta stock, no queda nada.
3. Los reintentos (RF-008) se resuelven con una **clave de idempotencia** que envía el frontend: la misma clave devuelve el ticket ya creado.
4. Anular revierte los movimientos de venta del ticket con `InventoryService::reverse()` y conserva el número.
5. En el frontend, una feature nueva `sales`: vender, ver/imprimir el ticket (80 mm con CSS de impresión) y consultar ventas. Aprovecho para reordenar la navegación, que a 360 px ya no cabía (observación de la 002).

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos | API y frontend |
| Dependencias nuevas | Ninguna |
| Base de datos | Tres tablas nuevas; ningún cambio en tablas existentes |
| Impresión | Vista del frontend con `@page { size: 80mm auto }` e impresión o PDF del navegador (A-18). Sin Bluetooth ni ESC/POS |
| Pruebas | SQLite y PostgreSQL; prueba de numeración concurrente en `tests/Integration` (solo PostgreSQL, `pcntl_fork`) |

## Chequeo de constitución (v1.1.2)

| Principio | Cumple | Cómo / justificación |
|---|---|---|
| I. Spec primero | ✅ | Spec 1.0 aprobada; aclaraciones A-17 a A-19, A-29 y A-30 resueltas. |
| II. Corrección fiscal | ✅ | El ticket no es comprobante: no genera XML ni va a SUNAT. Aun así, los importes se calculan solo en el servidor con decimales exactos (RF-007), y la afectación al IGV de cada línea se guarda para la futura conversión en comprobante. |
| III. Contrato de API | ⚠️ | OpenAPI y tipos regenerados; entradas por FormRequest (para que queden tipadas). Las respuestas se protegen con pruebas de contrato, como en las specs anteriores. |
| IV. Seguridad | ✅ | Validación en servidor; el precio nunca viene del cliente (RF-011); anular solo el administrador. |
| V. Simplicidad | ✅ | Sin caja, sin pagos mixtos, sin impresión térmica; se reutiliza `InventoryService`. |
| VI. Verificable | ✅ | Pruebas por escenario, de numeración concurrente, de idempotencia y de redondeo. |
| VII. Personas reales | ✅ | Pantalla de venta pensada para el celular: buscar, tocar para agregar, cobrar (CE-001, menos de 30 s). Leyenda de documento interno siempre visible. |
| VIII. Aislamiento y auditoría | ✅ | Modelos con `BelongsToCompany`; rutas en `TenantRoutes.php`; el vendedor solo ve sus tickets; se auditan emisión y anulación. |

## Modelo de datos

| Tabla | Columnas relevantes | Restricciones |
|---|---|---|
| `ticket_sequences` | `company_id` (PK), `last_number` | Una fila por empresa; se bloquea con `FOR UPDATE` al numerar |
| `tickets` | `company_id`, `number` (entero), `status` (`issued`/`voided`), `seller_id`, `customer_name` nulo, `customer_document` nulo, `payment_method` (`cash`/`card`/`yape_plin`/`transfer`), `subtotal`, `discount_total`, `total` decimal(12,2), `idempotency_key` (uuid), `issued_at`, `voided_at`, `voided_by`, `void_reason` | únicos (`company_id`, `number`) y (`company_id`, `idempotency_key`); índice (`company_id`, `issued_at`) |
| `ticket_lines` | `company_id`, `ticket_id`, `product_id`, copia de `product_code`, `product_name`, `unit` e `igv_affectation`, `quantity` decimal(14,3), `unit_price`, `gross_amount`, `discount`, `amount` decimal(12,2) | orden de línea |

- **Número visible:** `T-` + número con 6 dígitos (`T-000001`), como atributo derivado. No puede confundirse con una serie SUNAT (4 caracteres).
- **Cálculo por línea:** bruto = redondeo a 2 decimales (mitad hacia arriba) de cantidad × precio; importe = bruto − descuento; descuento ≤ bruto. Total = suma de importes. Se añaden a `App\Support\Decimal` las operaciones `mul` y `round`, con pruebas unitarias.
- **Copias** de código, nombre, unidad, precio y afectación: el ticket conserva lo vigente al vender (RF-009).

## `TicketService`

| Método | Qué hace | Spec |
|---|---|---|
| `issue(datos, User)` | Si la clave de idempotencia ya existe en la empresa, devuelve ese ticket. Si no, en una transacción: bloquea la secuencia, asigna el número, crea el ticket y sus líneas con precios del catálogo (rechaza productos inactivos o de otra empresa), calcula importes, descuenta el stock de los bienes con `consume()` (origen = ticket) y audita `ticket.issued`. | HU-1, RF-001 a 011 |
| `void(Ticket, motivo, User)` | Solo tickets emitidos. En una transacción: revierte cada movimiento de venta del ticket con `reverse()` (motivo `error`), marca anulado con quién, cuándo y por qué, y audita `ticket.voided`. | HU-3, RF-006 |

- **Cambio en la 002 (consistencia):** `InventoryService::reverse()` rechazará revertir a mano un movimiento cuyo origen es un ticket: se corrige anulando el ticket. Así ticket y stock no pueden quedar desalineados.
- **Stock insuficiente:** `InsufficientStock` deshace todo y la API responde 422 indicando el producto y el disponible (ya existe en la 002).

## Contrato de API (`/api/v1`, grupo de empresa)

| Método | Ruta | Rol | Descripción | Spec |
|---|---|---|---|---|
| POST | `/tickets` | Admin y vendedor | `{ idempotency_key, payment_method, customer_name?, customer_document?, lines: [{ product_id, quantity, discount? }] }`. 201 al crear; 200 si la clave ya existía | HU-1, RF-008 |
| GET | `/tickets` | Admin y vendedor | Paginado; filtros de fechas, estado, medio de pago y vendedor (solo el administrador). Incluye `totals` por medio de pago sin anulados. El vendedor solo recibe los suyos | HU-4 |
| GET | `/tickets/{ticket}` | Admin y vendedor | Detalle con líneas, `display_number` y `legal_notice`. El ticket de otro vendedor da 404 al vendedor | HU-2, HU-4.2 |
| POST | `/tickets/{ticket}/void` | Admin | `{ reason }` | HU-3 |

- `legal_notice` = «Documento interno — no es comprobante de pago» viaja en cada ticket, para que cualquier vista o impresión futura la tenga (CE-002).

## Frontend: feature `sales`

| Ruta | Rol | Pantalla |
|---|---|---|
| `/vender` | Admin y vendedor | Venta rápida: buscar producto (solo activos, con disponible), tocar para agregar, cantidad con +/−, descuento por línea opcional, medio de pago, cliente opcional plegado, total de **vista previa** y «Cobrar». La clave de idempotencia se genera al abrir la venta y se reutiliza en reintentos |
| `/ventas` | Admin y vendedor | Listado por fecha (hoy por defecto) con totales por medio de pago |
| `/ventas/:id` | Admin y vendedor | Ticket de 80 mm con la leyenda, botón «Imprimir» (impresión o PDF del navegador) y «Anular» (solo administrador) |

- **Navegación:** principal con Inicio, Vender, Ventas y Productos; para el administrador, un menú «Más» con Alertas, Usuarios, Auditoría y Mi empresa. Así cabe a 360 px.
- **Stock insuficiente:** se marca la línea con el disponible que devuelve la API.
- **Importes:** la vista previa del total usa la misma regla de redondeo, pero el valor guardado y mostrado en el ticket es el del servidor.

## Estrategia de pruebas

**API** (SQLite y PostgreSQL):

| Escenario / requisito | Prueba |
|---|---|
| HU-1.1–1.5, RF-001, RF-009, RF-011, RF-013 | `Sales/IssueTicketTest` |
| Redondeo y descuentos | `Unit/Support/DecimalTest` (mul, round) y casos de importe en `IssueTicketTest` |
| RF-008 idempotencia | `Sales/TicketIdempotencyTest` |
| HU-1.3, RF-003 (stock insuficiente, nada queda) | `Sales/TicketStockTest` |
| HU-2 (`legal_notice`, sin «boleta»/«factura» en la respuesta) | `Sales/TicketContentTest` |
| HU-3, RF-006 (anular, stock repuesto, número conservado, no dos veces, solo admin; bloqueo de reversión manual) | `Sales/VoidTicketTest` |
| HU-4, RF-012 (filtros, totales sin anulados, vendedor solo lo suyo) | `Sales/ListTicketsTest` |
| CE-003 numeración concurrente | `Integration/Sales/ConcurrentTicketNumberingTest` (solo PostgreSQL) |
| Aislamiento | Rutas en `TenantRoutes.php`; datos de B (ticket) en `CrossTenantAccessTest` |
| Contratos | `Contract/TicketResourceTest` |

**Frontend (Vitest):** venta (agregar, cantidades, descuento, vista previa, clave de idempotencia reutilizada en reintento, stock insuficiente), ticket (leyenda siempre visible, anular solo administrador), listado (filtros y totales), navegación con menú «Más».

**Verificación manual:** recorrido de extremo a extremo por el proxy de Vite, capturas a 360 px (incluida la vista de impresión) y cronometrar una venta de 3 productos (CE-001).

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Números repetidos o con huecos | Secuencia por empresa bloqueada en la misma transacción + índice único + prueba concurrente |
| Doble venta por reintento | Clave de idempotencia con índice único |
| Ticket y stock desalineados | Una sola transacción; anular revierte; se bloquea la reversión manual de ventas de tickets |
| Confundir el ticket con un comprobante | Leyenda en el recurso y en la vista, número `T-` sin formato de serie, pruebas que buscan «boleta» y «factura» |
| Diferencias de redondeo entre vista previa y servidor | La misma regla en ambos lados; el ticket siempre muestra el total del servidor |

## Aprobación

Aprobado por el usuario el 2026-09-28, incluidos el bloqueo de la reversión manual de ventas de tickets (cambio en la 002) y la navegación con menú «Más».
