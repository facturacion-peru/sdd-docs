# Tareas 011: Panel de inicio

**Estado:** Implementada (2026-10-03) · **Fecha:** 2026-10-03 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior.

## Fase 1 · Preparación

- [x] **T001** Rama `011-panel-inicio` en la API — Verificación: `git branch --show-current`
- [x] **T002** Confirmar índices `(company_id, issued_at)` en `tickets` y `sales_documents` — Verificación: migraciones existentes (no hace falta una nueva)

## Fase 2 · API (HU-1, HU-2, HU-3)

- [x] **T003** Extraer `InventoryAlertService` de `AlertController` — Verificación: pruebas actuales de alertas en verde
- [x] **T004** Pruebas de `GET /api/v1/dashboard` (deben fallar): reglas de A-54, alcance de A-53, borde de día en Lima, 7 días con ceros, por atender, últimas 5, empresa vacía — Verificación: `php artisan test --filter=Dashboard` en rojo
- [x] **T005** Ruta en `TenantRoutes.php` (`list`) — Verificación: `TenantRoutesCoverageTest` y `CrossTenantAccessTest` en verde tras T006
- [x] **T006** `DashboardService` y `DashboardController` — Verificación: T004 en verde
- [x] **T007** Prueba de contrato `DashboardResourceTest` — Verificación: en verde
- [x] **T008** `php artisan openapi:generate`, suites SQLite y PostgreSQL y Pint — Verificación: códigos de salida 0

## Fase 3 · Frontend

- [x] **T009** `pnpm api:types`; `features/dashboard/types.ts` y `api.ts` — Verificación: `pnpm type-check`
- [x] **T010** Pruebas de los componentes y de `HomeView` (deben fallar) — Verificación: en rojo
- [x] **T011** `TodaySummary`, `WeekChart`, `AttentionList` y `RecentSales` — Verificación: T010 de componentes en verde
- [x] **T012** `HomeView` compone el panel (carga, error, vacío, rol) — Verificación: T010 en verde
- [x] **T013** `SalesDocumentsView` aplica `?status=` de la URL (prueba primero) — Verificación: prueba en verde

## Cierre

- [x] **T014** Recorrido a 360 px y en escritorio con las cuentas demo (administrador y vendedor) — Verificación: capturas revisadas. Corrigió un desbordamiento horizontal a 360 px (columna implícita del grid con texto truncado: `grid-cols-1`) y el alto del gráfico en escritorio
- [x] **T015** Comprobaciones del principio VI en verde: API 639 (SQLite) y 650 (PostgreSQL), Pint; frontend 185 pruebas, tipos, lint y build
- [x] **T016** Bitácora de `AGENTS.md` y estado en `docs/specs/README.md`
