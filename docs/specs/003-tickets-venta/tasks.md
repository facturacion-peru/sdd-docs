# Tareas 003: Tickets de venta internos

**Estado:** Borrador (pendiente de aprobación) · **Fecha:** 2026-09-28 · **Plan:** [plan.md](./plan.md) (aprobado)

> Mismas convenciones que las specs anteriores: tareas pequeñas con su verificación; `[P]` = en paralelo con la anterior; 🔒 = confirmación del usuario al ejecutarla.
> **Pruebas primero:** las tareas «Escribir pruebas» terminan en rojo y la siguiente las pone en verde.
> Antes de cada commit se verifica con el **código de salida** de `php artisan test`, `composer test:pgsql` y `vendor/bin/pint --test` (API) o de las comprobaciones del frontend.

## Fase 1 · Preparación

- [ ] **T001** Crear la rama `003-tickets-venta` desde `main` en la API y en el frontend. — Verificación: `git branch --show-current` en ambos.

## Fase 2 · Fundamentos

- [ ] **T010** Escribir pruebas unitarias de `Decimal::mul` y `Decimal::round` (mitad hacia arriba, también negativos y escala 2). — Verificación: en rojo.
- [ ] **T011** Implementar `Decimal::mul` y `Decimal::round`. — Verificación: T010 en verde.
- [ ] **T012** Enum `PaymentMethod` (`cash`, `card`, `yape_plin`, `transfer`, con etiquetas) y `TicketStatus`, con pruebas. — Verificación: pruebas en verde.
- [ ] **T013** Migraciones `ticket_sequences`, `tickets` y `ticket_lines`, según el plan. — Verificación: `test` y `test:pgsql` en verde; `php artisan migrate` en local.
- [ ] **T014** Modelos `Ticket` (con `display_number`), `TicketLine` y `TicketSequence` con `BelongsToCompany`, y *factories*. — Verificación: `pint --test`; creación por *factory* en una prueba.

## Fase 3 · HU-1 Registrar una venta con ticket (P1)

- [ ] **T020** Escribir pruebas `Sales/IssueTicketTest`:
  - HU-1.1: ticket con número propio, líneas, total, medio de pago, vendedor, fecha y hora; stock descontado.
  - HU-1.2: sin cliente, «Cliente varios» (`customer_name` nulo).
  - HU-1.4: un servicio no toca el stock.
  - HU-1.5: descuento mayor que el importe, rechazado.
  - RF-011: el precio viene del catálogo; un `unit_price` enviado se ignora.
  - RF-009: el ticket conserva código, nombre, precio y afectación aunque el producto cambie después.
  - Productos inactivos o inexistentes, cantidades con decimales según la unidad, medio de pago inválido: rechazados.
  - Importes con redondeo (casos límite) y auditoría `ticket.issued`.
  - El administrador y el vendedor pueden vender.
  - Verificación: en rojo.
- [ ] **T021** Implementar `TicketService::issue`, `POST /tickets`, `StoreTicketRequest` y `TicketResource` (con `display_number` y `legal_notice`). — Verificación: T020 en verde en SQLite y PostgreSQL.
- [ ] **T022** Escribir y poner en verde `Sales/TicketStockTest` (HU-1.3, RF-003): si una línea no tiene stock, no queda ticket, ni líneas, ni movimientos, ni número consumido; la respuesta 422 indica el producto y el disponible. — Verificación: en verde en ambas bases.
- [ ] **T023** Escribir y poner en verde `Sales/TicketIdempotencyTest` (RF-008): la misma clave devuelve el mismo ticket (200) sin descontar stock otra vez; una clave de otra empresa no colisiona. — Verificación: en verde.
- [ ] **T024** Escribir `Integration/Sales/ConcurrentTicketNumberingTest` (solo PostgreSQL, `pcntl_fork`): N procesos emiten a la vez y los números quedan 1..N sin huecos ni duplicados (CE-003). Verificar con una mutación temporal (sin bloqueo de la secuencia) que la prueba falla. — Verificación: en verde en `test:pgsql` y omitida en `test`.

## Fase 4 · HU-2 Contenido del ticket (P1)

- [ ] **T030** Escribir y poner en verde `Sales/TicketContentTest`: `GET /tickets/{ticket}` con líneas, `display_number` `T-000001` y `legal_notice`; la respuesta nunca contiene «boleta» ni «factura»; un ticket anulado muestra su estado. — Verificación: en verde.

## Fase 5 · HU-3 Anular un ticket (P2)

- [ ] **T040** Escribir pruebas `Sales/VoidTicketTest`:
  - HU-3.1: anular con motivo; el número se conserva y el stock vuelve con reversiones enlazadas.
  - HU-3.2: el vendedor no anula.
  - HU-3.3: no se anula dos veces.
  - Un ticket de servicios se anula sin movimientos.
  - La reversión manual de un movimiento de venta de un ticket se rechaza (cambio en la 002).
  - Auditoría `ticket.voided`.
  - Verificación: en rojo.
- [ ] **T041** Implementar `TicketService::void`, `POST /tickets/{ticket}/void` y el bloqueo en `InventoryService::reverse()`. — Verificación: T040 y la suite de la 002 en verde, en ambas bases.

## Fase 6 · HU-4 Consultar ventas (P2)

- [ ] **T050** Escribir pruebas `Sales/ListTicketsTest`:
  - HU-4.1: filtro por fechas con totales por medio de pago sin anulados.
  - Filtros por estado, medio de pago y vendedor (este último solo para el administrador).
  - HU-4.2: el vendedor solo ve sus tickets en el listado y en el detalle (404 para los ajenos).
  - Paginación.
  - Verificación: en rojo.
- [ ] **T051** Implementar `GET /tickets` con `IndexTicketRequest` y el alcance del vendedor en el detalle. — Verificación: T050 en verde.

## Fase 7 · Aislamiento y contrato

- [ ] **T060** Añadir las 4 rutas a `TenantRoutes.php` y un ticket de B en `CrossTenantAccessTest`. — Verificación: cobertura y acceso cruzado en verde.
- [ ] **T061** [P] `Contract/TicketResourceTest`. — Verificación: en verde.
- [ ] **T062** `php artisan openapi:generate` y tickets demo en `DevelopmentSeeder` (emitidos y uno anulado). — Verificación: el OpenAPI incluye las 4 rutas; `migrate:fresh --seed` en local (autorizado) carga las ventas demo.

## Fase 8 · Frontend

- [ ] **T070** `pnpm api:types`; `features/sales/types.ts` y `api.ts`. — Verificación: `pnpm type-check`.
- [ ] **T071** Navegación: principal (Inicio, Vender, Ventas, Productos) y menú «Más» del administrador (Alertas, Usuarios, Auditoría, Mi empresa), accesible con teclado. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T072** Venta rápida `/vender`: búsqueda de productos activos con disponible, agregar con un toque, cantidades con +/−, descuento por línea, medio de pago, cliente opcional plegado, vista previa del total con la regla de redondeo del servidor, clave de idempotencia reutilizada en reintentos, y línea marcada si la API responde stock insuficiente. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T073** Ticket `/ventas/:id`: vista de 80 mm con la leyenda siempre visible, estilos de impresión (`@page`), «Imprimir» y «Anular» (solo el administrador, con motivo). Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T074** [P] Listado `/ventas`: hoy por defecto, filtros y totales por medio de pago. Pruebas incluidas. — Verificación: pruebas en verde.

## Cierre

- [ ] **T080** API: `php artisan test`, `composer test:pgsql` y `vendor/bin/pint --test` en verde (códigos de salida). — Verificación: salidas adjuntas.
- [ ] **T081** Frontend: `pnpm type-check`, `pnpm lint`, `pnpm test:unit --run` y `pnpm build-only` en verde. — Verificación: salidas adjuntas.
- [ ] **T082** Verificación de extremo a extremo por el proxy de Vite (vender, idempotencia, stock insuficiente, anular, listado, vendedor solo lo suyo, otra empresa) y capturas a 360 px, incluida la vista de impresión. Se cronometra una venta de 3 productos (CE-001). Resultado en `verificacion.md`. — Verificación: `verificacion.md` con resultado por paso.
- [ ] **T083** Documentación: README de la API y del frontend, `AGENTS.md` y estado de la spec en `specs/README.md`. — Verificación: enlaces válidos.
- [ ] **T084** 🔒 Merge local de `003-tickets-venta` a `main` en ambos repositorios y commit de la raíz. — Verificación: suites en verde sobre `main`.

## Trazabilidad

| Spec | Tareas |
|---|---|
| HU-1, RF-001, RF-007, RF-009 a RF-013 | T010–T014, T020–T021, T072 |
| HU-1.3, RF-003 | T022, T072 |
| RF-008 | T023, T072 |
| RF-002, CE-003 | T013, T024 |
| HU-2, RF-004, RF-005, CE-002 | T030, T073 |
| HU-3, RF-006 | T040–T041, T073 |
| HU-4, RF-012 | T050–T051, T074 |
| CE-001 | T072, T082 |
| Principio VIII | T060 |
| Principio III | T061–T062, T070 |
