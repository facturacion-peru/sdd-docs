# Verificación 007: Anulación y corrección de comprobantes

**Fecha:** 2026-09-30 · **Tareas:** T070–T073 · **Rama:** `007-notas-credito`

## Suites (T071, T072)

| Proyecto | Resultado |
|---|---|
| API, SQLite | ✅ 609 pruebas |
| API, PostgreSQL | ✅ 609 pruebas, incluidas las devoluciones simultáneas (5 procesos sobre 3 unidades → 3 pasan) |
| API, Pint | ✅ |
| Frontend | ✅ `type-check`, `lint`, 162 pruebas y `build-only` |
| Consistencia (CE-002) | ✅ `ReturnConsistencyTest`: 4 semillas × 30 pasos aleatorios, más de 2 600 comprobaciones de stock y saldos |

Prueba con mutación:

| Protección | Mutación | Resultado |
|---|---|---|
| Bloqueo del comprobante al emitir la nota | Sin `lockForUpdate` | ❌ se devuelven 5 unidades de una línea de 3 |

Con un bien (no un servicio) la mutación no bastaba para fallar: `restock()` bloquea el producto y tampoco repone más de lo vendido, así que es una segunda red. La prueba usa un servicio, donde solo protege el bloqueo del comprobante.

## SUNAT beta real (T002, T070)

- **Spike:** boleta y factura aceptadas y, sobre ellas, notas BC01 (motivos 07 y 01) y FC01 (motivo 06), todas con CDR 0 y sin observaciones. La boleta de S/ 42.50 cuadra con sus dos notas (7.25 + 35.25).
- **Con el código de producción (`SunatBetaTest`, 9 casos):** los 8 de la 005 y el de notas (devolución parcial con descuento, anulación del resto y devolución total de factura), aceptados sin observaciones.

## Recorrido de extremo a extremo (T073)

Por el proxy de Vite, sobre la base sembrada; notas reales en SUNAT beta. La nota pendiente se creó con una instancia de la API con SUNAT inalcanzable. 15/15 pasos correctos.

| # | Paso | Spec | Resultado |
|---|---|---|---|
| 1 | Boleta con descuento, producto por peso y servicio | — | ✅ S/ 88.00 |
| 2 | Devolución parcial hecha por el vendedor | HU-2, A-41 | ✅ BC01-00000001 aceptada; 206 ms de API |
| 3 | Stock repuesto y «devuelto parcialmente» | HU-2.1, A-40 | ✅ quedan 2 |
| 4 | Devolver más de lo que queda | HU-2.2 | ✅ 422 |
| 5 | Devolución total del resto | HU-2.3, CE-002 | ✅ notas S/ 88.00 = boleta S/ 88.00 |
| 6 | Stock igual que antes de la venta | CE-002 | ✅ |
| 7 | Anular algo ya devuelto | HU-1.3 | ✅ 422 |
| 8 | Anulación de factura sin reponer | HU-1.2 | ✅ FC01-00000001 aceptada, factura «anulada», stock sin cambio |
| 9 | Nota con SUNAT caída | Casos límite | ✅ pendiente; el comprobante no cambia |
| 10 | Reintento | Casos límite | ✅ aceptada; comprobante «devuelto parcialmente» |
| 11 | Descarte de un rechazado (rechazo **simulado** en la base) | HU-3, A-42 | ✅ el vendedor no puede; el administrador sí; el stock vuelve |
| 12 | El número del descartado no se reutiliza | HU-3.1 | ✅ B001-00000004 → B001-00000005 |
| 13 | Revertir a mano la venta de un comprobante | Hallazgo del plan | ✅ 422 |
| 14 | Otra empresa | Principio VIII | ✅ 404 |
| 15 | Búsqueda de secretos en 42 respuestas | Principio IV | ✅ sin coincidencias |

## Pantallas a 360 px

| Pantalla | Resultado |
|---|---|
| Comprobante devuelto parcialmente | ✅ Distintivo, «Quedan 1 por devolver», notas con estado y botones de devolución y anulación |
| Nota de crédito | ✅ Enlace al comprobante que modifica y motivo |
| Listado | ✅ «Descartado» y «Devuelto parcialmente» a la vista |
| Series | ✅ Grupo «Notas de crédito» (BC01, FC01) |

## Hallazgos corregidos

- **La plantilla de nota de Greenter no admite descuentos de línea:** el descuento se descartaba sin aviso y SUNAT solo lo aceptaba por tolerancia. Ahora las líneas de la nota van por sus valores netos, confirmado contra SUNAT beta.
- **Reversión manual de ventas de comprobantes** desde el inventario: estaba permitida (solo se bloqueaban las de tickets). Ahora solo se revierten con una nota o un descarte.
- **La emisión de la 005 habría aceptado el tipo 07** al añadirlo al enum; ahora solo admite factura y boleta.
- **OpenAPI:** el generador marcaba como obligatorio un campo con `required_if` cuando sus hijos eran obligatorios.

## Pendiente de confirmación del usuario

- **CE-003:** cronometrar una devolución desde el celular (objetivo: menos de 1 minuto).
- ⚖️ **A-39:** revisar con el contador los motivos 01, 06 y 07 antes de producción.
- **Rechazo real:** el descarte se probó con un rechazo simulado en la base; SUNAT beta aceptó todos los envíos del recorrido.
- **Datos de prueba:** el recorrido dejó comprobantes y notas en la base local. Se limpian con `php artisan migrate:fresh --seed`.
