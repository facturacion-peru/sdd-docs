# Spec 014: Exportar e importar datos

**Estado:** Implementada (2026-10-07; falta el merge) · **Versión:** 1.0.1 · **Fecha:** 2026-10-07
**Depende de:** 001, 002, 003, 005, 007 · **Aclaraciones:** A-68 a A-74 (retoma A-26)
**Evaluación previa:** [`investigacion/002-exportar-importar-datos.md`](../../investigacion/002-exportar-importar-datos.md)

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que el administrador de una empresa pueda **sacar sus datos** (productos, stock, clientes y ventas) a una hoja de cálculo para revisarlos, cuadrar caja o llevarlos a otro sistema, y **cargar en bloque** su catálogo y sus clientes cuando viene de otro sistema, sin registrar cientos de productos a mano (A-26).

Las ventas no se importan (A-68): un comprobante solo existe si se firmó y se envió a SUNAT desde aquí (principio II), y un ticket mueve el stock.

## Historias de usuario

### HU-1 · Exportar productos y stock (P1)

Como administrador de empresa, quiero descargar mi catálogo con su stock para revisarlo en Excel o compartirlo con mi contador.

**Por qué esta prioridad:** es la exportación más simple y la base de la plantilla de importación (el mismo formato de columnas).
**Prueba independiente:** con productos activos e inactivos, con y sin lotes, la descarga trae todos los productos de la empresa y ninguno de otra.

**Escenarios de aceptación**

1. **Dado** productos registrados, **cuando** exporto el catálogo, **entonces** descargo un archivo XLSX (o CSV si lo elijo) con una fila por producto: código, nombre, tipo, unidad, precio con IGV, afectación, stock mínimo, controla vencimiento, activo, stock actual y costo del stock (A-69).
2. **Dado** el catálogo, **cuando** elijo «con lotes», **entonces** el archivo trae además una hoja con un lote por fila: producto, lote, ingreso, vencimiento, cantidad restante y costo unitario. En CSV es un segundo archivo.
3. **Dado** un filtro activo en el listado de productos (búsqueda, activos o inactivos), **cuando** exporto, **entonces** el archivo respeta ese filtro.

### HU-2 · Exportar clientes (P2)

Como administrador de empresa, quiero descargar mis clientes para tenerlos fuera del sistema o llevarlos a otro.

**Prueba independiente:** con clientes de dos empresas, la descarga de una trae solo los suyos y queda registrada en la auditoría.

**Escenarios de aceptación**

1. **Dado** clientes registrados, **cuando** exporto, **entonces** descargo una fila por cliente: tipo de documento, número, nombre y dirección.
2. **Dado** una exportación de clientes, **cuando** reviso la auditoría, **entonces** veo quién la hizo, cuándo y cuántas filas tenía (A-73).

### HU-3 · Exportar ventas (P1)

Como administrador de empresa, quiero descargar las ventas de un periodo, por documento y por línea, para cuadrar caja y ver qué se vende (A-74).

**Por qué esta prioridad:** hoy no hay forma de sacar las ventas de un mes sin abrir los documentos uno por uno.
**Prueba independiente:** con tickets, boletas, facturas, anulados, rechazados y notas de crédito en un mes, el total de la hoja de documentos coincide con el total del panel de inicio (A-54) en los mismos días.

**Escenarios de aceptación**

1. **Dado** un rango de fechas de hasta 12 meses, **cuando** exporto ventas, **entonces** el XLSX trae dos hojas. En CSV son dos archivos:
   - **Documentos:** una fila por ticket o comprobante, con fecha y hora de Lima, tipo, serie y número, cliente, vendedor, medio de pago, bases, IGV, descuento, total, estado y, en las notas, el documento que modifican.
   - **Líneas:** una fila por línea vendida, con el documento, producto, cantidad, precio, descuento e importe.
2. **Dado** filtros de tipo (ticket, boleta, factura, nota de crédito) y de estado, **cuando** exporto, **entonces** solo salen los documentos que los cumplen.
3. **Dado** una nota de crédito, **cuando** aparece en el archivo, **entonces** sus importes van en negativo y la hoja de documentos permite sumar el neto vendido.
4. **Dado** un documento anulado, rechazado o descartado, **cuando** aparece en el archivo, **entonces** se distingue por su estado y no hay que adivinar si cuenta como venta. Una columna indica si suma según la regla de A-54.
5. **Dado** un rango de más de 12 meses o con la fecha final antes que la inicial, **cuando** pido la exportación, **entonces** se rechaza con un mensaje claro (A-73).

### HU-4 · Importar productos (P1)

Como administrador de una empresa que viene de otro sistema, quiero cargar mi catálogo y su stock inicial desde una hoja de cálculo, para empezar a vender el mismo día (A-26, A-72).

**Por qué esta prioridad:** es la razón de A-26: cientos de productos a mano son días de trabajo y errores.
**Prueba independiente:** una plantilla con productos nuevos, uno existente y una fila con error se valida, muestra la vista previa y solo se aplica al confirmar; el stock y el kárdex quedan como si se hubieran registrado a mano.

**Escenarios de aceptación**

1. **Dado** la pantalla de importación, **cuando** descargo la plantilla, **entonces** recibo un XLSX con las columnas, un ejemplo y los valores admitidos: tipo, unidad, afectación 10/20/30 y sí/no (A-69).
2. **Dado** un archivo XLSX o CSV con el formato de la plantilla, **cuando** lo subo, **entonces** veo una vista previa:
   - cuántas filas se **crean**, cuántas se **actualizan** y cuántas tienen **errores**;
   - cada error con su número de fila, su columna y el motivo.

   Hasta que confirme no se guarda nada (A-71).
3. **Dado** una vista previa con al menos un error, **cuando** intento confirmar, **entonces** no se puede: hay que corregir el archivo y subirlo de nuevo (todo o nada, A-71).
4. **Dado** el modo «solo crear» (por defecto), **cuando** una fila trae un código que ya existe en la empresa, **entonces** se marca como error (A-70).
5. **Dado** el modo «crear y actualizar», **cuando** una fila trae un código existente, **entonces** la vista previa muestra campo por campo qué cambia (por ejemplo, el precio de 3.50 a 3.80), y al confirmar se actualiza (A-70).
6. **Dado** un producto **nuevo** con las columnas de stock inicial (cantidad, costo y, si corresponde, lote y vencimiento), **cuando** confirmo, **entonces** se registra una entrada de mercadería normal: un lote y un movimiento en el kárdex, atribuidos a mí (A-72, A-28).
7. **Dado** un producto **existente** con columnas de stock, **cuando** confirmo, **entonces** el stock no cambia y la vista previa avisa que esas columnas se ignoran (A-72).
8. **Dado** la confirmación, **cuando** termina, **entonces** veo cuántos productos se crearon y actualizaron, y la importación queda en la auditoría con cada alta y cada cambio, como si se hubieran hecho a mano (A-70).

### HU-5 · Importar clientes (P2)

Como administrador de empresa, quiero cargar mis clientes frecuentes desde una hoja de cálculo para no escribir sus datos al emitir facturas.

**Prueba independiente:** una plantilla con DNI, RUC y carné de extranjería válidos e inválidos se valida con las mismas reglas que el alta manual.

**Escenarios de aceptación**

1. **Dado** la plantilla de clientes, **cuando** la subo, **entonces** se validan como en el alta manual el tipo de documento, el número (8 dígitos para DNI; 11 para RUC, con dígito verificador) y el nombre. La vista previa sigue las mismas reglas de HU-4: crear o actualizar, errores por fila y todo o nada.
2. **Dado** el modo «crear y actualizar», **cuando** una fila trae un documento existente, **entonces** se actualizan el nombre y la dirección, con vista previa del cambio. El tipo y el número no cambian porque identifican al cliente (A-70).

## Casos límite

- **Mismo código dos veces en el archivo:** las dos filas se marcan como error. No gana «la última».
- **Archivo muy grande:** si supera 2 000 filas o 5 MB, se rechaza antes de procesarlo y se indica el límite (A-73).
- **Archivo equivocado:** si es de otro tipo, está dañado, tiene columnas que faltan o están cambiadas de nombre, se rechaza con un mensaje que dice qué columna falta. Las columnas sobrantes se ignoran con aviso.
- **Fórmulas en las celdas:**
  - al importar, se lee el valor, nunca se evalúa una fórmula;
  - al exportar, un texto que empieza con `=`, `+`, `-` o `@` (un nombre de producto, por ejemplo) se guarda de modo que Excel no lo ejecute al abrirlo.
- **Decimales y textos:**
  - se aceptan precios con punto o con coma decimal, con las mismas precisiones que el alta manual: 2 decimales en precio, 3 en cantidades y 4 en costo unitario;
  - el sistema guarda y exporta los importes como decimales exactos, sin redondeos de coma flotante;
  - al exportar CSV, las tildes y la «ñ» se ven bien al abrir el archivo en Excel.
- **Códigos y documentos con ceros a la izquierda** (`0012`, DNI `01234567`): se conservan, aunque Excel los haya guardado como número. Si el número de DNI llega con 7 dígitos, se rechaza; no se completa con ceros.
- **Cambios entre la vista previa y la confirmación:** si otro usuario crea un producto con el mismo código en ese momento, la confirmación se rechaza entera y se pide repetir la vista previa. No queda nada a medias.
- **Vista previa vieja:** una vista previa no confirmada caduca al cabo de un tiempo y no se puede confirmar después.
- **Empresa desactivada o sesión vencida durante la importación:** no se aplica nada.
- **Vendedor:** no ve las opciones de exportar ni importar, y la API se lo niega aunque llame directamente (A-73).
- **Administrador de la plataforma:** no exporta datos de las empresas (A-37).
- **Producto de tipo servicio con columnas de stock:** error en esa fila (los servicios no tienen stock, spec 002).
- **Exportación sin resultados:** se descarga el archivo con los encabezados y sin filas; no es un error.
- **Celular:** exportar comparte o descarga el archivo como el resto de descargas de la app (A-63). Importar también funciona, pero la vista previa está pensada para pantallas grandes.

## Requisitos funcionales

- **RF-001:** Solo el administrador de empresa DEBE poder exportar e importar. Cada operación DEBE limitarse a su empresa (principio VIII, A-73).
- **RF-002:** El sistema DEBE exportar productos (con stock y, opcionalmente, lotes), clientes y ventas en XLSX y en CSV (A-68, A-69).
- **RF-003:** La exportación de ventas DEBE incluir tickets, comprobantes y notas de crédito de un rango de fechas de Lima de hasta 12 meses, con hojas de documentos y de líneas, filtros de tipo y estado, y notas en negativo (A-74).
- **RF-004:** El sistema DEBE importar productos y clientes desde XLSX o CSV con el formato de una plantilla descargable (A-68, A-69).
- **RF-005:** Toda importación DEBE pasar por una vista previa que no guarda nada. La confirmación DEBE aplicar todas las filas en una sola operación, o ninguna si hay un error o un conflicto (A-71).
- **RF-006:** La importación DEBE validar cada fila con las mismas reglas que el alta y la edición manuales (principio II).
- **RF-007:** El modo por defecto DEBE ser «solo crear». «Crear y actualizar» DEBE mostrar los cambios campo por campo antes de confirmar (A-70).
- **RF-008:** El stock inicial de un producto nuevo importado DEBE registrarse como una entrada de mercadería normal. El stock de un producto existente NO DEBE cambiar por una importación (A-72).
- **RF-009:** El sistema DEBE rechazar importaciones de más de 2 000 filas o 5 MB, y exportaciones de ventas de más de 12 meses (A-73).
- **RF-010:** Cada exportación e importación DEBE quedar en la auditoría con usuario, fecha, tipo de dato, filtros y número de filas. Cada alta o cambio importado DEBE auditarse como uno manual (principio VIII).
- **RF-011:** Las exportaciones DEBEN neutralizar los textos que una hoja de cálculo interpretaría como fórmula. Las importaciones NO DEBEN evaluar fórmulas ni macros (principio IV).
- **RF-012:** Los importes DEBEN exportarse e importarse como decimales exactos (principio II).
- **RF-013:** Exportar e importar DEBEN funcionar en la web y en la app Android a 360 px (principio VII).

## Entidades clave

- **Exportación:** archivo generado al momento con los datos de una empresa según un tipo y unos filtros. No se guarda; solo queda su registro en la auditoría.
- **Plantilla de importación:** archivo con las columnas admitidas, ejemplos y valores válidos de cada tipo de dato.
- **Vista previa de importación:** resultado de validar un archivo (filas que se crean, que se actualizan y con error, con su detalle). Es temporal, pertenece a un usuario y a una empresa, y caduca.
- **Importación confirmada:** aplicación de una vista previa sin errores. Queda en la auditoría junto con las altas y los cambios que produjo.

## Criterios de éxito

- **CE-001:** Un catálogo de 500 productos con stock inicial, en una plantilla correcta, se importa de punta a punta (subir, revisar y confirmar) en menos de 2 minutos. Después, el stock y el kárdex coinciden con el archivo.
- **CE-002:** El total neto de la hoja de documentos de un mes coincide con la suma de los totales diarios del panel de inicio (A-54) para el mismo periodo. Lo comprueba una prueba automatizada con anulados, rechazados y notas de crédito.
- **CE-003:** Una importación con una sola fila inválida entre 2 000 no guarda nada, y la vista previa señala esa fila y su columna.
- **CE-004:** Exportar un producto y volver a importar ese archivo en «crear y actualizar» no produce cambios: la vista previa muestra 0 cambios.
- **CE-005:** Ninguna exportación ni importación lee o modifica datos de otra empresa, ni las puede hacer un vendedor. Lo prueban las pruebas de aislamiento y acceso.

## Fuera de alcance

- Importar ventas, tickets o comprobantes de otro sistema (A-68).
- Formatos oficiales de SUNAT, como el PLE o el registro de ventas ⚖️: otra spec, con el contador.
- Exportar o importar desde el panel de la plataforma (A-37).
- Respaldo o migración completa de una empresa, con usuarios, configuración SUNAT y archivos XML/CDR. Las copias las cubre la 009.
- Exportaciones programadas o enviadas por correo.
- Importar entradas de stock de productos existentes, ajustes, usuarios o series.
- Exportación del vendedor con sus propias ventas (A-73: por ahora solo el administrador).

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-07 | A-68 Qué datos | Exportar productos con stock, clientes y ventas; importar solo productos y clientes |
| 2026-10-07 | A-69 Formato | Exportar en XLSX (por defecto) y CSV; importar XLSX o CSV desde la plantilla |
| 2026-10-07 | A-70 Registros existentes | «Solo crear» por defecto o «crear y actualizar» con vista previa campo por campo; el stock no se toca |
| 2026-10-07 | A-71 Filas con errores | Todo o nada, con vista previa de errores por fila y columna |
| 2026-10-07 | A-72 Stock inicial | Columnas opcionales que generan una entrada normal, solo en productos nuevos |
| 2026-10-07 | A-73 Quién y límites | Solo el administrador; 2 000 filas y 5 MB por importación; ventas de hasta 12 meses; todo auditado |
| 2026-10-07 | A-74 Ventas | Dos hojas (documentos y líneas), filtros de fecha, tipo y estado; notas en negativo; sin PLE |
