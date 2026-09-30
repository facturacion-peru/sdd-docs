# Plan 007: Anulación y corrección de comprobantes

**Estado:** Borrador · **Fecha:** 2026-09-30 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

1. **La nota de crédito es un comprobante más.** Va en la misma tabla `sales_documents` con tipo `07`, una referencia al comprobante que modifica y su motivo. Así reutiliza todo lo de la 005: correlativo bloqueado, cálculo, XML firmado, envío, reintentos, *lease*, CDR, PDF, listado y aislamiento.
2. **Cantidades devueltas por línea:** cada línea de la nota apunta a la línea original. Lo que queda por devolver se calcula sumando las notas que no fueron rechazadas, con la fila del comprobante original **bloqueada** mientras se emite la nota. Dos devoluciones simultáneas no pueden pasarse del total (RF-005).
3. **Stock:** la reposición se hace **al emitir la nota**, dentro de su transacción, con movimientos `return` hacia los lotes de los que salió la venta (A-40). Si SUNAT rechaza la nota, esos movimientos se revierten.
4. **Estado de corrección del comprobante original** (vigente, devuelto parcialmente, devuelto totalmente, anulado): se actualiza cuando SUNAT **acepta** la nota.
5. **Rechazados (A-42):** nuevo estado interno `discarded`. El administrador descarta con un motivo, se revierten las ventas del comprobante y el número queda usado.
6. **Primero, un *spike*** contra SUNAT beta: notas de anulación y de devolución parcial sobre boletas y facturas, con series `BC01` y `FC01`.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos | API y frontend |
| Dependencias nuevas | Ninguna: Greenter ya trae `Note` (tipo 07, motivo, documento afectado) |
| Migraciones | Columnas nuevas en `sales_documents` y `sales_document_lines`; `series.document_type` admite `07` |
| Series de notas | Tipo `07`, código con la letra del comprobante que modifican (`FC01` para facturas, `BC01` para boletas). Se crean en «Series» (spec 004); el Nuevo RUS no puede crear series `F…` |

## Chequeo de constitución (v1.1.2)

| Principio | Cumple | Cómo / justificación |
|---|---|---|
| I. Spec primero | ✅ | Spec 1.0 aprobada; A-39 a A-42 resueltas. |
| II. Corrección fiscal | ✅ ⚖️ | Importes de la nota con las reglas de la 005 y prorrateo exacto del descuento; nunca más de lo emitido; aceptación en SUNAT beta como criterio (CE-001). Motivos pendientes de revisión contable (A-39). |
| III. Contrato de API | ⚠️ | FormRequests para todas las entradas; respuestas con pruebas de contrato (excepción heredada). |
| IV. Seguridad | ✅ | Mismo firmado y envío que la 005; el descarte solo para el administrador; se amplían las pruebas de aislamiento y de no filtración. |
| V. Simplicidad | ✅ | Una tabla para comprobantes y notas; sin baja ni resumen diario (A-39). |
| VI. Verificable | ✅ | Pruebas por escenario, concurrencia real de devoluciones, consistencia de stock y saldos (CE-002) y *spike* y prueba contra SUNAT beta. |
| VII. Personas reales | ✅ | Devolución desde el detalle del comprobante, en pocos toques (CE-003). |
| VIII. Aislamiento y auditoría | ✅ | Rutas en `TenantRoutes.php`; se auditan notas, descartes y reposiciones de stock. |

## Modelo de datos

| Tabla | Cambios | Restricciones |
|---|---|---|
| `sales_documents` | `document_type` admite `07`. Nuevas: `reference_document_id` (FK a sí misma, solo notas), `note_reason_code` (`01`/`06`/`07`), `note_reason` (texto), `restock` (booleano), `correction_status` (`none`/`partially_returned`/`fully_returned`/`voided`), `discarded_at`, `discarded_by`, `discard_reason` | Una nota siempre referencia un comprobante `01` o `03` de la misma empresa |
| `sales_document_lines` | Nueva: `reference_line_id` (FK a la línea original, solo notas) | — |
| `series` | `document_type` admite `07`; el formato del código es `F` o `B` y tres letras o números | Único por empresa, como hoy |

- **Estado `discarded`:** se suma a `SalesDocumentStatus` como estado final e interno. No se envía a SUNAT y no admite notas.
- **Campos que cambian tras emitir:** a los del envío se suman `correction_status` y los del descarte. Todo lo demás sigue inmutable (RF-014 de la 005).

## Cálculo de la nota

- **Motivos 01 (anulación) y 06 (devolución total):** copian lo que queda por devolver de cada línea.
- **Motivo 07 (devolución por ítem):** el usuario elige cantidades por línea.
- **Importes de una devolución parcial:** `TaxCalculator::line()` con la cantidad devuelta y el mismo precio. El descuento de la línea se **prorratea**: `redondeo(descuento × devuelto / cantidad, 2)`. Cuando se devuelve lo último que queda de una línea, se toma el resto exacto de importe y descuento, para que la suma de todas las notas coincida al céntimo con la línea original (CE-002).
- **Totales:** `TaxCalculator::totals()`; la leyenda y el QR como en la 005.

## Servicios

| Pieza | Responsabilidad | Spec |
|---|---|---|
| `CreditNoteService::issue()` | Transacción: idempotencia → bloqueo del original → comprobaciones (aceptado u observado, no anulado ni devuelto totalmente, cantidades ≤ restantes, serie de nota activa con la letra correcta) → correlativo → líneas y cálculo → XML firmado → reposición de stock (según motivo y `restock`) → auditoría. El envío va tras el *commit*, con el mismo `SunatDispatcher` | HU-1, HU-2 |
| `UblBuilder` | Para el tipo `07`, construye la `Note` de Greenter: documento afectado (tipo y serie-número), motivo y descripción | RF-001 |
| `SunatDispatcher` | Al dar un estado final a una nota: si es **aceptada**, recalcula el `correction_status` del original; si es **rechazada**, revierte los movimientos `return` de esa nota | Casos límite |
| `InventoryService::restock()` | Repone una cantidad en los lotes de los que salió la venta original (en orden inverso a como salió), con movimientos `return` que referencian a la nota | A-40 |
| `InventoryService::reverse()` | Ahora también bloquea la reversión manual de ventas de un comprobante (hasta hoy solo bloqueaba las de tickets). Solo se revierten desde el descarte o desde el rechazo de una nota | Hallazgo del plan |
| `SalesDocumentService::discard()` | Solo `rejected` → `discarded` con motivo; revierte las ventas del comprobante; audita | HU-3 |

## Contrato de API (`/api/v1`, grupo de empresa)

| Método | Ruta | Rol | Descripción | Spec |
|---|---|---|---|---|
| POST | `/sales-documents/{salesDocument}/credit-notes` | Todos (A-41) | `{ idempotency_key, reason_code: 01\|06\|07, reason, restock?, series_id?, lines?: [{ line_position, quantity }] }` → 201 con la nota tras el primer envío | HU-1, HU-2 |
| POST | `/sales-documents/{salesDocument}/discard` | Admin (A-42) | `{ reason }` → comprobante descartado | HU-3 |
| GET | `/sales-documents/{salesDocument}` | Todos | Ahora incluye `correction_status`, `remaining` por línea, `credit_notes[]` (número, motivo, estado, total) y, si es nota, `reference` (número y tipo del comprobante modificado) | HU-4 |
| GET | `/sales-documents?document_type=07` | Todos | Las notas aparecen en el listado de la 005 con su filtro de tipo | HU-4 |
| POST | `/series` | Admin | Admite `document_type: 07` (`FC01`, `BC01`) | RF-003 |

Las descargas (PDF, XML, CDR) y el reintento de la 005 sirven igual para las notas. El PDF de la nota lleva el título «NOTA DE CRÉDITO ELECTRÓNICA», el comprobante que modifica y el motivo.

## Frontend

| Dónde | Cambio |
|---|---|
| Detalle del comprobante (`sales`) | «Registrar devolución» y «Anular» para comprobantes aceptados u observados. Un diálogo pide el motivo escrito, las cantidades por línea (devolución) o si la mercadería volvió (anulación, por defecto sí). Se muestran el estado de corrección, las notas con su estado y lo que queda por devolver por línea |
| Detalle de una nota | Mismo detalle de comprobante, con el documento que modifica enlazado y el motivo |
| Comprobante rechazado | «Descartar» para el administrador, con motivo; explica que se repone el stock y que hay que emitir uno nuevo |
| Listado de comprobantes | Filtro «Notas de crédito» y distintivo «Anulado» o «Devuelto» en los comprobantes corregidos |
| Series (`sunat`) | Crear series de nota de crédito (`FC01`, `BC01`) |

## Estrategia de pruebas

**API** (SQLite y PostgreSQL):

| Escenario / requisito | Prueba |
|---|---|
| Cálculo y prorrateo | `Unit/Sunat/CreditNoteCalculatorTest`: devoluciones parciales sucesivas que suman exacto al céntimo, descuentos prorrateados y resto final |
| XML | `Unit/Sunat/UblBuilderTest` ampliado: nota con documento afectado, motivo, líneas y totales |
| HU-1 | `Sales/IssueCreditNoteTest`: anulación con y sin reposición; no se anula dos veces; el original solo cambia cuando la nota es aceptada |
| HU-2 | Devolución parcial y total, límite por línea, segunda devolución, producto inactivo, serie de nota con la letra equivocada, idempotencia, el vendedor también emite (A-41) |
| Casos límite | Nota sobre comprobante pendiente, rechazado o descartado → 422; nota rechazada por SUNAT → se revierte su reposición |
| HU-3 | `Sales/DiscardRejectedTest`: solo rechazados, solo el administrador, stock repuesto, número usado, auditoría |
| Hallazgo | La reversión manual de una venta de comprobante → 422 |
| Concurrencia | `Integration/Sales/ConcurrentReturnTest` (solo PostgreSQL): devoluciones simultáneas de la misma línea nunca superan lo emitido; con mutación sin el bloqueo del original |
| CE-002 | `Integration/Sales/ReturnConsistencyTest`: secuencias aleatorias de ventas, devoluciones, anulaciones y rechazos → stock y saldos cuadran |
| CE-001 real | `tests/Beta/SunatBetaTest` ampliado con notas de anulación y de devolución parcial sobre boleta y factura |
| Principio VIII / IV | Rutas nuevas en `TenantRoutes.php` y `CrossTenantAccessTest`; `NoSunatSecretsLeakTest` con una nota |
| Contratos | `Contract/SalesDocumentResourcesTest` ampliado |

**Frontend (Vitest):** diálogo de devolución (cantidades, límite por línea), anulación con la opción de reposición, notas y saldo en el detalle, descarte (solo el administrador), filtro de notas y creación de series de nota.

**Verificación manual:** recorrido de extremo a extremo con notas aceptadas por SUNAT beta, devolución cronometrada (CE-003) y capturas a 360 px.

## Riesgos

| Riesgo | Mitigación |
|---|---|
| SUNAT beta rechaza la nota (referencia, serie o motivo) | *Spike* como primera tarea técnica |
| Descuadre al céntimo tras varias devoluciones parciales | Prorrateo con resto exacto en la última devolución, más la prueba de consistencia aleatoria |
| Devolver dos veces lo mismo por concurrencia | Bloqueo de la fila del comprobante original, más prueba concurrente con mutación |
| Stock repuesto por una nota que SUNAT luego rechaza | Reversión automática al rechazo, con prueba |
| Reversión manual de ventas de comprobantes (hallazgo) | Bloqueada en `InventoryService::reverse()` |

## Aprobación

Pendiente.
