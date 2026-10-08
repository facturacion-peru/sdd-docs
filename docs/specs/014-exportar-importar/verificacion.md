# Verificación 014: Exportar e importar datos

**Fecha:** 2026-10-07 · **Tareas:** T035–T037 · **Rama:** `014-exportar-importar` (API y frontend)

## Suites (T026, T034, T037)

| Proyecto | Resultado |
|---|---|
| API, SQLite | ✅ 757 pruebas (13 omitidas: requieren PostgreSQL o pcntl) |
| API, PostgreSQL | ✅ 770 pruebas, incluidas las confirmaciones simultáneas y el alta durante una importación (T021) |
| API, Pint | ✅ |
| Frontend | ✅ `type-check`, `lint`, 281 pruebas y `build-only` |

Prueba con mutación (T021):

| Protección | Mutación | Resultado |
|---|---|---|
| Bloqueo de la vista previa al confirmar | Sin `lockForUpdate` | ❌ la segunda confirmación simultánea recibe 409 en vez de 410 |

## Rendimiento

| Medición | Resultado |
|---|---|
| Exportar 20 000 líneas de venta (T014) | XLSX: ~27 MB adicionales y 8,4 s. CSV: ~25 MB y 3,8 s. Con 40 000 líneas la memoria es la misma |
| Importar 2 000 productos con stock, PostgreSQL de pruebas (T025) | Vista previa 6,0 s. Confirmación 35,8 s, que bajó a **13,3 s** al revalidar las altas con una sola consulta |
| **CE-001**: 500 productos con stock por el proxy de Vite (T035) | Vista previa **1,5 s**, confirmación **3,8 s**: 5,3 s de sistema, muy por debajo de los 2 minutos |

## Recorrido de extremo a extremo (T035)

Se hizo por el proxy de Vite, sobre la base local sembrada. Las exportaciones usaron la empresa demo, que tiene ventas. Las importaciones, «Otra Empresa», para no llenar el catálogo de la demo. Resultado: 15/15 comprobaciones correctas.

| # | Paso | Spec | Resultado |
|---|---|---|---|
| 1 | Productos en XLSX y en CSV con lotes (ZIP con `productos.csv` y `lotes.csv`) | HU-1 | ✅ |
| 2 | Clientes en XLSX y en CSV | HU-2 | ✅ |
| 3 | Ventas del mes en XLSX y en CSV (ZIP con `documentos.csv` y `lineas.csv`) | HU-3 | ✅ |
| 4 | Ventas de 13 meses | HU-3.5 | ✅ 422 |
| 5 | Plantilla de productos | HU-4.1 | ✅ |
| 6 | El vendedor exporta o importa | RF-001 | ✅ 403 |
| 7 | CE-001: 500 productos con stock, vista previa y confirmación | HU-4, CE-001 | ✅ 500 creados y 500 entradas |
| 8 | Stock de un importado | A-72 | ✅ `E2E-005`: lote con vencimiento 2027-06-30, costo 2.5000 y movimiento `entry` de 15 |
| 9 | Confirmar dos veces | — | ✅ 410 |
| 10 | Otra empresa confirma esa vista previa | CE-005 | ✅ 404 |
| 11 | Archivo con un código existente y un precio con 3 decimales | HU-4.3 | ✅ errores en la fila 2 (`codigo`) y en la fila 3 (`precio_venta`); no se puede confirmar |
| 12 | Archivo corregido en «Crear y actualizar» | HU-4.5 | ✅ 1 nuevo y 1 con cambio de nombre, confirmado |

LibreOffice abre las tres exportaciones XLSX con todas sus hojas. Los importes llegan como números, y las fechas y horas, en hora de Lima.

## Pantallas (T036)

- **Web, con Chrome sin pantalla por CDP:** «Importar y exportar» a 360 px en claro y oscuro y en escritorio; vista previa con errores; vista previa con cambios; diálogo de confirmación; resultado; y «Exportar» en Productos.
  - **Corregido en el camino:**
    - **Desborde horizontal a 360 px:** venía de la rejilla de la vista sin `grid-cols-1` (la trampa conocida de la columna implícita con texto truncado).
    - **Fechas cortadas:** se apilan por debajo de 400 px.
    - **Espacio:** faltaba el espacio tras «Nombre:».
- **Emulador Android 15:**
  - «Productos y stock» abre «Compartir» con `productos.xlsx`.
  - Se elige `android-import.csv` desde el selector del teléfono, se revisa (1 nuevo) y se confirma («1 creado… y 1 con stock inicial»).
- **En T003 (antes de escribir código):** se comprobó que el WebView de Capacitor 8 entrega el archivo elegido con su nombre, tamaño, tipo MIME y contenido.

## Hallazgos

- **OpenSpout:**
  - **Textos con `=`:** el lector devuelve como «fórmula» sin valor guardado cualquier texto que empieza con `=`. En ese caso se usa el texto, que nunca se evalúa.
  - **Filas vacías:** no se escriben las filas formadas solo por celdas vacías. Por eso la plantilla lleva cadenas vacías con formato de texto.
- **PostgreSQL** (SQLite no lo detecta):
  - **UUID en la auditoría:** `audit_logs.auditable_id` es entero, así que `import.confirmed` se audita sin modelo y con el id de la vista previa en los cambios.
  - **Revalidación lenta:** revalidar el código único fila por fila dentro de la transacción costaba 17,6 s de 26 en consultas.
- **Eloquent:** `map()` sobre una colección vacía de modelos sigue siendo una colección de Eloquent, y su `intersect()` falla con textos. Hay que usar `toBase()`.
- **OpenAPI:** el generador tipa los parámetros de ruta como números. El frontend pasa `kind` y el UUID con un *cast* comentado.

## Integración (T039)

Merge a `main` de los dos repositorios el 2026-10-07. Sobre `main`: API 757 (SQLite) y 770 (PostgreSQL), Pint, y frontend 281 pruebas, tipos, lint y build.
