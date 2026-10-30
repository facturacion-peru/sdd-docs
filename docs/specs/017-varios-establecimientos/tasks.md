# Tareas 017: Varios establecimientos

**Estado:** Aprobadas (2026-10-29) · **Fecha:** 2026-10-29 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior. 🔒 = requiere confirmación explícita del usuario.
> Reparto: las tareas **T** (API y cierre) las lleva la sesión de backend; las **F** (frontend), la sesión del frontend.

## Fase 1 · Preparación y datos (API)

- [x] **T001** Rama `017-varios-establecimientos` desde `main` en la API — Verificación: `git branch --show-current`
- [x] **T002** Prueba de la migración en `tests/Integration` (debe fallar): una base con productos, lotes, entradas, ventas con ticket y con boleta, notas con devolución, ajustes y vendedores, sembrada **antes** de migrar. Tras migrar:
  - todo queda en el principal de su empresa (dos empresas, para no mezclar);
  - los vendedores quedan asignados al principal;
  - el inicio, el stock, las listas y los saldos de movimientos son idénticos a los de antes (CE-003).

  Verificación: en rojo
- [x] **T003** Migración:
  - `establishments.active`;
  - `establishment_user` e `invitations.establishment_ids`;
  - `stock_transfers`;
  - `establishment_id` en `product_lots`, `inventory_movements`, `tickets` y `sales_documents`, relleno y luego obligatorio;
  - único de lotes `(product_id, establishment_id, lot_number)`.

  Modelos, relaciones y factories, con el principal como valor por defecto en las factories para que las pruebas existentes no cambien. Verificación: T002 en verde, migra y revierte en SQLite y PostgreSQL, y las suites siguen en verde

## Fase 2 · API: locales y acceso (HU-1, A-87)

- [x] **T004** Pruebas de `EstablishmentAccess` (deben fallar):
  - `allowed`: el administrador ve todos los activos; el vendedor, sus asignados activos;
  - `resolve`: sin id con un solo permitido usa ese; con varios, 422 «Elige el local»; un id no permitido, inactivo o de otra empresa, el mismo 422;
  - `filter`: «Todos» son los permitidos, incluidos los inactivos.

  Verificación: en rojo
- [x] **T005** `App\Establishments\EstablishmentAccess` y `scopeVisibleTo` en `Ticket`, `SalesDocument`, `ProductLot` e `InventoryMovement` — Verificación: T004 en verde
- [x] **T006** Pruebas de las rutas de locales y entradas en `TenantRoutes.php` (deben fallar):
  - `GET /establishments` (los permitidos; `include_inactive` solo para el administrador);
  - `POST` y `PATCH`: 4 dígitos, código repetido, código con comprobantes (`code_locked`), el principal no se desactiva, reactivar;
  - `PUT /users/{user}/establishments`: al menos uno activo, solo vendedores;
  - 403 para el vendedor y 404 para otra empresa;
  - auditoría `establishment.*` y `user.establishments_changed`.

  Verificación: en rojo
- [x] **T007** `EstablishmentController`, sus `FormRequest`, `EstablishmentService`, `EstablishmentResource` (con `code_locked`) y la asignación en `UserController`/`UserService` (`establishment_ids` en `UserResource`) — Verificación: T006 en verde
- [x] **T008** [P] Pruebas y cambios de sesión e invitaciones: `establishments` y `company.active_establishments_count` en `login`, aceptar invitación, `/auth/me` y `CompanyResource`; `POST /invitations` con `establishment_ids` (obligatorio para vendedor con varios locales activos; con uno, se asigna ese) y `InvitationResource` — Verificación: en verde

## Fase 3 · API: stock por local y traslados (HU-3, A-86)

- [x] **T009** Pruebas de `InventoryService` por local (deben fallar):
  - `registerEntry` crea el lote en el local;
  - `consume` toma FEFO solo del local y `InsufficientStock` informa el disponible del local;
  - `balance` por local y total; `product_balance_after` es el saldo del local;
  - `restock` repone en los lotes del local del original.

  Verificación: en rojo
- [x] **T010** `InventoryService` con el local en cada firma, `Product::scopeWithStock(?array)` y los llamados existentes (tickets, comprobantes, importación) pasando el principal por ahora — Verificación: T009 y las suites en verde
- [x] **T011** Pruebas del traslado (deben fallar):
  - FEFO de varios lotes, mismo número, fechas y costo en el destino (crea o suma al lote);
  - sin stock o con destino inactivo, 422 sin cambios; origen inactivo se permite;
  - `reverse` rechaza los movimientos de traslado;
  - `POST /products/{product}/transfers` responde 201 con `stock_by_establishment`, 403 para el vendedor y auditoría `inventory.transfer`.

  Verificación: en rojo
- [x] **T012** `InventoryService::transfer`, `StockTransfer`, `MovementType` `transfer_out`/`transfer_in`, `StoreTransferRequest` y la ruta — Verificación: T011 en verde
- [x] **T013** CE-004 en `tests/Integration` (PostgreSQL, con forks): traslado y venta simultáneos en el origen, y traslados cruzados A→B y B→A al mismo tiempo; stock final correcto y sin deadlock. Mutación: sin el bloqueo del producto, la prueba falla — Verificación: en verde en PostgreSQL y en rojo con la mutación
- [x] **T014** [P] Pruebas y cambios en lecturas de inventario: `establishment_id` en productos, catálogo, lotes, movimientos y alertas; `establishment` en cada fila; `stock_by_establishment` en `GET /products/{id}`; alertas por local (un producto puede salir dos veces); entradas con `establishment_id` — Verificación: en verde

## Fase 4 · API: venta por local (HU-2)

- [ ] **T015** Pruebas de tickets (deben fallar): queda en el local resuelto y consume allí; sin `establishment_id` y un solo permitido, como hoy; con varios, 422; no permitido o inactivo, 422; el correlativo sigue por empresa (A-88); `establishment` en la lista y `{id, code, name, address}` en el detalle — Verificación: en rojo
- [ ] **T016** `StoreTicketRequest` y `TicketService` con el local — Verificación: T015 en verde
- [ ] **T017** Pruebas de comprobantes y notas (deben fallar):
  - el local es el de la serie; `establishment_id` distinto, 422; serie de un local inactivo o no permitido, 422;
  - la serie automática se elige entre las del local;
  - el XML lleva el `CodLocal` y la dirección del local (`UblBuilder`);
  - la nota usa una serie de notas del local del original y repone allí; sin serie en ese local, el mensaje lo nombra;
  - descartar un rechazado repone en su local.

  Verificación: en rojo
- [ ] **T018** `SalesDocumentService`, `CreditNoteService` (`noteSeries` por local) y `SeriesController`/`SeriesService` (`establishment_id` al crear, fijo con comprobantes, `establishment_id` en `SeriesResource`) — Verificación: T017 en verde
- [ ] **T019** Ticket y comprobante impresos: nombre y dirección del local en `pdf/ticket.blade.php` y `TicketPrint`, y nombre del local (si no es el principal) en `pdf/sales-document.blade.php` y `DocumentPrint` — Verificación: `PrintParityTest` y las pruebas de PDF en verde
- [ ] **T020** Importación de productos con el local cerrado: el stock inicial va al `establishment_id` de la vista previa (por defecto el principal) — Verificación: pruebas de importación en verde

## Fase 5 · API: cifras y aislamiento (HU-4, CE-002)

- [ ] **T021** Pruebas y filtros por local (`establishment_id` o los permitidos) en `GET /dashboard`, `GET /tickets` y `GET /sales-documents`; 404 en el detalle de un ticket o comprobante de un local no permitido (también PDF, XML, CDR e impresión) — Verificación: en verde
- [ ] **T022** [P] Exportaciones con la columna `local` y el filtro; la hoja de productos suma los locales pedidos y la de lotes trae una fila por lote y local — Verificación: pruebas de exportación en verde
- [ ] **T023** `CrossEstablishmentAccessTest`, generado desde el router como `CrossTenantAccessTest`: un vendedor del local A no ve en ninguna respuesta marcadores de tickets, comprobantes, lotes ni movimientos del local B, y el detalle de B da 404. Mutación: sin `scopeVisibleTo` en tickets, falla — Verificación: en verde y en rojo con la mutación
- [ ] **T024** [P] `establishments_count` en `PlatformCompanyDetailResource`; `PlatformPrivacyTest` en verde — Verificación: en verde
- [ ] **T025** Contrato en `tests/Feature/Contract` (`EstablishmentResource`, `establishment` en filas y detalle, sesión, `stock_by_establishment`, `SeriesResource`), `php artisan openapi:generate`, suites SQLite y PostgreSQL y Pint — Verificación: códigos de salida 0 y cifras anotadas

## Fase 6 · Frontend

> Las lleva la sesión del frontend, según la sección «Frontend» del plan.
> - **Dependencias de la API** (por el `pnpm api:types` de los parámetros nuevos):
>   - F002 y F013–F015 necesitan T007–T008;
>   - F016–F017 necesitan T012–T014;
>   - F009 y F014 necesitan T016–T018;
>   - F010–F011 necesitan T021, y F018 necesita T020 y T022;
>   - F019 necesita T024, y F021 el `openapi.json` final de T025.
> - **En paralelo con la API:** F001, F003–F008 y F012 no dependen de ella. Las respuestas se tipan a mano y las pruebas simulan la API, así que pueden empezar enseguida.
> - **Regla de todas las pantallas (RF-010):** cada tarea que agrega un selector, filtro o nombre de local prueba también que, con un solo local activo, no se pinta nada y las peticiones son las de `main`.
> - **Verificación completa:** `pnpm type-check && pnpm lint && pnpm test:unit --run && pnpm build-only`.

- [x] **F001** Rama `017-varios-establecimientos` desde `main` en el frontend — Verificación: `git branch --show-current`
- [x] **F002** (después de T007–T008) `pnpm api:types`. Luego:
  - `shared/establishments/types.ts`: `Establishment` y `EstablishmentRef`;
  - `establishments` y `company.active_establishments_count` en `core/auth/types.ts`;
  - `shared/establishments/api.ts`: `list(includeInactive)`, `create`, `update` y `assignToUser`, con sus pruebas.

  Verificación: `pnpm type-check` y pruebas en verde
- [x] **F003** [P] Pruebas de `useEstablishmentStore` (deben fallar):
  - `multi` solo con `active_establishments_count > 1`;
  - recuerda `current` en `sunat.establishment.<empresa>.<usuario>`, aparte para cada usuario y empresa;
  - con un solo permitido, lo toma solo;
  - un id guardado que ya no está permitido o activo deja `null`;
  - `noneAllowed` para el vendedor sin locales;
  - sin almacenamiento, funciona en memoria;
  - cerrar sesión no borra la elección.

  Verificación: en rojo
- [x] **F004** `shared/establishments/store.ts` — Verificación: F003 en verde
- [x] **F005** Pruebas y `EstablishmentSwitcher.vue`, y su lugar en `AppLayout` (barra lateral y contraída con `aria-label`):
  - con uno permitido es texto;
  - con varios, `BaseDialog` «¿En qué local trabajas?» con radios, el actual con `data-autofocus` y «Atrás» que lo cierra (`dismissTop`);
  - con una venta abierta, avisa «Tu venta en curso (N productos) queda guardada en …»;
  - sin `multi`, no se pinta.

  Verificación: en verde y `theme.spec.ts` en verde
- [x] **F006** [P] Pruebas y `EstablishmentFilter.vue`:
  - «Todos los locales» no manda `establishment_id`;
  - por defecto, «Todos» para el administrador y el local actual para el vendedor;
  - el administrador ve los inactivos con «(inactivo)»;
  - sin `multi`, no se pinta.

  Verificación: en verde
- [x] **F007** [P] Pruebas de `sale-draft.ts` por local (deben fallar):
  - claves `….<local>` para la venta y la preferencia;
  - cambiar de local guarda una venta y carga la otra, y volver recupera la primera;
  - la preferencia de serie es por local;
  - la clave de 0.3.0 se mueve solo al principal y se borra; en otro local no se toca;
  - `clearAll()` borra las ventas de todos los locales del usuario y nada de otro usuario;
  - sin local actual no se guarda nada.

  Verificación: en rojo
- [x] **F008** `sale-draft.ts` con el local en la clave, la migración y `clearAll()`, y `AppLayout.logout()` con `clearAll()`. Un 401 sigue sin borrar (principio VII; `AGENTS.md`) — Verificación: F007 y las pruebas de `sale-draft` y `AppLayout` en verde
- [ ] **F009** (después de T016–T018) Pruebas y cambios en «Vender» (`NewSaleView`, `ProductCatalog`, `SaleCart`, `sales/api.ts`):
  - catálogo, `availability` y cobro con `establishment_id`; el cobro usa el local de la venta guardada;
  - solo las series del local (`SeriesResource.establishment_id`);
  - sin local elegido con `multi`, el selector en lugar del catálogo;
  - el vendedor sin locales ve «No tienes un local asignado…» y no se carga el catálogo;
  - 422 «No puedes trabajar en ese local» al cobrar → refresca, pide elegir y la venta sigue en su clave;
  - al entrar, refresca los locales con `GET /establishments`.

  Verificación: en verde
- [ ] **F010** (después de T021) Pruebas y cambios en `TicketsView`, `SalesDocumentsView`, `TicketView`, `SalesDocumentView` y `TicketReceipt`:
  - `EstablishmentFilter` en las listas y el código del local en cada fila (sin columna nueva a 360 px);
  - nombre del local en el detalle, y nombre y dirección en el recibo, si `multi`.

  Verificación: en verde y `theme.spec.ts` en verde (`TicketReceipt` sigue en blanco y negro)
- [ ] **F011** [P] (después de T021) Pruebas y `EstablishmentFilter` en el inicio: las tarjetas, el gráfico, las recientes y los pendientes se recargan con el local — Verificación: en verde
- [x] **F012** [P] `UbigeoPicker` pasa de `platform/components` a `shared/ui` y recibe la función de búsqueda por prop: la plataforma le pasa `/platform/ubigeos/search` y la empresa, `/ubigeos/search`. Pruebas de la plataforma sin cambios — Verificación: suite completa en verde
- [x] **F013** (después de T007) Pruebas y cambios en «Mi empresa → Locales»:
  - sección «Locales» en `CompanyView` con «Principal» e «Inactivo» (`BaseBadge`);
  - rutas `/empresa/locales/nuevo` y `/empresa/locales/:id` (solo el administrador) con `EstablishmentFormView`;
  - código de 4 dígitos con `inputmode="numeric"`, deshabilitado con `code_locked`;
  - 422 de código repetido en el campo;
  - aviso ⚖️ A-89 visible;
  - «Desactivar» y «Activar» con confirmación; el principal no lo ofrece.

  Verificación: en verde y `router.spec.ts` en verde
- [ ] **F014** [P] (después de T018) Pruebas y cambios en `SeriesView`: las series se agrupan por local si `multi`, y «Nueva serie» pide el local (por defecto el principal) — Verificación: en verde
- [x] **F015** [P] (después de T007–T008) Pruebas y cambios en `UsersView` e `InviteUserDialog`:
  - los vendedores muestran sus locales y «Locales» abre un diálogo de casillas (al menos uno, solo activos) → `PUT /users/{user}/establishments`;
  - el administrador muestra «Todos los locales», sin acción;
  - al invitar a un vendedor con `multi`, las casillas, por defecto el local actual;
  - con un solo local no se piden.

  Verificación: en verde
- [x] **F016** (después de T014) Pruebas y cambios en las lecturas de inventario:
  - `ProductsView` con el filtro: el disponible es el del local o la suma;
  - `ProductDetailView` con «Stock por local» (`stock_by_establishment`);
  - `LotList` y `MovementList` con el filtro y el local en cada fila; los traslados se leen «Traslado a …» o «Traslado desde …» y no ofrecen «Revertir»;
  - `AlertsView` con el filtro y el local en cada alerta (un producto puede salir dos veces);
  - `StockEntryView` con «Local», por defecto el actual.

  Verificación: en verde
- [x] **F017** (después de T012) Pruebas y `TransferStockDialog.vue` en la ficha:
  - solo el administrador y con `multi`;
  - origen con su disponible; destino solo activo y distinto del origen;
  - 422 de stock insuficiente o destino inactivo en su campo (`useApiForm`);
  - al terminar, «Traslado registrado.» y recarga la ficha, los lotes y los movimientos;
  - «Atrás» cierra el diálogo.

  Verificación: en verde
- [ ] **F018** [P] (después de T020 y T022) Pruebas y cambios en `data-transfer`: `EstablishmentFilter` al exportar productos y ventas, y «Local del stock inicial» (por defecto el principal) en la vista previa de productos — Verificación: en verde
- [ ] **F019** [P] (después de T024) «Locales: N» en `CompanyDetailView` de la plataforma, con su prueba — Verificación: en verde
- [ ] **F020** Pruebas de RF-010 juntas, con un solo local activo:
  - «Vender», inicio, listas, ficha, alertas, «Usuarios», invitación, «Series» y exportar no muestran nada de locales;
  - las peticiones no llevan `establishment_id`, salvo las de escritura, que mandan el único local.

  Verificación: en verde
- [ ] **F021** (después de T025) `pnpm api:types` con el `openapi.json` final. `version` 0.4.0 en `package.json` — Verificación: verificación completa del frontend con código de salida 0 y cifras anotadas, y `pnpm android:dev` compila el APK *debug* (JDK 21)
- [ ] **F022** Verificación manual del frontend (plan, «Verificación manual del frontend»):
  - con un solo local, en web y en el teléfono, igual que en `main`;
  - con dos locales, en el teléfono real:
    - elegir el local;
    - armar una venta, cambiar de local y volver;
    - vender desde el segundo y trasladar;
    - «Atrás» en los diálogos nuevos;
    - modo avión con la venta abierta;
    - actualizar desde la 0.3.0 con una venta a medias;
  - 360 px y modo oscuro en «Vender», la barra lateral, «Mi empresa → Locales», «Usuarios» y la ficha;
  - TalkBack en el selector.

  Verificación: resultados en `verificacion.md`
- [ ] **F023** README del frontend: `shared/establishments`, `UbigeoPicker` en `shared/ui`, claves de `localStorage` por local, y la fila de `company` con «Locales» — Verificación: revisado

## Cierre

- [ ] **T026** Verificación manual (plan, pasos 1 a 5): CE-001 con una boleta desde el segundo local en SUNAT beta; migración de la base local con las mismas cifras; recorrido con dos locales; con un solo local igual que en `main` — Verificación: resultados en `verificacion.md`
- [ ] **T027** Comprobaciones del principio VI en verde (API SQLite y PostgreSQL, Pint; frontend completo) — Verificación: códigos de salida 0 y cifras anotadas
- [ ] **T028** Documentación: README de la API (locales, acceso, traslados), trampas en `AGENTS.md`, bitácora y estado en `docs/specs/README.md`. Si cambian las cuentas de prueba, `DevelopmentSeeder` con un segundo local de demostración. El README del frontend lo actualiza su sesión — Verificación: revisada
- [ ] **T029** 🔒 Merge local de `017-varios-establecimientos` a `main` en ambos repositorios y commit de la raíz — Verificación: suites en verde sobre `main`
