# Verificación 005: Emisión y consulta de comprobantes en beta

**Fecha:** 2026-09-29 · **Tareas:** T090–T093 · **Rama:** `005-emision-comprobantes`

## Suites (T091, T092)

| Proyecto | Resultado |
|---|---|
| API, SQLite | ✅ 520 pruebas |
| API, PostgreSQL | ✅ 520 pruebas, incluidas la emisión simultánea en una serie (6 procesos → 151–156) y el envío simultáneo (5 procesos → 1 envío) |
| API, Pint | ✅ |
| Frontend | ✅ `type-check`, `lint`, 137 pruebas y `build-only` |

Pruebas con mutación (fallan si se quita la protección):

| Protección | Mutación | Resultado |
|---|---|---|
| Correlativo bloqueado (`nextNumber`) | Sin `lockForUpdate` | ❌ números duplicados, frenados por el índice único |
| *Lease* de envío | Sin la condición de `locked_until` | ❌ 5 envíos en vez de 1 |
| Certificado solo en memoria | Registrar el PEM en el log | ❌ la prueba encuentra la clave privada en los logs |

## SUNAT beta real (T002, T090)

- **Spike (T002):** 9 casos de CE-001 aceptados con CDR 0 y sin observaciones. Hallazgos: beta limita la frecuencia (`HTTP 401` si se envía segundos después de otro documento), no detecta reenvíos (devuelve un CDR nuevo) y no tiene consulta de CDR (R-3).
- **Prueba con el código de producción (T090, `tests/Beta/SunatBetaTest`):** 8 casos (gravado, exonerado, inafecto, mixto, descuento, redondeos límite, boleta con DNI de más de S/ 700 y factura con RUC) aceptados **sin observaciones** y con CDR.

## Recorrido de extremo a extremo (T093)

Por el proxy de Vite, con `php artisan schedule:work` corriendo, sobre una base recién sembrada. La caída de SUNAT se simuló con una segunda instancia de la API (puerto 8001) con `SUNAT_BETA_ENDPOINT` inalcanzable. 16/16 pasos correctos.

| # | Paso | Spec | Resultado |
|---|---|---|---|
| 1 | Comprobantes demo del seeder | T073 | ✅ B001-00000001 y F001-00000001 aceptados por SUNAT beta |
| 2 | El vendedor registra un cliente con DNI | HU-5.2 | ✅ |
| 3 | Boleta a «Cliente varios» | HU-1.1 | ✅ B001-00000002 aceptada, 205 ms de API (incluido el envío a SUNAT) |
| 4 | Boleta de más de S/ 700 sin comprador | HU-1.2 | ✅ 422 |
| 5 | …con DNI | HU-1.2 | ✅ B001-00000003 (S/ 725.20) aceptada |
| 6 | Factura a un cliente sin RUC | HU-2.2 | ✅ 422, sugiere boleta |
| 7 | Factura con RUC, descuento y un servicio | HU-2.1, HU-3.3 | ✅ F001-00000002 aceptada sin observaciones |
| 8 | Doble envío con la misma clave | Idempotencia | ✅ 201 → 200, un solo comprobante |
| 9 | Descargas | RF-007, RF-021 | ✅ PDF A4 y 80 mm, XML firmado y CDR (zip) |
| 10 | SUNAT caída | HU-4.1 | ✅ pendiente con su número y próximo intento en 1 min |
| 11 | «Reintentar» con SUNAT de vuelta | HU-4.3 | ✅ aceptado de inmediato |
| 12 | La lista avisa de los pendientes | HU-6.1 | ✅ |
| 13 | El scheduler reenvía el pendiente al vencer la espera | HU-4.2, CE-002 | ✅ `issue:unreachable` → `scheduled:accepted` |
| 14 | El vendedor ve los comprobantes del administrador | A-32 | ✅ |
| 15 | Otra empresa | Principio VIII | ✅ 404 y lista vacía |
| 16 | Búsqueda de secretos en 42 respuestas | Principio IV | ✅ sin clave privada ni contraseña del certificado |

## Pantallas a 360 px

| Pantalla | Resultado |
|---|---|
| Vender | ✅ Selector Ticket · Boleta · Factura; el distintivo de pruebas aparece al elegir boleta o factura |
| Ventas › Comprobantes | ✅ Pestañas, filtros, estados y totales |
| Detalle de factura | ✅ Resultado de SUNAT, descargas, cliente, líneas con descuento, totales por afectación e intentos |
| Clientes (administrador) | ✅ Lista y edición |
| PDF A4 y 80 mm | ✅ Revisados a ojo: marca de pruebas, totales por afectación, importe en letras y QR |

## Hallazgos corregidos

- **Observaciones 4096–4098 de SUNAT:** el domicilio fiscal del emisor iba sin departamento, provincia ni distrito. Ahora se copian del ubigeo al emitir y se envían; los comprobantes quedan aceptados sin observaciones.
- **Número en letras heredado:** fallaba con 21–29, con los miles terminados en uno y con millones. Se reescribió con su tabla de pruebas.
- **Menú:** «Ventas» no quedaba marcado en las pantallas de comprobantes ni en los detalles (rutas hermanas). Ahora se compara por prefijo de ruta.
- **Plantillas PDF heredadas** (unas 4 000 líneas, acopladas a los modelos viejos): se reemplazaron por una plantilla compacta para A4 y 80 mm; las viejas pasaron a `legacy/`.

## Pendiente de confirmación del usuario

- **CE-003:** cronometrar una boleta de 3 productos en el celular (objetivo: menos de 45 s).
- ⚖️ **A-22:** revisar con un contador los tipos de operación y el tope de S/ 700 antes de producción.
- **Servidor:** en producción hará falta el cron de `schedule:run` para los reintentos automáticos (documentado en el README de la API).
- **Datos de prueba:** el recorrido dejó comprobantes y un cliente en la base local. Se limpian con `php artisan migrate:fresh --seed`.
