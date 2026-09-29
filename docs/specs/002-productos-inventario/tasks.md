# Tareas 002: Productos, lotes y movimientos de inventario

**Estado:** Borrador (pendiente de aprobación) · **Fecha:** 2026-09-28 · **Plan:** [plan.md](./plan.md) (aprobado)

> Mismas convenciones que la 001: tareas pequeñas con su verificación; `[P]` = en paralelo con la anterior; 🔒 = confirmación del usuario al ejecutarla.
> **Pruebas primero:** las tareas «Escribir pruebas» terminan en rojo y la siguiente las pone en verde.
> En toda tarea que toque stock, la verificación incluye `composer test:pgsql` además de `php artisan test`.

## Fase 1 · Preparación

- [ ] **T001** Crear la rama `002-productos-inventario` desde `main` en la API y en el frontend. — Verificación: `git branch --show-current` en ambos.
- [ ] **T002** Declarar `ext-bcmath` en `require` de `composer.json`. — Verificación: `composer validate` sin errores; `composer check-platform-reqs` lista `ext-bcmath` como presente.

## Fase 2 · Fundamentos

- [ ] **T010** Crear los *enums* `ProductType`, `UnitOfMeasure` (con `allowsDecimals()` y etiqueta), `IgvAffectation` (código SUNAT y etiqueta), `MovementType` y `AdjustmentReason`, con pruebas unitarias de sus reglas. — Verificación: `php artisan test --filter=Enums` en verde.
- [ ] **T011** Migraciones: `products`, `product_lots` (CHECK `remaining_quantity >= 0` solo en PostgreSQL), `inventory_movements` y `companies.expiry_warning_days`. — Verificación: `test` y `test:pgsql` en verde; `php artisan migrate` en local sin errores.
- [ ] **T012** Modelos `Product`, `ProductLot` e `InventoryMovement` con `BelongsToCompany`, casts `decimal`, relaciones y *factories*. — Verificación: `tinker` crea un producto con lote por *factory*; `pint --test`.
- [ ] **T013** Escribir pruebas `Inventory/MovementImmutabilityTest`: `update` y `delete` de un movimiento lanzan excepción; `reverses_id` es único. — Verificación: en rojo.
- [ ] **T014** Inmutabilidad de `InventoryMovement` (mismo patrón que `AuditLog`). — Verificación: T013 en verde.
- [ ] **T015** [P] Escribir la prueba del CHECK en PostgreSQL: un `UPDATE` directo que deja `remaining_quantity < 0` falla (se omite en SQLite). — Verificación: en verde con `test:pgsql` y omitida con `test`.

## Fase 3 · HU-1 Catálogo de productos (P1)

- [ ] **T020** Escribir pruebas `Inventory/ProductCatalogTest`:
  - HU-1.1: alta con todos los campos.
  - HU-1.2: código duplicado en la empresa, pero el mismo código se permite en otra.
  - HU-1.3: un producto desactivado no aparece para vender y conserva su historial.
  - HU-1.4 y RF-022: el vendedor ve precios y stock, pero no el costo, y no puede crear ni editar.
  - Búsqueda por código o nombre, filtros y paginación.
  - Auditoría de `product.created` y `product.updated`.
  - Verificación: en rojo.
- [ ] **T021** Implementar:
  - `GET/POST /products`, `GET/PATCH /products/{product}`.
  - `ProductService`, `StoreProductRequest` y `UpdateProductRequest`.
  - `ProductResource` (costo solo para el administrador).
  - `GET /catalogs/inventory`.
  - Verificación: T020 en verde.
- [ ] **T022** Escribir y poner en verde `Inventory/ProductEdgeCasesTest`:
  - No se pasa de bien a servicio con stock.
  - Las cantidades con decimales solo valen si la unidad lo permite.
  - Activar «controla vencimiento» con lotes sin fecha los deja sin fecha y con la salida más tardía.
  - Desactivar con stock se permite y avisa.
  - Verificación: en verde.

## Fase 4 · HU-2 Entrada de stock por lote (P1)

- [ ] **T030** Escribir pruebas `Inventory/StockEntryTest`:
  - HU-2.1: entrada completa que crea lote y movimiento y sube el stock.
  - HU-2.2: número de lote generado `L-AAAAMMDD-id`.
  - HU-2.3: vencimiento obligatorio y futuro si el producto lo controla.
  - HU-2.4: los servicios no admiten entradas.
  - HU-2.5: cantidad cero o negativa rechazada.
  - Número de lote duplicado en el mismo producto rechazado.
  - Vendedor rechazado y auditoría `inventory.entry`.
  - Verificación: en rojo.
- [ ] **T031** Implementar `InventoryService::registerEntry`, `POST /products/{product}/entries`, `StoreStockEntryRequest`, `GET /products/{product}/lots` y `LotResource`. — Verificación: T030 en verde en SQLite y PostgreSQL.

## Fase 5 · HU-3 Salida de stock al vender (P1)

- [ ] **T040** Escribir pruebas `Inventory/ConsumeStockTest`:
  - HU-3.1: FEFO por fecha de vencimiento.
  - HU-3.2: FIFO por fecha de ingreso.
  - HU-3.3: una venta consume varios lotes, con un movimiento por lote.
  - HU-3.4: stock insuficiente lanza `InsufficientStock` con el disponible y no descuenta nada.
  - HU-3.5: los lotes vencidos no cuentan.
  - Los servicios no descuentan.
  - Llamar sin transacción abierta lanza `LogicException` (RF-017).
  - Los movimientos guardan el documento de origen.
  - Verificación: en rojo.
- [ ] **T041** Implementar `InventoryService::consume` con bloqueo de la fila del producto y aritmética `bcmath`. — Verificación: T040 en verde en SQLite y PostgreSQL.
- [ ] **T042** Escribir `Inventory/ConcurrentConsumptionTest` (solo PostgreSQL, `pcntl_fork`, sin transacción envolvente, base limpiada al terminar): N procesos compiten por un stock menor que la demanda total. — Verificación: el total vendido es igual al stock inicial, ningún lote queda en negativo y el resto recibe `InsufficientStock`. Pasa en `test:pgsql` y se omite en `test`.

## Fase 6 · HU-4 Ajustes y correcciones (P2)

- [ ] **T050** Escribir pruebas `Inventory/AdjustAndReverseTest`:
  - HU-4.1: ajuste positivo y negativo con motivo.
  - HU-4.2: reversión enlazada al original, que se conserva intacto.
  - HU-4.3: un ajuste que deja el lote negativo se rechaza.
  - HU-4.4: el vendedor no ajusta ni revierte.
  - No se revierte dos veces.
  - No se revierte una entrada cuyo lote ya se consumió.
  - Auditoría `inventory.adjustment` e `inventory.reversal`.
  - Verificación: en rojo.
- [ ] **T051** Implementar `InventoryService::adjust` y `reverse`, `POST /lots/{lot}/adjustments` y `POST /movements/{movement}/reverse`. — Verificación: T050 en verde en SQLite y PostgreSQL.

## Fase 7 · HU-5 Historial y alertas (P2)

- [ ] **T060** Escribir pruebas `Inventory/MovementsAndAlertsTest`:
  - HU-5.1: historial paginado con tipo, lote, cantidad, saldos, responsable y origen.
  - HU-5.2: stock bajo cuando el disponible es igual o menor que `min_stock`.
  - HU-5.3: lotes que vencen dentro de `expiry_warning_days`.
  - `PATCH /company` acepta `expiry_warning_days` entre 1 y 365.
  - El vendedor no ve historial ni alertas.
  - Verificación: en rojo.
- [ ] **T061** Implementar `GET /products/{product}/movements`, `GET /inventory/alerts`, `MovementResource` y el campo en `UpdateCompanyRequest` y `CompanyResource`. — Verificación: T060 en verde.

## Fase 8 · Consistencia, aislamiento y contrato

- [ ] **T070** Escribir `Inventory/StockConsistencyTest` (CE-001): secuencia aleatoria reproducible de 300 operaciones. — Verificación: al final, para todo lote, saldo = suma de movimientos, y para todo producto, stock = suma de lotes. En verde en SQLite y PostgreSQL.
- [ ] **T071** Añadir las rutas nuevas a `TenantRoutes.php` (`resource`, `list` o `reference`) y ampliar los datos de B en `CrossTenantAccessTest` (producto, lote y movimiento de B). — Verificación: `TenantRoutesCoverageTest` y `CrossTenantAccessTest` en verde.
- [ ] **T072** [P] Pruebas de contrato: `ProductResourceTest` y `LotResourceTest` (variantes admin y vendedor), `MovementResourceTest` y `CompanyResourceTest` actualizada. — Verificación: en verde.
- [ ] **T073** `php artisan openapi:generate` y `DevelopmentSeeder` con productos demo: bienes con y sin vencimiento y un servicio, con lotes. — Verificación: el OpenAPI incluye las 11 rutas nuevas; `migrate:fresh --seed` en local (autorizado) deja el catálogo demo.

## Fase 9 · Frontend

- [ ] **T080** `pnpm api:types`; tipos en `features/inventory/types.ts`; `features/inventory/api.ts`. — Verificación: `pnpm type-check`.
- [ ] **T081** Catálogo `/productos`: búsqueda, filtros, tarjetas con stock y disponible; desactivados ocultos por defecto; sin costo para el vendedor. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T082** Formulario de producto (alta y edición): los decimales dependen de la unidad y los errores 422 aparecen por campo. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T083** Detalle del producto: lotes; historial, ajuste y reversión para el administrador, con los diálogos accesibles de `shared/ui`. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T084** Entrada rápida `/productos/:id/entrada`: solo la cantidad obligatoria; fecha de hoy por defecto; vencimiento obligatorio si el producto lo controla. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T085** [P] Alertas `/inventario/alertas`, navegación (Productos para todos, Alertas para el administrador) y campo de días de aviso en «Mi empresa». Pruebas incluidas. — Verificación: pruebas en verde.

## Cierre

- [ ] **T090** API: `php artisan test`, `composer test:pgsql` y `vendor/bin/pint --test` en verde. — Verificación: salidas adjuntas.
- [ ] **T091** Frontend: `pnpm type-check`, `pnpm lint`, `pnpm test:unit --run` y `pnpm build-only` en verde. — Verificación: salidas adjuntas.
- [ ] **T092** Verificación de extremo a extremo por el proxy de Vite (alta, entrada, consumo por servicio, ajuste, reversión, alertas, vendedor sin costo) y capturas a 360 px. Se cronometra la entrada de stock (CE-003). Resultado en `verificacion.md`. — Verificación: `verificacion.md` con resultado por paso.
- [ ] **T093** Documentación: README de la API y del frontend, `AGENTS.md` (bitácora y problemas conocidos) y spec 002 → estado final en `specs/README.md`. — Verificación: enlaces válidos.
- [ ] **T094** 🔒 Merge local de `002-productos-inventario` a `main` en ambos repositorios y commit de la raíz. — Verificación: suites en verde sobre `main`.

## Trazabilidad

| Spec | Tareas |
|---|---|
| HU-1, RF-001/002, RF-022 | T010, T020–T022, T081–T082 |
| HU-2, RF-010 | T030–T031, T084 |
| HU-3, RF-014/015/017 | T040–T042 |
| HU-4, RF-016 | T050–T051, T083 |
| HU-5, RF-021 | T060–T061, T085 |
| RF-011/012/013, CE-001 | T011–T015, T070 |
| CE-002 | T015, T041–T042 |
| CE-003 | T084, T092 |
| RF-020, principio VIII | T071 |
| Principio III | T072–T073, T080 |
