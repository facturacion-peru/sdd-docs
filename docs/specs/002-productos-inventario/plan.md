# Plan 002: Productos, lotes y movimientos de inventario

**Estado:** Aprobado (2026-09-28) · **Fecha:** 2026-09-28 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

Sobre la base de la 001 (aislamiento con `BelongsToCompany`, auditoría y contratos):

1. Tres tablas nuevas: `products`, `product_lots` e `inventory_movements`. Los movimientos son **inmutables**, como la auditoría.
2. Toda escritura de stock pasa por un único servicio, `InventoryService`. Antes de tocar lotes bloquea la fila del producto (`SELECT … FOR UPDATE`), lo que evita carreras y *deadlocks*, y asigna lotes FEFO o FIFO.
3. Las cantidades se calculan con `bcmath` (decimales exactos, sin `float`). El saldo de cada lote se guarda como caché derivada que solo escribe el servicio y que siempre coincide con la suma de sus movimientos (RF-013, verificado por prueba).
4. En el frontend, una feature nueva `inventory`.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos | API y frontend |
| Dependencias nuevas | Ninguna en Composer ni en pnpm. Se declara `ext-bcmath` en `composer.json` como requisito de plataforma (ya está instalada en local; hay que tenerla en el servidor) |
| Base de datos | Tres tablas nuevas y una columna en `companies` (`expiry_warning_days`) |
| Pruebas | Suite en SQLite y `composer test:pgsql`. La prueba de concurrencia corre solo en PostgreSQL, con `pcntl_fork` |

## Chequeo de constitución (v1.1.2)

| Principio | Cumple | Cómo / justificación |
|---|---|---|
| I. Spec primero | ✅ | Spec 1.0 aprobada, con aclaraciones A-12 a A-16 y A-26 a A-28 resueltas. |
| II. Corrección fiscal | ✅ | No se calculan impuestos. La afectación al IGV se guarda con su código SUNAT (catálogo 07) para la 005; los precios usan decimales exactos. |
| III. Contrato de API | ⚠️ | OpenAPI regenerado y `pnpm api:types`. Sigue la excepción heredada: las respuestas se tipan a mano y las protegen pruebas de contrato, ahora en dos variantes (con y sin costo). |
| IV. Seguridad | ✅ | Validación en `FormRequest`; `authorize()` según rol; el costo no sale en respuestas al vendedor (RF-022). |
| V. Simplicidad | ✅ | Sin almacenes, sin conversiones de unidades y sin paquetes. Un servicio único para el stock. |
| VI. Verificable | ✅ | Pruebas por escenario, prueba de consistencia de saldos (CE-001) y prueba de concurrencia real en PostgreSQL (CE-002). |
| VII. Personas reales | ✅ | Registrar una entrada solo exige la cantidad; lo demás tiene valores por defecto (CE-003). Pantallas *mobile-first*. |
| VIII. Aislamiento y auditoría | ✅ | Los tres modelos usan `BelongsToCompany`; cada ruta nueva entra en `TenantRoutes.php`; se auditan altas, cambios, entradas, ajustes y reversiones. |

## Modelo de datos

| Tabla | Columnas relevantes | Restricciones |
|---|---|---|
| `products` | `company_id`, `code` (32), `name`, `type` (`good`/`service`), `unit` (código SUNAT), `sale_price` decimal(12,2), `igv_affectation` (`10`/`20`/`30`), `min_stock` decimal(14,3) nulo, `tracks_expiry` bool, `active` bool | único (`company_id`, `code`); índice (`company_id`, `active`, `name`) |
| `product_lots` | `company_id`, `product_id`, `lot_number` (40), `received_at` fecha, `expires_at` fecha nula, `initial_quantity` y `remaining_quantity` decimal(14,3), `unit_cost` decimal(14,4) nulo, `reference` nulo, `created_by` | único (`product_id`, `lot_number`); índice (`product_id`, `expires_at`, `received_at`); **CHECK `remaining_quantity >= 0`** en PostgreSQL |
| `inventory_movements` | `company_id`, `product_id`, `lot_id`, `type` (`entry`/`sale`/`adjustment`/`return`/`reversal`), `quantity` decimal(14,3) con signo, `lot_balance_after` y `product_balance_after`, `reason` nulo, `note` nulo, `source_type`/`source_id` (documento de origen, polimórfico), `reverses_id` nulo, `created_by`, `created_at` | sin `updated_at`; `reverses_id` único (un movimiento se revierte una sola vez); el modelo impide `update` y `delete` |
| `companies` (+) | `expiry_warning_days` smallint, por defecto 30 | entre 1 y 365 |

**Stock:**
- *Stock físico* de un producto = suma de `remaining_quantity` de sus lotes.
- *Disponible para vender* = lo mismo, excluyendo los lotes vencidos.

**Catálogos como *enums*:**
- `UnitOfMeasure`: subconjunto del catálogo 03 de SUNAT (`NIU` unidad, `ZZ` servicio, `KGM`, `GRM`, `LTR`, `MLT`, `MTR`, `BX` caja, `PK` paquete, `DZN` docena), con `allowsDecimals()` para `KGM`, `GRM`, `LTR`, `MLT` y `MTR`.
- `IgvAffectation`: `10` gravado, `20` exonerado y `30` inafecto (catálogo 07, operaciones onerosas).
- `ProductType`, `MovementType` y `AdjustmentReason` (conteo, merma, daño, vencimiento, error, otro).

**Número de lote automático** (A-13): se genera como `L-{AAAAMMDD}-{id}` al crear el lote. Es único sin necesidad de contadores.

## `InventoryService` (único punto de escritura del stock)

| Método | Qué hace | Spec |
|---|---|---|
| `registerEntry(Product, datos, User)` | Crea el lote y un movimiento `entry`, y audita. Rechaza servicios, cantidades ≤ 0 y vencimientos faltantes o pasados si el producto controla vencimiento. | HU-2 |
| `consume(Product, cantidad, origen, User)` | Asigna lotes FEFO o FIFO sin usar lotes vencidos, crea un movimiento `sale` por lote y devuelve los movimientos. Si no alcanza, lanza `InsufficientStock` con el disponible. **Exige una transacción abierta por quien llama** (lanza `LogicException` si no la hay), para que la venta y el descuento se confirmen juntos (RF-017). Es para las specs 003 y 005: en la 002 no tiene endpoint. | HU-3 |
| `adjust(Lot, cantidad con signo, motivo, nota, User)` | Movimiento `adjustment`; rechaza dejar el lote en negativo. | HU-4.1, 4.3 |
| `reverse(Movement, motivo, User)` | Movimiento inverso enlazado por `reverses_id`. No se revierte dos veces, ni una entrada cuyo lote ya se consumió (el saldo no alcanza). | HU-4.2 |

**Concurrencia (CE-002):** cada operación bloquea primero la fila del **producto** (`lockForUpdate`) y después lee los lotes. Toda escritura de stock del mismo producto queda así serializada, siempre en el mismo orden, sin riesgo de *deadlock*. El CHECK de PostgreSQL es la última barrera si algún día alguien escribe fuera del servicio.

**Decimales:** `bcmath` con escala 3 para cantidades y 4 para costos. Los modelos usan casts `decimal:N`, que devuelven cadenas; nunca se hace aritmética con `float`.

## Contrato de API (`/api/v1`, grupo de empresa)

| Método | Ruta | Rol | Descripción | Spec |
|---|---|---|---|---|
| GET | `/catalogs/inventory` | Todos | Unidades (con `allows_decimals`), afectaciones y motivos de ajuste | RF-001 |
| GET | `/products` | Todos | Paginado; búsqueda por código o nombre; filtros de tipo y estado; stock y disponible | HU-1 |
| POST | `/products` | Admin | Alta | HU-1.1/1.2 |
| GET | `/products/{product}` | Todos | Detalle con stock | HU-1 |
| PATCH | `/products/{product}` | Admin | Edición, activar o desactivar; no pasa de bien a servicio con stock | HU-1.3, casos límite |
| GET | `/products/{product}/lots` | Todos | Lotes con saldo y vencimiento | HU-2 |
| GET | `/products/{product}/movements` | Admin | Historial paginado | HU-5.1 |
| POST | `/products/{product}/entries` | Admin | Entrada por lote | HU-2 |
| POST | `/lots/{lot}/adjustments` | Admin | Ajuste con motivo | HU-4.1 |
| POST | `/movements/{movement}/reverse` | Admin | Reversión con motivo | HU-4.2 |
| GET | `/inventory/alerts` | Admin | Stock bajo y lotes próximos a vencer | HU-5.2/5.3 |
| PATCH | `/company` (+) | Admin | Nuevo campo `expiry_warning_days` | RF-021 |

- **Costo (RF-022):** `ProductResource` y `LotResource` incluyen `unit_cost` y el costo de la última entrada solo si quien consulta es `company_admin`.
- **Stock insuficiente:** 422 con `errors.quantity` y `meta.available`, para que el frontend muestre cuánto hay.

## Frontend: feature `inventory`

| Ruta | Rol | Pantalla |
|---|---|---|
| `/productos` | Todos | Catálogo con búsqueda, filtros y tarjetas con stock; los desactivados, ocultos por defecto |
| `/productos/nuevo`, `/productos/:id/editar` | Admin | Formulario de producto (la unidad define si se admiten decimales) |
| `/productos/:id` | Todos | Detalle: stock, disponible, lotes; para el administrador además historial, ajustar y revertir |
| `/productos/:id/entrada` | Admin | Entrada rápida: solo cantidad obligatoria; fecha de hoy por defecto; lote, costo y referencia opcionales; vencimiento si aplica |
| `/inventario/alertas` | Admin | Stock bajo y próximos a vencer |

- **Navegación:** «Productos» (todos) y «Alertas» (admin). En `features/company`, un campo nuevo para los días de aviso de vencimiento.
- **Componentes:** `ProductForm`, `StockEntryForm`, `AdjustStockDialog`, `ReverseMovementDialog`, `LotList`, `MovementList`.
- **Cantidades:** `inputmode="decimal"` solo si la unidad lo permite. Los importes se muestran con `Intl.NumberFormat('es-PE', { style: 'currency', currency: 'PEN' })`.

## Estrategia de pruebas

**API** (en SQLite y en PostgreSQL):

| Escenario / requisito | Prueba |
|---|---|
| HU-1.1–1.4, RF-001/002, RF-022 | `Inventory/ProductCatalogTest` (incluye vendedor sin costo) |
| HU-2.1–2.5, RF-010 | `Inventory/StockEntryTest` |
| HU-3.1–3.5, RF-014/015/017 | `Inventory/ConsumeStockTest`: FEFO, FIFO, varios lotes, insuficiente, vencidos, falta de transacción |
| HU-4.1–4.4, RF-016 | `Inventory/AdjustAndReverseTest` |
| HU-5.1–5.3, RF-021 | `Inventory/MovementsAndAlertsTest` |
| Casos límite (bien → servicio con stock, decimales según unidad, activar vencimiento con lotes sin fecha) | `Inventory/ProductEdgeCasesTest` |
| CE-001 | `Inventory/StockConsistencyTest`: secuencia aleatoria reproducible (semilla fija) de entradas, ventas, ajustes y reversiones; al final, para todo lote, saldo = suma de movimientos, y stock del producto = suma de lotes |
| CE-002 | `Inventory/ConcurrentConsumptionTest` (solo PostgreSQL, `pcntl_fork`): varios procesos compiten por las últimas unidades; ninguno deja saldo negativo y el total vendido = stock inicial. Se omite en SQLite |
| Inmutabilidad de movimientos | `Inventory/MovementImmutabilityTest` |
| Aislamiento | Las 11 rutas nuevas en `TenantRoutes.php` (resource, list o reference); `CrossTenantAccessTest` las recorre solo |
| Contratos | `Contract/ProductResourceTest` y `Contract/LotResourceTest` (variantes admin y vendedor), `Contract/MovementResourceTest` |

**Frontend (Vitest):** catálogo (búsqueda y vendedor sin costo), formulario de producto (decimales según unidad, errores 422), entrada rápida (valores por defecto y vencimiento obligatorio si aplica), diálogos de ajuste y reversión, alertas.

**Verificación manual:** a 360 px, cronometrar una entrada de stock (CE-003, menos de 30 s).

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Errores de redondeo con decimales | `bcmath` en todo el cálculo; casts `decimal`; prueba de consistencia |
| Carreras al vender la última unidad | Bloqueo de la fila del producto + CHECK en PostgreSQL + prueba de concurrencia real |
| Diferencias SQLite/PostgreSQL (CHECK, `FOR UPDATE`) | `composer test:pgsql` obligatorio en cada tarea que toque stock |
| La 003 o la 005 descuentan stock fuera de una transacción | `consume()` lo rechaza con una excepción |
| Crecimiento de `inventory_movements` | Índices por producto y fecha; paginación del historial |
| `ext-bcmath` ausente en el servidor | Declarada en `composer.json`: `composer install` falla si falta |

## Aprobación

Aprobado por el usuario el 2026-09-28, incluidos el requisito de plataforma `ext-bcmath` y el campo `expiry_warning_days` editable desde «Mi empresa».
