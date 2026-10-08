# Tareas 014: Exportar e importar datos

**Estado:** Completadas (2026-10-07) · **Fecha:** 2026-10-07 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior. 🔒 = requiere confirmación explícita del usuario.

## Fase 1 · Preparación

- [x] **T001** Rama `014-exportar-importar` desde `main` en la API y en el frontend — Verificación: `git branch --show-current` en ambos
- [x] **T002** `composer require openspout/openspout:^4.32` (revisar la licencia, las dependencias y `composer audit`); agregar `xmlreader` a `PHP_EXTENSIONS` en la CI — Verificación: `composer show openspout/openspout`, suites en verde y `actionlint` Hecho: solo entró `openspout/openspout` 4.32.0 (MIT); `composer audit` sigue con los 21 avisos previos, ninguno nuevo.
- [x] **T003** Comprobar en el emulador Android que `<input type="file">` abre el selector de archivos de la app (pantalla de prueba temporal, sin commit) — Verificación: se elige un `.xlsx` y llega su nombre y tamaño. Si falla, detenerse y consultar al usuario (riesgo del plan) Hecho (2026-10-07, emulador Android 15): el selector de archivos de Android se abre desde la app (Capacitor 8 implementa `onShowFileChooser`) y la página recibe el `.xlsx` con nombre, tamaño, tipo MIME y contenido legible. Usar en `accept` las extensiones **y** los tipos MIME.

## Fase 2 · API: base compartida (sin cambiar comportamiento)

- [x] **T004** Extraer las reglas a `app/Validation/ProductValidation.php`, `CustomerValidation.php` y `StockEntryValidation.php`; los FormRequest actuales delegan en ellas — Verificación: suites SQLite y PostgreSQL en verde **sin tocar ninguna prueba existente** Hecho: suites sin cambios en verde (655 SQLite, 666 PostgreSQL) y `public/openapi.json` idéntico.
- [x] **T005** [P] Extraer la regla de A-54 a `app/Support/SalesCounting.php` y usarla en `DashboardService` — Verificación: pruebas del panel en verde
- [x] **T006** [P] Pasar los filtros de lista a `Product::scopeListFilter` y `Customer::scopeListFilter`, y usarlos en los controladores de listado — Verificación: pruebas de listado en verde
- [x] **T007** `app/DataTransfer/Spreadsheet.php` (lector XLSX/CSV detectado por contenido, escritores con hojas, BOM y ZIP, `safeText()`), con sus pruebas unitarias primero:
  - fórmulas neutralizadas solo en texto;
  - celdas con fórmula leídas como valor;
  - flotantes convertidos a decimal exacto;
  - fechas de Excel convertidas;
  - CSV con tildes.

  Verificación: pruebas en verde Hecho. Hallazgo: el lector de OpenSpout devuelve como «fórmula» sin valor calculado cualquier texto que empieza con `=`; en ese caso se usa el texto. En XLSX los textos van como celdas de texto (nunca fórmula) y el apóstrofo de RF-011 se aplica solo en CSV (la lectura lo quita, para CE-004).

## Fase 3 · API: exportaciones (HU-1, HU-2, HU-3)

- [x] **T008** Pruebas de acceso y aislamiento de las rutas de exportación, y rutas en `TenantRoutes.php` (deben fallar) — Verificación: en rojo Hecho: `TenantRoutes.php` admite `query` (las exportaciones van en CSV para buscar rastros de B en el contenido) y `expectNoTraceOfB` lee las descargas en streaming.
- [x] **T009** Pruebas `ProductsExportTest` (columnas, stock y costo, con y sin lotes, filtros, XLSX y CSV/ZIP, auditoría) (deben fallar) — Verificación: en rojo
- [x] **T010** `ProductsExport`, `ExportProductsRequest`, `ExportController::products`, la ruta y el límite `data-transfer` — Verificación: T008 (productos) y T009 en verde
- [x] **T011** [P] Pruebas de `CustomersExportTest`; `CustomersExport` y su ruta — Verificación: en verde
- [x] **T012** Pruebas de `SalesExportTest` (deben fallar):
  - tickets y anulados;
  - comprobantes en cada estado;
  - notas en negativo;
  - hoja de líneas;
  - filtros de tipo y estado;
  - límite de 12 meses y `from > to`;
  - borde de día en Lima;
  - CE-002: la suma neta de `suma_como_venta = si` iguala al panel de inicio.

  Verificación: en rojo
- [x] **T013** `SalesExport`, `ExportSalesRequest` y su ruta — Verificación: T012 en verde en SQLite y PostgreSQL Hecho: verde en SQLite y PostgreSQL.
- [x] **T014** Prueba de memoria: exportar 20 000 líneas de venta sin superar un tope razonable de memoria (medido con `memory_get_peak_usage`) — Verificación: en verde y la cifra anotada en esta tarea Hecho: 20 000 líneas → ~27 MB adicionales en XLSX (8,4 s) y ~25 MB en CSV (3,8 s); con 40 000 líneas la memoria es la misma (crece solo el tiempo).

## Fase 4 · API: importaciones (HU-4, HU-5)

- [x] **T015** Migración `import_previews` y modelo `ImportPreview` (scope de empresa, `Prunable`, caducidad de 30 minutos) y `model:prune` diario en `routes/console.php` — Verificación: migración en SQLite y PostgreSQL, y prueba del *prune* Hecho: `prunable()` usa `withoutTenancy()` (el scheduler corre sin empresa); `model:prune` diario a las 02:30.
- [x] **T016** Plantillas: `GET /imports/{kind}/template` (columnas de texto, hoja `instrucciones`), con sus pruebas — Verificación: en verde; la plantilla se abre en LibreOffice sin avisos Hecho. Hallazgo: OpenSpout no escribe filas de celdas vacías; las columnas de texto llevan una cadena vacía con formato «@» (numFmtId 49). LibreOffice abre la plantilla y lee sus dos hojas.
- [x] **T017** Pruebas de la vista previa de productos (deben fallar):
  - `create` y `upsert`;
  - repetidos dentro del archivo;
  - servicio con stock;
  - producto existente con stock (aviso);
  - vencimiento obligatorio;
  - coma decimal y precisiones;
  - columnas que faltan o sobran;
  - 2 001 filas, más de 5 MB, archivo dañado o vacío;
  - `unchanged`.

  Verificación: en rojo
- [x] **T018** `ProductRowMapper`, `ImportPreviewer`, `PreviewImportRequest`, `ImportController::preview` y las rutas — Verificación: T017 en verde Hecho: la vista previa responde 201. Un archivo dañado lo rechaza ya la regla `mimes` (detecta el tipo por el contenido).
- [x] **T019** Pruebas de la confirmación de productos (deben fallar):
  - crea productos, lotes, movimientos y auditoría como el alta manual;
  - `upsert` cambia solo lo indicado y no toca el stock;
  - con errores → 422;
  - caducada o ya confirmada → 410;
  - de otro usuario o de otra empresa → 404;
  - el vendedor → 403.

  Verificación: en rojo
- [x] **T020** `ImportApplier` e `ImportController::confirm` — Verificación: T019 en verde Hecho. Hallazgo de PostgreSQL: `audit_logs.auditable_id` es entero y la vista previa usa UUID; `import.confirmed` se audita sin modelo y con el id de la vista previa en los cambios.
- [x] **T021** `tests/Integration/DataTransfer/ImportConflictTest` (PostgreSQL), en dos casos: un producto creado por otra petición entre la vista previa y la confirmación da 409, y una confirmación doble simultánea solo aplica una vez. En los dos, no se aplica nada parcial. Comprobar con una mutación (sin la revalidación, o sin el `lockForUpdate`) que la prueba falla — Verificación: en verde, y en rojo con la mutación Hecho: verde en PostgreSQL; con la mutación (sin `lockForUpdate`) la segunda confirmación recibe 409 en vez de 410 y la prueba falla.
- [x] **T022** [P] Pruebas e implementación de clientes:
  - `CustomerRowMapper`;
  - DNI, CE y RUC (con dígito verificador);
  - ceros a la izquierda;
  - `upsert` solo de nombre y dirección.

  Verificación: en verde
- [x] **T023** CE-003 y CE-004: un error en la fila 2 000 no guarda nada; exportar y luego hacer la vista previa en `upsert` del mismo archivo da 0 cambios (productos y clientes) — Verificación: en verde
- [x] **T024** Prueba de contrato `ImportPreviewResourceTest` — Verificación: en verde
- [x] **T025** Medir la confirmación de 2 000 productos con stock en PostgreSQL. Si pasa de unos 30 s, aplicar la mitigación del plan — Verificación: tiempo anotado en esta tarea Hecho: 2 000 productos con stock en PostgreSQL: vista previa 6,0 s y confirmación **35,8 s → 13,3 s**. Causa: la revalidación de código único de cada alta (17,6 s en una consulta por fila sobre una tabla que crece en la transacción). Mitigación: una sola consulta para todas las altas (`assertStillNew`); el índice único cubre lo que ocurra después (409). No hizo falta agrupar la auditoría.
- [x] **T026** `php artisan openapi:generate`, suites SQLite y PostgreSQL y Pint — Verificación: códigos de salida 0 Hecho: API 757 pruebas en SQLite (13 omitidas: PostgreSQL/pcntl) y 770 en PostgreSQL, Pint limpio.

## Fase 5 · Frontend

- [x] **T027** `pnpm api:types`; `features/data-transfer/types.ts` y `api.ts` (exportaciones con `downloadFile`, subida con `FormData`) — Verificación: `pnpm type-check` Hecho: el generador de OpenAPI tipa los parámetros de ruta como número; `kind` y el UUID se pasan con un *cast* comentado.
- [x] **T028** Pruebas de `SalesExportForm` (deben fallar): rango por defecto del mes en Lima, validación del rango y descarga con los filtros — Verificación: en rojo
- [x] **T029** `SalesExportForm` — Verificación: T028 en verde Hecho: `todayInLima()` en `shared/utils/format.ts`; los estados se agrupan como «Todos», «Solo los que suman como venta» y «Solo anulados, rechazados y descartados».
- [x] **T030** Pruebas de `ImportWizard` (deben fallar):
  - los cuatro pasos y la plantilla;
  - modo por defecto «Solo crear»;
  - errores por fila y columna;
  - «Confirmar» deshabilitado con errores;
  - diálogo de confirmación;
  - 409 y 410 vuelven al paso 2 con un mensaje;
  - resultado.

  Verificación: en rojo
- [x] **T031** `ImportWizard` y componentes de la vista previa (resumen, errores, cambios en tarjetas paginadas de 50 en 50, avisos) — Verificación: T030 en verde
- [x] **T032** `DataTransferView`, la ruta `datos` (solo administrador) y la entrada «Importar y exportar» en la barra lateral, con pruebas de acceso por rol — Verificación: pruebas en verde
- [x] **T033** [P] Botón «Exportar» en `ProductsView` (filtros actuales y «con lotes») y en `CustomersView`, con pruebas — Verificación: en verde Hecho: el diálogo es un componente compartido (`shared/ui/ExportButton.vue`).
- [x] **T034** Comprobaciones del frontend: `pnpm type-check && pnpm lint && pnpm test:unit --run && pnpm build-only` — Verificación: códigos de salida 0 Hecho: 281 pruebas, tipos, lint y build en verde.

## Cierre

- [x] **T035** Recorrido de extremo a extremo con la API local y las cuentas demo (`verificacion.md`):
  - exportar los tres tipos de dato en XLSX y CSV y abrirlos en LibreOffice;
  - CE-001: importar 500 productos con stock en menos de 2 minutos;
  - importación con errores y reintento;
  - el vendedor sin acceso.

  Verificación: tiempos y resultados anotados Hecho: 15/15 por el proxy de Vite; CE-001 en 5,3 s (vista previa 1,5 s y confirmación 3,8 s). Ver [verificación](verificacion.md).
- [x] **T036** Pantallas a 360 px y en escritorio, y en el emulador Android (descarga y «Compartir», elegir archivo e importar) — Verificación: capturas revisadas Hecho: corrigió un desborde a 360 px (rejilla sin `grid-cols-1`), las fechas cortadas y un espacio. En el emulador: «Compartir» al exportar y selector del teléfono al importar.
- [x] **T037** Comprobaciones del principio VI en verde (API SQLite y PostgreSQL, Pint; frontend completo) — Verificación: códigos de salida 0 y cifras anotadas Hecho: API 757 (SQLite) y 770 (PostgreSQL), Pint; frontend 281 pruebas, tipos, lint y build.
- [x] **T038** Documentación:
  - README de la API (dependencia y extensiones) y del frontend (módulo nuevo);
  - trampas nuevas en `AGENTS.md`, si las hay;
  - bitácora;
  - estado en `docs/specs/README.md`.

  Verificación: revisada
- [x] **T039** 🔒 Merge local de `014-exportar-importar` a `main` en ambos repositorios y commit de la raíz — Verificación: suites en verde sobre `main` Hecho (2026-10-07): merge `--no-ff` en los dos repositorios; sobre `main`, API 757 (SQLite) y 770 (PostgreSQL), Pint, y frontend 281 pruebas, tipos, lint y build. Subido a GitHub.
