# Tareas 012: Vender — carrito persistente y diseño en dos columnas

**Estado:** Implementada (2026-10-03) · **Fecha:** 2026-10-03 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.

## Fase 1 · Preparación

- [x] **T001** Rama `012-vender-mejorado` en el frontend — Verificación: `git branch --show-current`
- [x] **T002** `salesApi.listProducts` y `getProduct` — Verificación: `pnpm type-check`

## Fase 2 · Venta en curso (HU-1, HU-2)

- [x] **T003** Pruebas de `sale-draft` (deben fallar) — Verificación: en rojo
- [x] **T004** Store `sale-draft` con persistencia, `refresh` y `clear` — Verificación: T003 en verde
- [x] **T005** Cerrar sesión en `AppLayout` borra la venta en curso (prueba primero) — Verificación: `AppLayout.spec.ts` en verde

## Fase 3 · Pantalla (HU-2, HU-3, HU-4)

- [x] **T006** Pruebas de `NewSaleView` (catálogo, persistencia, vaciar, éxito, idempotencia) adaptadas y nuevas (deben fallar) — Verificación: en rojo
- [x] **T007** `ProductCatalog` y `SaleCart` — Verificación: pruebas en verde
- [x] **T008** `NewSaleView` en dos columnas y barra con hoja en el celular — Verificación: T006 en verde

## Fase 4 · Pantalla completa (v1.1)

- [x] **T012** `meta.fullWidth` en `new-sale`; catálogo y carrito con desplazamiento propio a toda la altura — Verificación: capturas en escritorio (1280 y 1920) y celular; en escritorio la página no se desplaza (alto del documento = alto de la ventana). Catálogo con columnas automáticas (`auto-fill`, mínimo 9.5 rem) y búsqueda fija arriba

## Fase 5 · Tipo en el carrito (v1.2)

- [x] **T013** Prueba de la barra del celular con el tipo (debe fallar); selector, distintivo y motivo dentro de `SaleCart` — Verificación: pruebas de `NewSaleView` en verde y capturas (escritorio claro y oscuro con boleta, barra y hoja del celular)

## Fase 6 · Orden del cobro (v1.3)

- [x] **T014** Prueba del orden dentro del carrito (debe fallar) y reorden de `SaleCart` — Verificación: pruebas en verde y capturas (factura: comprobante, cliente con RUC, medio de pago y «Emitir factura» deshabilitado hasta elegir cliente)
- [x] **T015** Más productos de demostración en `DevelopmentSeeder` (API, solo `local`) para ver la paginación del catálogo — Verificación: pruebas del seeder y «Cargar más» en pantalla. 41 productos de bodega (rama `012-vender-mejorado` de la API); ejecutado con `db:seed --class=DevelopmentSeeder` (idempotente, sin `migrate:fresh`): «Bodega Demo» con 46 activos, 20 → 40 → 46 con «Cargar más»

## Fase 7 · Confirmación sin salir (v1.4)

- [x] **T016** Pruebas del modal (ticket y comprobante, pendiente o rechazado, nueva venta con foco, imprimir y ver detalle) (deben fallar) — Verificación: en rojo
- [x] **T017** `document-result.ts`, `data-autofocus` en `BaseDialog`, `SaleDoneDialog` y su uso en `NewSaleView` — Verificación: T016 en verde. Además: el catálogo se recarga tras la venta (el disponible descuenta lo vendido) y el ticket a imprimir se retira en `afterprint` (en móviles `print()` no siempre bloquea)
- [x] **T018** Venta real con ticket y con boleta contra SUNAT beta; capturas en claro, oscuro y 360 px — Verificación: revisión manual. T-000008 y T-000009 (celular) y B001-00000008 aceptada por SUNAT beta, sin salir de `/vender`; en el celular la hoja del carrito se cierra y el foco queda en «Nueva venta»

## Cierre

- [x] **T009** Capturas a 360 px (barra y hoja) y en escritorio, en claro y oscuro; venta de 3 productos solo con toques (CE-002) — Verificación: revisión manual. CE-002 contra la API local: tres toques y «Cobrar» crearon el ticket T-000006 y borraron la venta guardada. Corrigió: el total y «Cobrar» quedaban fuera de vista con carritos largos (pie `sticky` en columna y hoja; `BaseDialog` con alto máximo y desplazamiento) y la columna del carrito pasó a 24 rem
- [x] **T010** Comprobaciones del principio VI en verde (frontend 214 pruebas; API 639 SQLite y PostgreSQL, Pint)
- [x] **T011** Bitácora de `AGENTS.md` y estado en `docs/specs/README.md`
