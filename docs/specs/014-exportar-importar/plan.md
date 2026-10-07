# Plan 014: Exportar e importar datos

**Estado:** Aprobado (2026-10-07) · **Fecha:** 2026-10-07 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0.1)

## Resumen técnico

**Exportaciones:** tres endpoints `GET` de solo administrador generan el archivo al vuelo, sin guardarlo, a partir de consultas por lotes, y lo envían en *streaming*. Hay uno por cada tipo de dato: productos, clientes y ventas.

**Importaciones:** tienen dos pasos.
1. Un `POST` con el archivo lo lee, valida cada fila con las **mismas reglas** que el alta manual y guarda una *vista previa* temporal en la base. Guarda las filas ya normalizadas, nunca el archivo.
2. Otro `POST` confirma esa vista previa. Vuelve a validar contra el estado actual y aplica todo en **una transacción** con los servicios existentes (`ProductService`, `CustomerService` e `InventoryService::registerEntry`), así que la auditoría y el kárdex son los de siempre.

Una sola librería, **OpenSpout**, lee y escribe XLSX y CSV.

En el frontend, la feature nueva `data-transfer` reúne la exportación de ventas y las importaciones. Además, «Productos» y «Clientes» tienen su propio botón de exportar, que respeta el filtro de la lista.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | API y frontend |
| Ramas | `014-exportar-importar` en los dos, desde `main` |
| Dependencias nuevas | API: `openspout/openspout` **^4.32** (MIT, lectura y escritura por *streaming* de XLSX y CSV). Se elige la 4.x porque la 5.x exige PHP 8.4 y el proyecto declara `^8.3`. Usa `ext-zip`, `ext-xmlreader` y `ext-dom`, que ya están en el servidor de la 009 (`php-xml` y `php-zip`). Frontend: ninguna |
| Migraciones | `import_previews` (nueva) |
| CI | Agregar `xmlreader` a `PHP_EXTENSIONS` en `.github/workflows/ci.yml` |
| Spec | La v1.0.1 corrige las precisiones de los decimales (3 en cantidades y 4 en costo, como el alta manual), detectadas al leer las reglas actuales |

## Chequeo de constitución

| Principio | Cumple | Cómo |
|---|---|---|
| I. Spec primero | Sí | Spec 014 v1.0.1 aprobada, con A-68 a A-74 |
| II. Corrección fiscal | Sí | Las ventas no se importan. Las filas se validan con las reglas del alta manual, extraídas a clases compartidas. Los importes se manejan como cadenas decimales con bcmath (`Decimal`), y la columna «¿Suma como venta?» usa la misma regla del panel de inicio (A-54) |
| III. Contrato de API | Sí | Rutas bajo `/api/v1` con sus FormRequest (el archivo se documenta como `binary`), y `pnpm api:types`. Las respuestas JSON se tipan a mano y las protegen pruebas en `tests/Feature/Contract` |
| IV. Seguridad | Sí | Solo el administrador, con límite de frecuencia propio y límite de 5 MB y 2 000 filas. Las fórmulas se leen como valor y nunca se evalúan. Al exportar se neutralizan los textos que empiezan con `=`, `+`, `-`, `@`, tabulador o retorno de carro. El archivo subido no se guarda en disco |
| V. Simplicidad | Sí | Una librería para los dos formatos y todo síncrono (sin colas). La vista previa es una fila en la base con caducidad, y la confirmación reutiliza los servicios existentes |
| VI. Verificable | Sí | Pest en SQLite y PostgreSQL, Vitest, y recorrido manual en la web y en el emulador |
| VII. Personas reales | Sí | Plantilla con ejemplos y valores admitidos, errores por fila y columna en español, vista previa en tarjetas a 360 px, y descarga o «Compartir» en la app (A-63) |
| VIII. Aislamiento y auditoría | Sí | Las rutas van en `TenantRoutes.php`, y la vista previa guarda `company_id` y `user_id`: solo la confirma quien la creó, en su empresa. Cada exportación e importación se audita, y cada alta o cambio importado se audita como uno manual |

## Formato de los archivos

**Encabezados en español, sin tildes ni espacios** (`codigo`, `nombre`…). Así resisten la edición en Excel y la importación encuentra las columnas aunque cambien de orden. Mayúsculas y espacios alrededor no importan.

### Productos (plantilla, exportación e importación)

| Columna | Obligatoria al importar | Valores |
|---|---|---|
| `codigo` | Sí | Texto de hasta 32 caracteres. Es la clave, única por empresa |
| `nombre` | Sí | Hasta 255 caracteres |
| `tipo` | Sí | `bien` o `servicio` |
| `unidad` | Sí | El código SUNAT (`NIU`, `KGM`, `ZZ`…) o el nombre visible de `UnitOfMeasure` |
| `precio_venta` | Sí | Con IGV, hasta 2 decimales (punto o coma) |
| `afectacion_igv` | Sí | `10`, `20` o `30` |
| `stock_minimo` | No | Hasta 3 decimales; vacío en los servicios |
| `controla_vencimiento` | No (`no`) | `si` o `no` |
| `activo` | No (`si`) | `si` o `no` |
| `stock_inicial` | No | Solo para productos nuevos que son bienes. Hasta 3 decimales y mayor que 0 |
| `costo_unitario` | No | Hasta 4 decimales, con `stock_inicial` |
| `lote` | No | Con `stock_inicial`; si falta, se numera automáticamente como en la 002 |
| `vencimiento` | Si `controla_vencimiento` = `si` y hay stock | Fecha `AAAA-MM-DD` o una fecha de Excel |

La exportación agrega `stock_actual` y `costo_stock` (el costo de los lotes con saldo). Al reimportar, la vista previa ignora esas dos columnas y avisa de ello, así que CE-004 da 0 cambios.

Con «con lotes» se agrega la hoja `lotes`: `codigo`, `nombre`, `lote`, `ingreso`, `vencimiento`, `cantidad_restante` y `costo_unitario`.

### Clientes

`tipo_documento` (`DNI`, `CE`, `RUC` o el código `1`, `4`, `6`), `numero_documento`, `nombre` y `direccion`.

### Ventas (solo exportación, A-74)

- **Hoja `documentos`:**
  - fecha y hora de Lima, `tipo` (ticket, boleta, factura o nota de crédito) y `numero`;
  - cliente (tipo y número de documento, nombre), `vendedor` y `medio_pago`;
  - `op_gravadas`, `op_exoneradas`, `op_inafectas`, `igv`, `descuento` y `total`;
  - `estado` y `documento_modificado`, para las notas;
  - `suma_como_venta` (`si` o `no`).
- **Hoja `lineas`:** `numero` del documento, `fecha`, `tipo`, `codigo`, `producto`, `unidad`, `cantidad`, `precio_unitario`, `descuento`, `importe` y `estado_documento`.
- Las notas de crédito llevan sus importes **en negativo** en las dos hojas.
- **Tickets:** como no tienen bases ni IGV desglosados, esas columnas van vacías. El total sí va.

### Detalles comunes

- **Celdas en XLSX:** los importes y cantidades se escriben como números con formato `0.00` y `0.000` (para poder sumarlos en Excel). Los códigos y documentos se escriben como **texto**, para conservar los ceros a la izquierda.
- **Plantilla:** marca esas columnas con formato de texto y trae una hoja `instrucciones` con los valores admitidos.
- **CSV:** UTF-8 con BOM, separado por comas y con punto decimal. Las exportaciones de varias hojas en CSV (productos con lotes y ventas) se entregan en un **ZIP con un CSV por hoja**.
- **Nombres de archivo:** `productos-2026-10-07.xlsx`, `ventas-2026-09-01_2026-09-30.zip`, etc.
- **Fórmulas al exportar:** la neutralización (anteponer `'`) se aplica solo a las columnas de **texto libre** (nombres, direcciones y referencias). Las numéricas no se tocan: un `-12.50` de una nota sigue siendo un número.
- **Lectura al importar:**
  - se usa el valor guardado de cada celda, no su fórmula;
  - los números que llegan como coma flotante se convierten a texto decimal sin artefactos y luego se validan con la precisión de cada campo;
  - las fechas de Excel se convierten a `AAAA-MM-DD`.

## Contrato de API

Todas las rutas van en el grupo `role:company_admin`, con `throttle:data-transfer` (10 por minuto por usuario).

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/v1/exports/products?format=xlsx\|csv&lots=0\|1&search=&type=&status=` | Catálogo con stock. Los filtros son los de `GET /products` |
| GET | `/api/v1/exports/customers?format=&search=` | Clientes |
| GET | `/api/v1/exports/sales?format=&from=&to=&types[]=&statuses[]=` | Ventas. `from` y `to` son obligatorios y abarcan como máximo 12 meses (A-73) |
| GET | `/api/v1/imports/{kind}/template` | Plantilla XLSX. `kind` es `products` o `customers` |
| POST | `/api/v1/imports/{kind}/preview` | Multipart: `file` (XLSX o CSV, hasta 5 MB) y `mode` (`create`, el valor por defecto, o `upsert`). Devuelve la vista previa |
| POST | `/api/v1/imports/{preview}/confirm` | Aplica la vista previa |

Respuesta de la vista previa:

```json
{
  "success": true,
  "data": {
    "id": "9b1e…",
    "kind": "products",
    "mode": "upsert",
    "expires_at": "2026-10-07T15:30:00-05:00",
    "can_confirm": false,
    "summary": { "rows": 120, "create": 100, "update": 18, "unchanged": 1, "errors": 1 },
    "errors": [{ "row": 14, "column": "precio_venta", "message": "Debe tener como máximo 2 decimales." }],
    "warnings": [{ "row": null, "column": "stock_actual", "message": "Columna ignorada." }],
    "changes": [
      { "row": 3, "action": "create", "key": "P-001", "name": "Arroz 1 kg", "stock": "50.000" },
      { "row": 7, "action": "update", "key": "P-007", "name": "Aceite", "fields": { "sale_price": { "from": "3.50", "to": "3.80" } } }
    ]
  }
}
```

- **`row`:** es el número de fila tal como lo ve el usuario en Excel (el encabezado es la fila 1).
- **Errores por archivo:** si el archivo no se puede leer, falta una columna obligatoria, tiene más de 2 000 filas o está vacío, la API responde **422** con el error general. No crea ninguna vista previa.
- **Confirmación:**
  - **200**, con `{ created, updated, entries }`, si todo se aplica;
  - **409**, si los datos cambiaron desde la vista previa;
  - **410**, si la vista previa caducó o ya se confirmó;
  - **422**, si la vista previa tenía errores.
- **Visibilidad:** una vista previa de otro usuario o de otra empresa responde **404** (no se distingue si existe).
- **Errores de exportación:** un rango inválido o de más de 12 meses responde **422**.

## API

### Validación compartida

Las reglas de `ProductRules`, `CustomerRules` y `StoreStockEntryRequest` hoy dependen del FormRequest (`$this->input()`).
- Se extraen a `app/Validation/ProductValidation.php`, `CustomerValidation.php` y `StockEntryValidation.php`, con métodos que reciben los datos y el registro existente.
- Los FormRequest actuales delegan en ellas **sin cambiar su comportamiento**. Las pruebas actuales de productos, clientes y entradas son la red de seguridad.
- El importador valida cada fila con `Validator::make()` y esas mismas reglas.

### Servicios y piezas nuevas

- `app/DataTransfer/Spreadsheet.php`: envoltorio pequeño de OpenSpout.
  - Abre un lector (XLSX o CSV según el contenido, no la extensión) que entrega filas como arreglos de texto.
  - Crea escritores con hojas, BOM en CSV y ZIP cuando hay varias hojas.
  - Contiene `safeText()`, que neutraliza las fórmulas.
- `app/DataTransfer/Exports/{Products,Customers,Sales}Export.php`:
  - las consultas recorren los datos en bloques (`lazyById`) para no cargar todo en memoria;
  - el stock y el costo se suman por producto con `Decimal`;
  - el vendedor sale de un mapa de usuarios de la empresa.
- `app/Support/SalesCounting.php`: la regla de A-54, hoy en `DashboardService::COUNTED`. La usan el panel y la exportación, y así no se desalinean.
- `app/DataTransfer/Imports/{Product,Customer}RowMapper.php`:
  - encabezados a campos;
  - `si`/`no`, `bien`/`servicio` y unidades a sus enums;
  - coma decimal a punto, y fechas.
- `app/DataTransfer/Imports/ImportPreviewer.php`:
  - lee y mapea las filas;
  - detecta códigos o documentos repetidos dentro del archivo (las dos filas son error);
  - valida cada fila;
  - aplica `mode`: con `create`, un registro existente es error; con `upsert`, calcula el diff con `AuditLogger::diff` sin guardar, y si no hay cambios cuenta la fila como `unchanged`;
  - guarda la vista previa.
- `app/DataTransfer/Imports/ImportApplier.php`:
  1. Dentro de `DB::transaction` bloquea la vista previa (`lockForUpdate`) y la marca como confirmada, para que dos confirmaciones simultáneas no apliquen dos veces.
  2. **Vuelve a validar** cada fila contra el estado actual. Si un código ya existe, o un registro cambió desde la vista previa (se compara el `updated_at` guardado), aborta todo con 409.
  3. Aplica cada fila con los servicios: `create` o `update`, y `registerEntry` para el stock inicial.
  4. Registra `import.confirmed` con el tipo, el modo y los totales.
  5. Una violación del índice único (otra petición creó el mismo código entre la validación y el insert) se traduce en 409, y la transacción revierte todo.
- **Auditoría:** `export.products`, `export.customers` y `export.sales`, con los filtros, el formato y las filas exportadas; e `import.confirmed`. Las altas y los cambios usan las acciones de siempre (`product.created`, `product.updated`, `inventory.entry`, `customer.created` y `customer.updated`).
- `app/Models/ImportPreview.php`, con `Prunable`: guarda `id` UUID, `company_id`, `user_id`, `kind`, `mode`, `rows` (JSON con las filas normalizadas, el número de fila y el `updated_at` de cada registro existente), `summary`, `errors`, `warnings`, `expires_at` (30 minutos) y `confirmed_at`.
  - Usa el scope de empresa como el resto de los modelos.
  - `model:prune` diario en `routes/console.php` borra las vistas previas caducadas y las confirmadas con más de 1 día.

### Controladores, rutas y FormRequest

- **Controladores:** `ExportController` (tres acciones, con `StreamedResponse`) e `ImportController` (`template`, `preview` y `confirm`).
- **FormRequest:**
  - `ExportProductsRequest`, `ExportCustomersRequest` y `ExportSalesRequest`, que valida el rango de hasta 12 meses y que `from <= to`;
  - `PreviewImportRequest`, con `file` (`required`, `file`, `max:5120`, `mimes:xlsx,csv,txt`) y `mode`.
- **Filtros de la lista:** los de productos y clientes pasan de los controladores de listado a *scopes* (`Product::scopeListFilter` y `Customer::scopeListFilter`). Así la exportación respeta exactamente el filtro de la pantalla (HU-1, escenario 3).
- **Rutas:** `RateLimiter::for('data-transfer', …)` en `AppServiceProvider`, y las rutas nuevas en `TenantRoutes.php`.

## Frontend

### Feature `features/data-transfer/`

Archivos: `api.ts`, `types.ts` y `routes.ts`, con la ruta `datos` (nombre `data-transfer`, `meta.roles: ['company_admin']` y título «Importar y exportar»).

`views/DataTransferView.vue` tiene tres secciones:

1. **Exportar ventas:** `SalesExportForm.vue`, con:
   - un rango de fechas, que por defecto es el mes actual (fecha de Lima);
   - casillas de tipo y estado;
   - el formato (XLSX o CSV);
   - el botón «Descargar».
2. **Exportar catálogo y clientes:** accesos directos, sin filtros.
3. **Importar:** `ImportWizard.vue`, con estos pasos:
   1. Elegir «Productos» o «Clientes» y descargar la plantilla.
   2. Elegir el archivo y el modo («Solo crear», el valor por defecto, o «Crear y actualizar», con su explicación).
   3. Revisar la vista previa:
      - el resumen;
      - la lista de errores por fila y columna;
      - los cambios en tarjetas, con el antes y el después de cada campo, paginados en el cliente de 50 en 50;
      - los avisos.

      «Confirmar» se deshabilita si hay errores o si la vista previa caducó.
   4. Ver el resultado, con enlaces a «Productos» o «Clientes».

   Un 409 o un 410 vuelve al paso 2 con un mensaje claro.

### Otras piezas

- **Descargas:** las exportaciones usan `downloadFile()` de `core/api/download.ts` (descarga en la web, «Compartir» en Android).
- **Subida:** el archivo se sube con `FormData`, como el logo y el certificado (`bodySerializer`).
- **Botones en otras pantallas:** «Exportar» en `ProductsView` (pasa los filtros actuales y tiene la casilla «con lotes») y en `CustomersView`. Cada feature llama a su propio `api.ts` con `downloadFile`, sin importar `data-transfer`.
- **Menú:** entrada «Importar y exportar» en la barra lateral, solo para el administrador.
- **Diseño:** tokens del tema, campos compartidos (`BaseSelect` y `BaseInput`) y `BaseDialog` para confirmar la importación («Se crearán 100 productos y se actualizarán 18. ¿Continuar?»).

## Estrategia de pruebas

| Requisito / escenario | Prueba | Tipo |
|---|---|---|
| Refactor de reglas | Las pruebas actuales de productos, clientes y entradas siguen en verde sin cambios | Pest |
| RF-001, CE-005 | `DataTransferAccessTest`: el vendedor recibe 403 en las seis rutas, y otra empresa recibe 404 en la vista previa y nunca ve datos ajenos en las exportaciones; además, `TenantRoutes.php` | Pest |
| HU-1, RF-002 | `ProductsExportTest`: se leen las columnas, el stock y el costo del archivo generado (XLSX y CSV, con y sin lotes, y con filtros) | Pest |
| HU-2 | `CustomersExportTest`, incluida la auditoría | Pest |
| HU-3, RF-003, CE-002 | `SalesExportTest`: tickets, anulados, comprobantes en cada estado y notas en negativo. La suma neta de `suma_como_venta = si` es igual al total del panel de inicio en los mismos días. También el límite de 12 meses y los filtros | Pest (SQLite y PostgreSQL) |
| RF-011 | Un nombre `=HYPERLINK(...)` sale neutralizado y un importe negativo sale intacto. Al importar, una celda con fórmula entra como su valor | Pest |
| HU-4 y RF-005 a RF-008 | `ProductImportTest`: modo `create` y `upsert`, repetidos dentro del archivo, servicio con stock, producto existente con stock (aviso), vencimiento obligatorio y comas decimales. El stock inicial crea el lote, el movimiento y la auditoría | Pest |
| HU-5 | `CustomerImportTest`: DNI, CE y RUC (con dígito verificador), ceros a la izquierda y `upsert` de nombre y dirección | Pest |
| CE-003, RF-005 | Un error en la fila 2 000 no guarda nada, y la confirmación con errores responde 422 | Pest |
| Casos límite | Archivo de 2 001 filas, de más de 5 MB, dañado o sin columnas; vista previa caducada (410) o ya confirmada; confirmación doble simultánea | Pest |
| Conflicto | Un producto creado por otra petición entre la vista previa y la confirmación da 409 y no aplica nada | Pest en `tests/Integration` (sin transacción envolvente, PostgreSQL) |
| CE-004 | Exportar y luego hacer la vista previa en `upsert` del mismo archivo da 0 cambios | Pest |
| Contrato | `ImportPreviewResourceTest` | Pest |
| Frontend | `ImportWizard` (pasos, errores, «Confirmar» deshabilitado, 409 y 410), `SalesExportForm` (rango por defecto y validación) y los botones de exportar en «Productos» y «Clientes» | Vitest |
| CE-001, RF-013 | 500 productos con stock importados de punta a punta en menos de 2 minutos, y recorrido a 360 px en la web y en el emulador Android (descarga y «Compartir», y elegir un archivo) | Manual (`verificacion.md`) |

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Confirmar 2 000 filas pase por los servicios uno a uno (auditoría y entradas) y tarde | Se mide con 2 000 productos con stock en PostgreSQL. Si pasa de unos 30 s, se agrupan las inserciones de auditoría (sigue siendo una fila por cambio) sin dejar los servicios |
| Exportaciones grandes de ventas (12 meses) | *Streaming* con `lazyById`, sin cargar todo, y una prueba de memoria con 20 000 líneas. Si alguna empresa lo necesita, se pasa a una cola sin cambiar la interfaz (fuera de esta spec) |
| Excel cambia códigos o DNI a número y pierde los ceros | La plantilla marca esas columnas como texto. Si aun así llegan con menos dígitos, el error lo explica («El DNI debe tener 8 dígitos; revisa que la columna esté como texto en Excel») |
| Diferencias de redondeo por los flotantes de las hojas de cálculo | Conversión a texto decimal con la precisión de cada campo antes de validar, y las reglas `decimal:0,N` de siempre |
| Que el refactor de las reglas cambie el alta manual | Se hace como primera tarea, sola, y debe pasar con las suites existentes sin tocarlas |
| El selector de archivos en el WebView de Android | Capacitor lo admite con `<input type="file">`. Se verifica en el emulador al inicio del frontend, y si falla se usa un plugin para elegir archivos (lo decidiría el usuario) |
| OpenSpout 4.x deja de recibir mantenimiento | La 5.x se adopta cuando el mínimo de PHP suba a 8.4. El envoltorio `Spreadsheet` aísla el cambio |
