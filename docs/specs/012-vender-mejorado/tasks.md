# Tareas 012: Vender — carrito persistente y diseño en dos columnas

**Estado:** Lista (2026-10-03) · **Fecha:** 2026-10-03 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.

## Fase 1 · Preparación

- [ ] **T001** Rama `012-vender-mejorado` en el frontend — Verificación: `git branch --show-current`
- [ ] **T002** `salesApi.listProducts` y `getProduct` — Verificación: `pnpm type-check`

## Fase 2 · Venta en curso (HU-1, HU-2)

- [ ] **T003** Pruebas de `sale-draft` (deben fallar) — Verificación: en rojo
- [ ] **T004** Store `sale-draft` con persistencia, `refresh` y `clear` — Verificación: T003 en verde
- [ ] **T005** Cerrar sesión en `AppLayout` borra la venta en curso (prueba primero) — Verificación: `AppLayout.spec.ts` en verde

## Fase 3 · Pantalla (HU-2, HU-3, HU-4)

- [ ] **T006** Pruebas de `NewSaleView` (catálogo, persistencia, vaciar, éxito, idempotencia) adaptadas y nuevas (deben fallar) — Verificación: en rojo
- [ ] **T007** `ProductCatalog` y `SaleCart` — Verificación: pruebas en verde
- [ ] **T008** `NewSaleView` en dos columnas y barra con hoja en el celular — Verificación: T006 en verde

## Cierre

- [ ] **T009** Capturas a 360 px (barra y hoja) y en escritorio, en claro y oscuro; venta de 3 productos solo con toques (CE-002) — Verificación: revisión manual
- [ ] **T010** Comprobaciones del principio VI en verde
- [ ] **T011** Bitácora de `AGENTS.md` y estado en `docs/specs/README.md`
