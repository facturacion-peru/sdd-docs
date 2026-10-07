# Tareas 014: Exportar e importar datos

**Estado:** Aprobadas (2026-10-07) · **Fecha:** 2026-10-07 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior. 🔒 = requiere confirmación explícita del usuario.

## Fase 1 · Preparación

- [ ] **T001** Rama `014-exportar-importar` desde `main` en la API y en el frontend — Verificación: `git branch --show-current` en ambos
- [ ] **T002** `composer require openspout/openspout:^4.32` (revisar la licencia, las dependencias y `composer audit`); agregar `xmlreader` a `PHP_EXTENSIONS` en la CI — Verificación: `composer show openspout/openspout`, suites en verde y `actionlint`
- [ ] **T003** Comprobar en el emulador Android que `<input type="file">` abre el selector de archivos de la app (pantalla de prueba temporal, sin commit) — Verificación: se elige un `.xlsx` y llega su nombre y tamaño. Si falla, detenerse y consultar al usuario (riesgo del plan)

## Fase 2 · API: base compartida (sin cambiar comportamiento)

- [ ] **T004** Extraer las reglas a `app/Validation/ProductValidation.php`, `CustomerValidation.php` y `StockEntryValidation.php`; los FormRequest actuales delegan en ellas — Verificación: suites SQLite y PostgreSQL en verde **sin tocar ninguna prueba existente**
- [ ] **T005** [P] Extraer la regla de A-54 a `app/Support/SalesCounting.php` y usarla en `DashboardService` — Verificación: pruebas del panel en verde
- [ ] **T006** [P] Pasar los filtros de lista a `Product::scopeListFilter` y `Customer::scopeListFilter`, y usarlos en los controladores de listado — Verificación: pruebas de listado en verde
- [ ] **T007** `app/DataTransfer/Spreadsheet.php` (lector XLSX/CSV detectado por contenido, escritores con hojas, BOM y ZIP, `safeText()`), con sus pruebas unitarias primero:
  - fórmulas neutralizadas solo en texto;
  - celdas con fórmula leídas como valor;
  - flotantes convertidos a decimal exacto;
  - fechas de Excel convertidas;
  - CSV con tildes.

  Verificación: pruebas en verde

## Fase 3 · API: exportaciones (HU-1, HU-2, HU-3)

- [ ] **T008** Pruebas de acceso y aislamiento de las rutas de exportación, y rutas en `TenantRoutes.php` (deben fallar) — Verificación: en rojo
- [ ] **T009** Pruebas `ProductsExportTest` (columnas, stock y costo, con y sin lotes, filtros, XLSX y CSV/ZIP, auditoría) (deben fallar) — Verificación: en rojo
- [ ] **T010** `ProductsExport`, `ExportProductsRequest`, `ExportController::products`, la ruta y el límite `data-transfer` — Verificación: T008 (productos) y T009 en verde
- [ ] **T011** [P] Pruebas de `CustomersExportTest`; `CustomersExport` y su ruta — Verificación: en verde
- [ ] **T012** Pruebas de `SalesExportTest` (deben fallar):
  - tickets y anulados;
  - comprobantes en cada estado;
  - notas en negativo;
  - hoja de líneas;
  - filtros de tipo y estado;
  - límite de 12 meses y `from > to`;
  - borde de día en Lima;
  - CE-002: la suma neta de `suma_como_venta = si` iguala al panel de inicio.

  Verificación: en rojo
- [ ] **T013** `SalesExport`, `ExportSalesRequest` y su ruta — Verificación: T012 en verde en SQLite y PostgreSQL
- [ ] **T014** Prueba de memoria: exportar 20 000 líneas de venta sin superar un tope razonable de memoria (medido con `memory_get_peak_usage`) — Verificación: en verde y la cifra anotada en esta tarea

## Fase 4 · API: importaciones (HU-4, HU-5)

- [ ] **T015** Migración `import_previews` y modelo `ImportPreview` (scope de empresa, `Prunable`, caducidad de 30 minutos) y `model:prune` diario en `routes/console.php` — Verificación: migración en SQLite y PostgreSQL, y prueba del *prune*
- [ ] **T016** Plantillas: `GET /imports/{kind}/template` (columnas de texto, hoja `instrucciones`), con sus pruebas — Verificación: en verde; la plantilla se abre en LibreOffice sin avisos
- [ ] **T017** Pruebas de la vista previa de productos (deben fallar):
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
- [ ] **T018** `ProductRowMapper`, `ImportPreviewer`, `PreviewImportRequest`, `ImportController::preview` y las rutas — Verificación: T017 en verde
- [ ] **T019** Pruebas de la confirmación de productos (deben fallar):
  - crea productos, lotes, movimientos y auditoría como el alta manual;
  - `upsert` cambia solo lo indicado y no toca el stock;
  - con errores → 422;
  - caducada o ya confirmada → 410;
  - de otro usuario o de otra empresa → 404;
  - el vendedor → 403.

  Verificación: en rojo
- [ ] **T020** `ImportApplier` e `ImportController::confirm` — Verificación: T019 en verde
- [ ] **T021** `tests/Integration/DataTransfer/ImportConflictTest` (PostgreSQL), en dos casos: un producto creado por otra petición entre la vista previa y la confirmación da 409, y una confirmación doble simultánea solo aplica una vez. En los dos, no se aplica nada parcial. Comprobar con una mutación (sin la revalidación, o sin el `lockForUpdate`) que la prueba falla — Verificación: en verde, y en rojo con la mutación
- [ ] **T022** [P] Pruebas e implementación de clientes:
  - `CustomerRowMapper`;
  - DNI, CE y RUC (con dígito verificador);
  - ceros a la izquierda;
  - `upsert` solo de nombre y dirección.

  Verificación: en verde
- [ ] **T023** CE-003 y CE-004: un error en la fila 2 000 no guarda nada; exportar y luego hacer la vista previa en `upsert` del mismo archivo da 0 cambios (productos y clientes) — Verificación: en verde
- [ ] **T024** Prueba de contrato `ImportPreviewResourceTest` — Verificación: en verde
- [ ] **T025** Medir la confirmación de 2 000 productos con stock en PostgreSQL. Si pasa de unos 30 s, aplicar la mitigación del plan — Verificación: tiempo anotado en esta tarea
- [ ] **T026** `php artisan openapi:generate`, suites SQLite y PostgreSQL y Pint — Verificación: códigos de salida 0

## Fase 5 · Frontend

- [ ] **T027** `pnpm api:types`; `features/data-transfer/types.ts` y `api.ts` (exportaciones con `downloadFile`, subida con `FormData`) — Verificación: `pnpm type-check`
- [ ] **T028** Pruebas de `SalesExportForm` (deben fallar): rango por defecto del mes en Lima, validación del rango y descarga con los filtros — Verificación: en rojo
- [ ] **T029** `SalesExportForm` — Verificación: T028 en verde
- [ ] **T030** Pruebas de `ImportWizard` (deben fallar):
  - los cuatro pasos y la plantilla;
  - modo por defecto «Solo crear»;
  - errores por fila y columna;
  - «Confirmar» deshabilitado con errores;
  - diálogo de confirmación;
  - 409 y 410 vuelven al paso 2 con un mensaje;
  - resultado.

  Verificación: en rojo
- [ ] **T031** `ImportWizard` y componentes de la vista previa (resumen, errores, cambios en tarjetas paginadas de 50 en 50, avisos) — Verificación: T030 en verde
- [ ] **T032** `DataTransferView`, la ruta `datos` (solo administrador) y la entrada «Importar y exportar» en la barra lateral, con pruebas de acceso por rol — Verificación: pruebas en verde
- [ ] **T033** [P] Botón «Exportar» en `ProductsView` (filtros actuales y «con lotes») y en `CustomersView`, con pruebas — Verificación: en verde
- [ ] **T034** Comprobaciones del frontend: `pnpm type-check && pnpm lint && pnpm test:unit --run && pnpm build-only` — Verificación: códigos de salida 0

## Cierre

- [ ] **T035** Recorrido de extremo a extremo con la API local y las cuentas demo (`verificacion.md`):
  - exportar los tres tipos de dato en XLSX y CSV y abrirlos en LibreOffice;
  - CE-001: importar 500 productos con stock en menos de 2 minutos;
  - importación con errores y reintento;
  - el vendedor sin acceso.

  Verificación: tiempos y resultados anotados
- [ ] **T036** Pantallas a 360 px y en escritorio, y en el emulador Android (descarga y «Compartir», elegir archivo e importar) — Verificación: capturas revisadas
- [ ] **T037** Comprobaciones del principio VI en verde (API SQLite y PostgreSQL, Pint; frontend completo) — Verificación: códigos de salida 0 y cifras anotadas
- [ ] **T038** Documentación:
  - README de la API (dependencia y extensiones) y del frontend (módulo nuevo);
  - trampas nuevas en `AGENTS.md`, si las hay;
  - bitácora;
  - estado en `docs/specs/README.md`.

  Verificación: revisada
- [ ] **T039** 🔒 Merge local de `014-exportar-importar` a `main` en ambos repositorios y commit de la raíz — Verificación: suites en verde sobre `main`
