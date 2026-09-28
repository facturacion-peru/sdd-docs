# Tareas NNN: <Nombre de la capacidad>

**Estado:** Borrador · **Fecha:** AAAA-MM-DD · **Plan:** [plan.md](./plan.md)

> Solo se redacta cuando el plan está **Aprobado**. Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior.

## Fase 1 · Preparación

- [ ] **T001** <acción concreta> — Verificación: <comando o comprobación>

## Fase 2 · <Historia HU-1>

- [ ] **T010** Escribir pruebas de <…> (deben fallar) — Verificación: `php artisan test --filter=…` en rojo
- [ ] **T011** Implementar <…> — Verificación: las pruebas de T010 en verde

## Cierre

- [ ] Comprobaciones del principio VI en verde
- [ ] OpenAPI regenerado y `pnpm api:types` ejecutado (si cambió el contrato)
- [ ] Entrada en la bitácora de `AGENTS.md`
