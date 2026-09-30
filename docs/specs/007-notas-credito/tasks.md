# Tareas 007: Anulación y corrección de comprobantes

**Estado:** Borrador · **Fecha:** 2026-09-30 · **Plan:** [plan.md](./plan.md) (aprobado)

> Mismas convenciones que las specs anteriores: tareas pequeñas con su verificación; `[P]` = en paralelo con la anterior; 🔒 = confirmación del usuario al ejecutarla.
> **Pruebas primero:** las tareas «Escribir pruebas» terminan en rojo y la siguiente las pone en verde.
> Antes de cada commit se verifica con el **código de salida** de las suites de la API (`php artisan test`, `composer test:pgsql`, `vendor/bin/pint --test`) o del frontend.

## Fase 1 · Preparación y *spike*

- [ ] **T001** Crear la rama `007-notas-credito` desde `main` en la API y en el frontend. — Verificación: rama en ambos.
- [ ] **T002** *Spike* contra SUNAT beta (`tests/Beta/spike_007.php`): notas de crédito con motivos 01, 06 y 07 sobre una boleta y una factura aceptadas, con series `BC01` y `FC01`; devolución parcial con descuento prorrateado. — Verificación: resultado por caso anotado en `plan.md`; si algún caso falla, la regla corregida queda en el plan.

## Fase 2 · Fundamentos

- [ ] **T010** Enums: `DocumentType::CreditNote` (`07`, prefijo según el comprobante modificado), `CreditNoteReason` (`01`, `06`, `07`, con etiquetas), `CorrectionStatus` y el estado `discarded` en `SalesDocumentStatus`. Con pruebas. — Verificación: pruebas en verde.
- [ ] **T011** Migración: columnas nuevas de `sales_documents` y `sales_document_lines`; `series` admite `07`. Modelos actualizados (relaciones `reference`, `creditNotes`, `referenceLine`; campos mutables tras emitir). — Verificación: `test` y `test:pgsql` en verde; `php artisan migrate` en local.
- [ ] **T012** Series de nota de crédito en la 004: `StoreSeriesRequest` admite `07` con código `F…` o `B…`, y el Nuevo RUS no puede crear `F…`. Ampliar `SeriesTest`. — Verificación: pruebas en verde.

## Fase 3 · Cálculo y XML

- [ ] **T020** Escribir pruebas `Unit/Sunat/CreditNoteCalculatorTest`: devolución parcial de una línea con descuento, devoluciones sucesivas que suman exacto al céntimo, resto final, anulación del restante. — Verificación: en rojo.
- [ ] **T021** Implementar el cálculo de las líneas de la nota (prorrateo y resto) sobre `TaxCalculator`. — Verificación: T020 en verde.
- [ ] **T022** Escribir e implementar el caso de nota en `UblBuilderTest` y en `UblBuilder` (`Note` de Greenter: documento afectado, motivo, descripción, totales). — Verificación: en verde.

## Fase 4 · Emisión de notas (HU-1, HU-2)

- [ ] **T030** Bloquear la reversión manual de ventas de un comprobante en `InventoryService::reverse()` (hallazgo del plan) y añadir `InventoryService::restock()` hacia los lotes de la venta original. Pruebas incluidas. — Verificación: en verde.
- [ ] **T031** Escribir pruebas `Sales/IssueCreditNoteTest` (con `FakeSunatSender`):
  - HU-1: anulación con y sin reposición; no se anula dos veces; el original pasa a «anulado» solo cuando la nota es aceptada.
  - HU-2: devolución parcial y total; límite por línea; segunda devolución; producto inactivo; el stock vuelve a los lotes originales.
  - A-41: el vendedor también emite.
  - Serie de nota inexistente o con la letra equivocada → 422; idempotencia.
  - Casos límite: sobre pendiente, rechazado o descartado → 422; SUNAT sin respuesta → nota pendiente y original sin cambios.
  - Auditoría de la nota y de la reposición.
  - Verificación: en rojo.
- [ ] **T032** Implementar `CreditNoteService`, request, ruta y recurso ampliado (`correction_status`, `remaining`, `credit_notes`, `reference`). — Verificación: T031 en verde en ambas bases.
- [ ] **T033** Escribir e implementar en `SunatDispatcher` el efecto del estado final de una nota: aceptada → `correction_status` del original; rechazada → se revierte su reposición. Pruebas incluidas. — Verificación: en verde.
- [ ] **T034** [P] Escribir e implementar `Integration/Sales/ConcurrentReturnTest` (solo PostgreSQL): devoluciones simultáneas de la misma línea nunca superan lo emitido. Comprobar con una mutación (sin el bloqueo del original) que falla. — Verificación: en verde, y en rojo con la mutación.

## Fase 5 · Rechazados (HU-3)

- [ ] **T040** Escribir pruebas `Sales/DiscardRejectedTest`: solo rechazados, solo el administrador, motivo obligatorio, stock repuesto, número usado, no admite notas, auditoría. — Verificación: en rojo.
- [ ] **T041** Implementar `SalesDocumentService::discard()` y su ruta. — Verificación: T040 en verde.

## Fase 6 · Consistencia, seguridad y contrato

- [ ] **T050** `Integration/Sales/ReturnConsistencyTest` (CE-002): secuencias aleatorias de ventas, devoluciones, anulaciones, notas rechazadas y descartes → el stock y los saldos cuadran con lo emitido menos lo devuelto. — Verificación: en verde con varias semillas.
- [ ] **T051** [P] Rutas nuevas en `TenantRoutes.php` y casos de la empresa B en `CrossTenantAccessTest`; `NoSunatSecretsLeakTest` con una nota. — Verificación: en verde.
- [ ] **T052** [P] `Contract/SalesDocumentResourcesTest` ampliado (nota, notas del comprobante, restantes). — Verificación: en verde.
- [ ] **T053** PDF de la nota (título, comprobante modificado y motivo) con prueba; `php artisan openapi:generate`; series `FC01` y `BC01` en `DevelopmentSeeder`; README de la API. — Verificación: prueba en verde; OpenAPI con las rutas nuevas; `migrate:fresh --seed` en local (autorizado).

## Fase 7 · Frontend

- [ ] **T060** `pnpm api:types`; tipos y API de notas, descarte y campos nuevos del comprobante. — Verificación: `pnpm type-check`.
- [ ] **T061** Detalle del comprobante: «Registrar devolución» y «Anular» (diálogo con motivo, cantidades por línea o reposición), estado de corrección, notas y restantes por línea; detalle de una nota con su documento modificado. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T062** «Descartar» para rechazados (solo el administrador), filtro «Notas de crédito» y distintivos «Anulado» y «Devuelto» en el listado. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T063** [P] Series: crear series de nota de crédito. Pruebas incluidas. — Verificación: pruebas en verde.

## Fase 8 · Cierre

- [ ] **T070** Ampliar `tests/Beta/SunatBetaTest` con notas de anulación y de devolución parcial sobre boleta y factura, y correrlo contra SUNAT beta real. — Verificación: casos aceptados; salida en `verificacion.md`.
- [ ] **T071** API: `php artisan test`, `composer test:pgsql` y `vendor/bin/pint --test` en verde (códigos de salida). — Verificación: salidas adjuntas.
- [ ] **T072** Frontend: `pnpm type-check`, `pnpm lint`, `pnpm test:unit --run` y `pnpm build-only` en verde. — Verificación: salidas adjuntas.
- [ ] **T073** Verificación de extremo a extremo por el proxy de Vite:
  - Devolución parcial (cronometrada, CE-003) y total; anulación con y sin reposición; notas aceptadas por SUNAT beta real.
  - Stock y saldos después de cada paso.
  - Nota con SUNAT caída (pendiente y reintento) y descarte de un rechazado.
  - El vendedor emite una nota; otra empresa recibe 404; búsqueda de secretos.
  - Capturas a 360 px. Resultado en `verificacion.md`.
  - Verificación: `verificacion.md` con resultado por paso.
- [ ] **T074** Documentación: README de la API y del frontend, `AGENTS.md` y estado de la spec en `specs/README.md`. — Verificación: enlaces válidos.
- [ ] **T075** 🔒 Merge local de `007-notas-credito` a `main` en ambos repositorios y commit de la raíz. — Verificación: suites en verde sobre `main`.

## Trazabilidad

| Spec | Tareas |
|---|---|
| HU-1, RF-001–002 | T031–T033, T061 |
| HU-2, RF-004–005 | T020–T021, T031–T034, T061 |
| HU-3 | T040–T041, T062 |
| HU-4 | T032, T061–T062 |
| RF-003 | T012, T063 |
| RF-006 | T022, T033, T053 |
| RF-007 (A-40) | T030–T033 |
| RF-008 (A-41, A-42) | T031, T040 |
| RF-009 | T031, T040 |
| CE-001 | T002, T070 |
| CE-002 | T034, T050 |
| CE-003 | T073 |
| Principio VIII / IV | T051 |
| Principio III | T052–T053, T060 |
