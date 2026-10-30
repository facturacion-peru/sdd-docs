# Plan 017: Varios establecimientos

**Estado:** Aprobado (2026-10-29) · **Fecha:** 2026-10-29 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.1)

> Reparto: la **API** y la migración de datos las escribe la sesión de backend (este documento). La sección **Frontend** la completa la sesión del frontend antes de la aprobación.

## Resumen técnico

- **El local ya existe** (`establishments`, con `code`, `is_main` y series colgando de él). Se agrega `active` y se abre su alta y edición al administrador de empresa (`/establishments`).
- **Todo lo que pasa en un local lo lleva escrito:** `establishment_id` en lotes, movimientos, tickets y comprobantes. La migración lo rellena con el principal de cada empresa (A-91) y luego lo vuelve obligatorio.
- **Stock por local (A-86):** el lote pertenece a un local; el stock de un producto en un local es la suma de sus lotes allí. `InventoryService` recibe el local en cada operación. El bloqueo sigue siendo la fila del producto, así las operaciones de un producto siguen serializadas en el mismo orden, también los traslados entre dos locales (sin deadlocks).
- **Traslado:** una operación nueva, `transfer`, que saca FEFO del origen y entra al destino en un lote con el mismo número, fechas y costo, dentro de una transacción. Queda una fila en `stock_transfers` como origen (`source`) de los dos movimientos.
- **Usuarios (A-87):** tabla `establishment_user`. El administrador de empresa tiene todos los locales sin filas; el vendedor, los asignados. Un servicio, `EstablishmentAccess`, decide qué locales puede usar cada usuario y lo aplica en escrituras (422) y lecturas (filtro, y 404 en el detalle).
- **El local va explícito en cada petición:** `establishment_id` en el cuerpo de las escrituras y en la consulta de las lecturas, documentado en OpenAPI por los `FormRequest`. Si se omite y el usuario tiene **un solo** local permitido, se usa ese: así una empresa con un local, y la app 0.3.0 ya instalada, siguen funcionando igual.
- **Tickets (A-88):** el correlativo sigue por empresa; el ticket guarda el local y lo muestra (PDF y ESC/POS).
- **Comprobantes:** el local sale de la serie (como hoy). La nota de crédito usa una serie de notas del **local del original** y repone el stock allí.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | API y frontend (incluida la app Android) |
| Ramas | `017-varios-establecimientos` en API y frontend, desde `main` |
| Dependencias nuevas | Ninguna |
| Migraciones | `establishments.active`; `establishment_user`; `stock_transfers`; `establishment_id` en `product_lots`, `inventory_movements`, `tickets`, `sales_documents` e `invitations` (lista de locales de la invitación); cambio del único de lotes |
| Datos existentes | Todo al principal de cada empresa; vendedores asignados al principal (A-91). Se prueba que las cifras no cambian (CE-003) |
| Compatibilidad | La app 0.3.0 sigue funcionando con un solo local permitido. `APP_ANDROID_MIN_VERSION` se sube a la nueva versión cuando se despliegue (spec 009) |

## Chequeo de constitución

| Principio | Cumple | Cómo |
|---|---|---|
| I. Spec primero | Sí | Spec 017 v1.0 aprobada, con A-86 a A-91 |
| II. Corrección fiscal | Sí | El comprobante lleva el código de anexo y la dirección del local de su serie, copiados al emitir (como hoy). La nota usa la serie de notas del local del original. CE-001 en SUNAT beta. ⚖️ A-89: el código lo pone el administrador, sin consulta a SUNAT |
| III. Contrato de API | Sí | `establishment_id` y las rutas nuevas salen de `FormRequest`; recursos fijados en `tests/Feature/Contract` |
| IV. Seguridad | Sí | Ningún id de local se acepta sin pasar por `EstablishmentAccess`; un vendedor no lee ni escribe en locales que no tiene (CE-002) |
| V. Simplicidad | Sí | Sin almacenes aparte ni flujo de aprobación: el local es el establecimiento ya existente. Sin contexto implícito en cabeceras: el local va en la petición |
| VI. Verificable | Sí | Pruebas de acceso entre locales generadas desde el router, migración con datos (CE-003) y traslado concurrente en PostgreSQL (CE-004) |
| VII. Personas reales | Sí | Con un solo local no cambia nada en pantalla (RF-010). La venta en curso es por local |
| VIII. Aislamiento y auditoría | Sí | Las tablas nuevas tienen `company_id` y scope de empresa, y entran en `TenantRoutes.php`. Auditoría: `establishment.created`, `establishment.updated`, `establishment.deactivated`, `establishment.activated`, `user.establishments_changed`, `inventory.transfer` |

## Modelo de datos

**`establishments`** (existe): se agrega `active` (por defecto `true`). El principal no se desactiva. Único `(company_id, code)` ya existe; `code` exige 4 dígitos.

**`establishment_user`** (nuevo): `company_id`, `establishment_id`, `user_id`, único `(establishment_id, user_id)`. Solo se usa con vendedores: el administrador de empresa trabaja en todos (A-87). Al pasar un vendedor a administrador las filas se conservan, pero no se leen; al volver a vendedor, vuelven a valer.

**`invitations`**: se agrega `establishment_ids` (JSON, nulo). Al aceptar una invitación de vendedor, se crean sus filas en `establishment_user`. Si la empresa tiene un solo local activo, se asigna ese aunque no venga en la invitación.

**`product_lots`**: `establishment_id` (obligatorio). El único pasa de `(product_id, lot_number)` a `(product_id, establishment_id, lot_number)`: el mismo lote puede estar repartido entre locales tras un traslado. Índice `(establishment_id, product_id)`.

**`inventory_movements`**: `establishment_id` (obligatorio, el del lote). `product_balance_after` pasa a ser el saldo del producto **en ese local**; en los datos existentes (todo en el principal) vale lo mismo. Tipos nuevos en `MovementType`: `transfer_out` y `transfer_in`.

**`stock_transfers`** (nuevo): `company_id`, `product_id`, `from_establishment_id`, `to_establishment_id` (distintos), `quantity`, `note` (nulo), `created_by`, `created_at`. Es el `source` de sus movimientos.

**`tickets`**: `establishment_id` (obligatorio). El correlativo sigue en `ticket_sequences` por empresa (A-88).

**`sales_documents`**: `establishment_id` (obligatorio, el de la serie al emitir). Repite el de la serie para filtrar sin unir tablas. No se desalinea: `series.establishment_id` queda fijo una vez que la serie tiene comprobantes (`SeriesService`).

**Migración** (una sola, reversible en local):
1. agrega las columnas como nulas;
2. rellena con el principal de cada empresa (`sales_documents` desde su serie);
3. asigna cada vendedor activo o inactivo al principal;
4. las vuelve obligatorias, con claves foráneas e índices.

Se prueba en `tests/Integration` con datos sembrados antes de migrar (CE-003).

## Reglas de acceso (`App\Establishments\EstablishmentAccess`)

- `allowed(User): Collection` — activos de la empresa para el administrador; los asignados y activos para el vendedor.
- `resolve(User, ?int $id, string $field = 'establishment_id'): Establishment` — para escrituras. Sin id: el único permitido, o 422 «Elige el local» si hay varios; con un id fuera de `allowed`: 422 «No puedes trabajar en ese local». Un local de otra empresa no existe para el scope de empresa: el mismo 422, sin distinguir.
- `filter(User, ?int $id): ?array` — para lecturas. Sin id: los ids permitidos (el administrador ve «Todos»); con id: ese, si está permitido (si no, 422). Los inactivos **sí** se leen (historial).
- `scopeVisibleTo(User)` en `Ticket`, `SalesDocument`, `ProductLot` e `InventoryMovement`: limita a los locales que el usuario puede leer. Las rutas de detalle (`/tickets/{ticket}`, `/sales-documents/{salesDocument}` y sus PDF, XML, CDR e impresión) responden 404 si no es visible, como entre empresas.

## Inventario (`InventoryService`)

Todas las firmas reciben el local; el bloqueo sigue siendo la fila del producto (orden único, sin deadlocks).

| Operación | Cambio |
|---|---|
| `registerEntry(Product, Establishment, data, actor)` | El lote nace en ese local |
| `consume(Product, Establishment, qty, source, actor)` | FEFO/FIFO solo entre los lotes del local. `InsufficientStock` informa el disponible **del local** |
| `adjust(lot, …)` | Sin cambio de firma: el local es el del lote |
| `reverse(movement, …)` | Además de ventas de tickets y comprobantes, rechaza los movimientos de traslado: se corrige con otro traslado |
| `restock(Product, qty, original, note, actor)` | Sin cambio: repone en los lotes de los que salió la venta, que ya son del local del original |
| `transfer(Product, from, to, qty, note, actor)` (nuevo) | Dentro de una transacción: bloquea el producto; toma FEFO los lotes no vencidos del origen (como `consume`); por cada uno, sale del origen (`transfer_out`) y entra (`transfer_in`) en el lote del destino con el mismo número, o lo crea con las mismas fechas y costo. Si falta stock, 422 sin cambios. Destino inactivo: 422. Origen inactivo: se permite (sacar el stock de un local cerrado) |
| `balance(Product, ?Establishment)` | Saldo en un local, o en todos |

`Product::scopeWithStock(?array $establishmentIds)`: las sumas `stock_sum` y `available_sum` filtradas por local. `stock` y `available_stock` en `ProductResource` son entonces del local pedido, o de todos los permitidos si se pide «Todos».

**Alertas:** `min_stock` sigue siendo del producto y se compara **con el stock de cada local**: un producto bajo el mínimo en dos locales da dos alertas, cada una con su local. Los lotes por vencer llevan su local.

## Venta

- **Tickets** (`StoreTicketRequest`): `establishment_id` opcional (regla anterior). `TicketService::issue` resuelve el local, lo guarda, consume en ese local y mantiene el correlativo de empresa. La idempotencia no cambia.
- **Comprobantes** (`StoreSalesDocumentRequest`): el local es el de `series_id`. Si viene `establishment_id`, debe coincidir (422). La serie debe estar activa, en un local activo y permitido al usuario (422). La selección automática de serie (`SalesDocumentService`, cuando no viene `series_id`) busca solo entre las series del local resuelto.
- **Notas de crédito:** el local es el del comprobante original. `noteSeries` busca y valida series de notas **de ese local**; si no hay, el mensaje lo dice: «No hay una serie de notas de crédito de boletas activa en Sucursal Centro…».
- **Descartar un rechazado** (spec 007): repone en el local del comprobante, ya incluido en sus movimientos.
- **Series** (`SeriesController`): `establishment_id` al crear (por defecto el principal, como hoy). No se cambia una vez que la serie tiene comprobantes. `GET /series` acepta `establishment_id` y por defecto lista las de los locales permitidos.

## Pantallas de la API que se filtran (A-90)

Aceptan `establishment_id` (opcional) y, sin él, usan los locales permitidos:

| Ruta | Detalle |
|---|---|
| `GET /dashboard` | Ventas, notas, recientes y pendientes del local o de todos los permitidos |
| `GET /tickets`, `GET /sales-documents` | Filtro y columna `establishment` (`{id, code, name}`) en cada fila |
| `GET /products`, `/products/{id}`, `/catalogs/inventory` | Stock del local o la suma |
| `GET /products/{id}/lots`, `/products/{id}/movements` | Del local o de todos, con `establishment` en cada fila |
| `GET /inventory/alerts` | Por local, con `establishment` en cada alerta |
| `GET /exports/products`, `/exports/sales` | Columna `local` (código y nombre); la hoja de productos suma el stock de los locales pedidos, y la de lotes trae una fila por lote y local |
| `POST /imports/products/preview` | `establishment_id` para el stock inicial (por defecto, el principal) |

## Contrato de API

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/establishments` | empresa | Locales permitidos al usuario (el administrador ve también los inactivos con `?include_inactive=1`) |
| POST | `/establishments` | administrador | Crea un local: `code` (4 dígitos, único), `name`, `address`, `ubigeo` |
| PATCH | `/establishments/{establishment}` | administrador | Edita nombre, dirección, ubigeo y `active`. El código no cambia si el local ya tiene comprobantes; el principal no se desactiva |
| PUT | `/users/{user}/establishments` | administrador | Locales de un vendedor (`establishment_ids`, al menos uno activo) |
| POST | `/products/{product}/transfers` | administrador | Traslado: `from_establishment_id`, `to_establishment_id`, `quantity`, `note`. Responde 201 con el `ProductResource` (con `stock_by_establishment`) |
| POST | `/invitations` | administrador | Acepta `establishment_ids`: al menos uno activo si el rol es vendedor y la empresa tiene varios locales activos; con uno solo, se asigna ese. `InvitationResource` lo devuelve |
| — | Rutas de venta, stock y listas | — | `establishment_id` según las secciones anteriores |

**Recursos:**
- `EstablishmentResource`: `id`, `code`, `name`, `address`, `ubigeo` (con nombres), `is_main`, `active` y `code_locked` (ya tiene comprobantes: el código no se puede cambiar).
- **Sesión:** la respuesta de `login`, la de aceptar la invitación y `/auth/me` traen `establishments` (los permitidos) y `company.active_establishments_count`. El vendedor solo recibe sus locales, así que la cuenta es lo que le dice a la app si la empresa tiene varios (RF-010 frente a HU-2, esc. 1). `CompanyResource` trae lo mismo.
- `SeriesResource` agrega `establishment_id`.
- Detalle de ticket y de comprobante: `establishment {id, code, name, address}`. En las filas de las listas, `establishment {id, code, name}`.
- `GET /products/{id}` agrega `stock_by_establishment: [{establishment_id, stock, available_stock}]`, solo con los locales permitidos.
- `UserResource` agrega `establishment_ids` para el administrador.
- `PlatformCompanyDetailResource` agrega `establishments_count`.
- **Ubigeo:** el formulario de locales usa `GET /ubigeos/search`, que ya existe para la empresa (grupo `tenant`) con el mismo método que el de la plataforma. No cambia.

**Ticket impreso:** el PDF de 80 mm y `TicketPrint` agregan el nombre y la dirección del local (cambian juntos por `PrintParityTest`). El PDF del comprobante ya trae la dirección del local; se agrega su nombre si no es el principal, también en `DocumentPrint`.

## Frontend

> Lo escribe la sesión del frontend. Rama `017-varios-establecimientos` del frontend, desde `main`. **Sin dependencias nuevas.** Versión 0.4.0 (de ahí sale `versionCode`); al desplegar, `APP_ANDROID_MIN_VERSION` sube a 0.4.0.

### Lo que el frontend necesita del contrato

Pedidos a la API, además de lo descrito arriba:

1. **`establishments` también en la respuesta de `login` y de aceptar la invitación**, no solo en `/auth/me`. La sesión nace de esas respuestas: sin el dato, la app no sabe si mostrar el selector hasta recargar.
2. **`company.active_establishments_count`** en la sesión. El vendedor recibe solo sus locales, así que con uno asignado no puede saber si la empresa tiene más. Con eso decide RF-010: con un solo local activo en la empresa no se ve nada, y con varios el vendedor ve el nombre de su local aunque no pueda cambiarlo (HU-2, esc. 1).
3. **Buscador de ubigeo para la empresa:** ya existe (`GET /api/v1/ubigeos/search`, grupo `tenant`, el mismo `UbigeoController@searchUbigeo` que el de `/platform`), así que en la API no cambia nada. El formulario de locales reusa `UbigeoPicker` y le pasa la función que llama a esa ruta.
4. **En `EstablishmentResource`, `code_locked`** (true si ya tiene comprobantes). Así el formulario deshabilita el código, en vez de esperar el 422.
5. **`establishment_id` en `SeriesResource`.** «Vender» filtra las series del local actual, y «Series» las agrupa por local.
6. **`establishment {id, code, name, address}` en el detalle** de ticket y comprobante, no solo en las filas. El ticket web (`TicketReceipt`, `window.print()`) lo imprime.
7. **`stock_by_establishment: [{establishment_id, stock, available_stock}]` en `GET /products/{id}`**, solo con los locales permitidos. El traslado muestra el disponible de cada origen, y la ficha muestra el stock por local, sin una llamada por local.
8. **`POST /products/{product}/transfers`** responde 201 con el producto (`ProductResource` con `stock_by_establishment`). La app refresca la ficha; los lotes y movimientos los vuelve a pedir.
9. **`POST /invitations`** acepta `establishment_ids` (al menos uno activo si el rol es vendedor y la empresa tiene varios locales). `InvitationResource` lo devuelve.

### Local actual (`shared/establishments/`)

Lo usan varias features (venta, inventario, inicio y listas), así que no va dentro de ninguna. Tiene la misma forma que `printer-settings` (spec 015).

- **`types.ts`:** `Establishment = {id, code, name, address, ubigeo, is_main, active, code_locked}` y `EstablishmentRef = {id, code, name}` (lo que traen las filas). Se tipan a mano, como el resto de las respuestas.
- **`store.ts` (`useEstablishmentStore`):**
  - **`allowed`:** sale de `session.establishments` y se refresca con `GET /establishments` al entrar a «Vender» (un local creado o desactivado durante la sesión).
  - **`multi`:** `company.active_establishments_count > 1`. Es la única condición de RF-010: ningún selector, filtro ni columna de local se pinta sin ella.
  - **`current`:** el local donde se trabaja, recordado **por dispositivo** en `sunat.establishment.<empresa>.<usuario>` (solo el id). Al iniciar:
    - si el id guardado está entre los `allowed` activos, se usa;
    - si hay un solo permitido, se toma solo;
    - si no, queda `null` y la app pide elegir (HU-2, esc. 2 y 5).

    No se borra al cerrar sesión, como la preferencia de serie: es del dispositivo.
  - **`noneAllowed`:** el vendedor sin locales activos. «Vender» muestra «No tienes un local asignado. Pide al administrador que te asigne uno.» y no carga el catálogo (caso límite).
- **`EstablishmentSwitcher.vue`:**
  - Muestra el nombre del local actual en la barra lateral, bajo la empresa, y en la cabecera de «Vender».
  - Con más de un permitido, es un botón que abre un `BaseDialog` con la lista (radios, `data-autofocus` en el actual). «Atrás» lo cierra (`useDismissable`).
  - Con uno solo, es texto. Sin `multi`, no se pinta.
- **`EstablishmentFilter.vue`:** un `BaseSelect` con «Todos los locales» y los locales.
  - Para el administrador, también los inactivos (`?include_inactive=1`), porque su historial se sigue viendo, con «(inactivo)».
  - **Por defecto:** «Todos» para el administrador y el local actual para el vendedor (A-87: trabaja en uno a la vez).
  - Con «Todos» no se manda `establishment_id`: la API ya usa los permitidos.
  - Sin `multi`, no se pinta.

### Venta en curso y preferencia por local (specs 012 y A-60)

- **Claves:**
  - venta en curso: `sunat.sale-draft.<empresa>.<usuario>.<local>`;
  - preferencia de comprobante y serie: `sunat.sale-preference.<empresa>.<usuario>.<local>` (la serie es de un local).

  `storageKey` depende de `current`. Cambiar de local guarda la venta del local anterior y carga la del nuevo: nunca se mezclan, y al volver al local anterior la venta sigue ahí (HU-2, esc. 2).
- **Al cambiar de local con una venta abierta,** el diálogo lo dice antes de confirmar: «Tu venta en curso (3 productos) queda guardada en Principal.»
- **Migración de las claves de 0.3.0 (sin local):** al cargar, si no existe la clave del local y sí la antigua, y el local actual es el **principal**, se mueve la antigua a la nueva y se borra. Todo lo de antes estaba en el principal (A-91), así que no se pierde una venta a medias al actualizar.
- **Cerrar sesión** (`AppLayout.logout()` → `draft.clearAll()`) borra las ventas en curso de **todos** los locales del usuario: recorre las claves `sunat.sale-draft.<empresa>.<usuario>.*`. Así no quedan ventas de un local desactivado en el teléfono. Un 401 sigue sin borrarlas (principio VII).
- **La idempotencia no cambia:** la clave viaja con cada venta guardada, y una venta solo se cobra en el local donde se armó, porque `establishment_id` sale del local de la clave, no del actual.
- **Local desactivado mientras se vende:** el cobro recibe 422 «No puedes trabajar en ese local». La app refresca `allowed`, deja `current` en `null`, pide elegir otro y conserva la venta en su clave. No se pasa sola al local nuevo (caso límite).

### Pantallas

| Dónde | Cambio |
|---|---|
| `AppLayout.vue` | `EstablishmentSwitcher` en la barra lateral, bajo el nombre de la empresa; contraído (spec 010 v1.4) solo queda el ícono con `aria-label` |
| `NewSaleView.vue` / `ProductCatalog.vue` / `SaleCart.vue` | El catálogo, `availability` (series) y el cobro van con `establishment_id` del local actual. Las series se filtran por `establishment_id` del local. Sin local elegido (`multi` y `current === null`), se muestra el selector en lugar del catálogo |
| `TicketView.vue`, `SalesDocumentView.vue`, `TicketReceipt.vue` | Nombre del local (y dirección en el recibo) si `multi` |
| `TicketsView.vue`, `SalesDocumentsView.vue` | `EstablishmentFilter` y el código del local en cada fila (texto pequeño, sin columna nueva a 360 px) |
| Inicio (`dashboard`) | `EstablishmentFilter` arriba; las tarjetas y el gráfico se recargan al cambiarlo |
| `ProductsView.vue` | `EstablishmentFilter`: el disponible es el del local elegido o la suma. El precio no cambia (A-90) |
| `ProductDetailView.vue` | «Stock por local» (lista corta con `stock_by_establishment`) si `multi`. `LotList` y `MovementList` con el filtro y el local en cada fila; los movimientos de traslado se leen «Traslado a Sucursal» o «Traslado desde Principal» y no ofrecen «Revertir» |
| `TransferStockDialog.vue` (nuevo, inventario) | Solo para el administrador y con `multi`. Campos: origen (con su disponible), destino (solo activos y distinto del origen), cantidad y nota. Usa `useApiForm` para los 422 (stock insuficiente, destino inactivo). Al terminar, «Traslado registrado.» y recarga la ficha |
| `StockEntryView.vue` | «Local» (`BaseSelect`, por defecto el actual) si `multi` |
| `AlertsView.vue` | `EstablishmentFilter` y el local en cada alerta (un producto puede salir dos veces) |
| «Mi empresa → Locales» (`company`) | Una sección «Locales» en `CompanyView` con la lista: código, nombre, «Principal» e «Inactivo» con `BaseBadge`. «Nuevo local» y «Editar» van a `/empresa/locales/nuevo` y `/empresa/locales/:id` (`EstablishmentFormView`, solo para el administrador) |
| `EstablishmentFormView.vue` (nuevo) | Código (4 dígitos, `inputmode="numeric"`, deshabilitado si `code_locked`), nombre, dirección y ubigeo con `UbigeoPicker`, que pasa a `shared/ui` y recibe la función de búsqueda por prop. Aviso fijo ⚖️ (A-89): «Usa el código de anexo de tu ficha RUC. Si no existe en SUNAT, tus comprobantes serán observados o rechazados.» «Desactivar» o «Activar» pide confirmación con `BaseDialog`; el principal no lo ofrece |
| `SeriesView.vue` | Las series se agrupan por local si `multi`, y «Nueva serie» pide el local (por defecto el principal) |
| `UsersView.vue` | Los vendedores muestran sus locales y una acción «Locales» con un diálogo de casillas (al menos uno, solo activos) → `PUT /users/{user}/establishments`. El administrador muestra «Todos los locales», sin acción |
| `InviteUserDialog.vue` | Con rol vendedor y `multi`: casillas de locales, por defecto el actual. Con un solo local no se piden: la API asigna ese |
| Exportar e importar (`data-transfer`) | `EstablishmentFilter` en la exportación de productos y de ventas; «Local del stock inicial» (por defecto el principal) en la vista previa de productos |
| Plataforma (`CompanyDetailView`) | «Locales: N» con `establishments_count` |

**Lo que no cambia:**
- clientes, precios, configuración de SUNAT y certificado (A-90);
- la nota de crédito: el local sale del original, y el diálogo solo muestra la serie que propone la API;
- la impresión Bluetooth: los bytes ya traen el local.

### 360 px y accesibilidad

- El nombre del local se trunca con `truncate` dentro de `min-w-0`, y el selector abre un diálogo, no un menú desplegable.
- Los filtros comparten la fila del grid existente (`grid-cols-1` en el celular).
- Colores solo del tema (`theme.spec.ts`). Las casillas y los radios llevan `label`, y el diálogo del selector tiene título («¿En qué local trabajas?»).

### Pruebas del frontend (Vitest)

| Requisito / escenario | Prueba |
|---|---|
| RF-010, A-91 | `EstablishmentSwitcher`, `EstablishmentFilter` y `NewSaleView` con un solo local activo: no hay selector, filtro ni nombre de local, y las peticiones no cambian respecto a `main` |
| HU-2 esc. 1–2 | `establishment-store.spec.ts`: recuerda el local por empresa y usuario; con un solo permitido lo toma solo; con un id guardado que ya no está permitido, queda en `null` |
| HU-2 esc. 2, spec 012 | `sale-draft.spec.ts`: cambiar de local guarda y carga ventas distintas; volver recupera la primera; la preferencia de serie es por local; migración de la clave 0.3.0 solo al principal; `clearAll()` borra todos los locales del usuario y nada de otro usuario |
| HU-2 esc. 5 | `NewSaleView.spec.ts`: 422 «No puedes trabajar en ese local» al cobrar → pide elegir y la venta sigue en su clave; el vendedor sin locales ve el mensaje y no se carga el catálogo |
| HU-2 esc. 3 | El cobro (ticket y comprobante) manda el `establishment_id` del local de la venta; las series del catálogo son solo las del local |
| HU-3 | `TransferStockDialog.spec.ts`: el destino excluye el origen y los inactivos; el 422 de stock insuficiente se ve en el campo; al terminar, recarga. `MovementList`: un traslado no ofrece «Revertir» |
| HU-1 | `EstablishmentFormView.spec.ts`: 4 dígitos, código deshabilitado con `code_locked`, el principal sin «Desactivar», 422 de código repetido en el campo y aviso A-89 visible |
| HU-4 | `EstablishmentFilter`: «Todos» no manda el parámetro; por defecto «Todos» para el administrador y el local actual para el vendedor. Inicio, listas y alertas recargan al cambiarlo |
| A-87 | `UsersView` / `InviteUserDialog`: asignación con al menos un local; el administrador sin acción |
| Tema y router | `theme.spec.ts` y `router.spec.ts` siguen en verde con las vistas nuevas |

**Verificación manual del frontend** (además de los pasos 1 a 4 de la API):
- **Con un solo local** (RF-010), en web y en el teléfono: «Vender», inicio, listas y ficha iguales a `main`;
- **con dos locales, en el teléfono real:**
  - elegir el local al entrar;
  - armar una venta, cambiar de local y volver;
  - vender desde el segundo y trasladar;
  - «Atrás» en los diálogos nuevos;
  - modo avión con la venta abierta;
  - actualizar desde la 0.3.0 con una venta a medias (migración de la clave);
- 360 px y modo oscuro en «Vender», la barra lateral, «Mi empresa → Locales», «Usuarios» y la ficha;
- TalkBack en el selector de local.

## Estrategia de pruebas

| Requisito / escenario | Prueba | Tipo |
|---|---|---|
| HU-1, RF-001 | Crear, editar, desactivar y reactivar; código repetido, sin 4 dígitos o cambiado con comprobantes: 422. El principal no se desactiva. El vendedor recibe 403 | Feature |
| RF-002, CE-001 | Una boleta con serie del segundo local lleva su `CodLocal` y su dirección en el XML (`UblBuilder`) | Feature + SUNAT beta en la verificación |
| HU-2, RF-003/004 | Ticket y comprobante quedan en el local resuelto; sin `establishment_id` y un solo local permitido, el de siempre; con varios, 422; en uno no permitido o inactivo, 422; serie de otro local con `establishment_id` distinto, 422 | Feature |
| HU-2 esc. 4 | Venta en el local B solo descuenta lotes de B; con stock solo en A, `InsufficientStock` con el disponible de B; la nota repone en B | Feature |
| HU-3, RF-005 | Traslado con FEFO de varios lotes: saldos de origen y destino, mismo número, fechas y costo; sin stock, 422 sin cambios; destino inactivo, 422; un movimiento de traslado no se revierte | Feature |
| CE-004 | Traslado y venta simultáneos en el origen, y dos traslados cruzados A→B y B→A (con forks en PostgreSQL): stock final correcto, sin deadlock | Integration (PostgreSQL) |
| CE-002, A-87 | `CrossEstablishmentAccessTest`, generado desde el router como `CrossTenantAccessTest`: un vendedor de A no ve en ninguna respuesta tickets, comprobantes, lotes ni movimientos de B (marcadores), y el detalle de B da 404. Toda ruta de empresa nueva queda en `TenantRoutes.php` | Feature |
| RF-007, HU-4 | Dashboard, listas, alertas y exportaciones filtradas por local y con «Todos» | Feature |
| RF-008 | Auditoría de locales, asignaciones y traslados | Feature |
| RF-009, CE-003 | Base con datos de las specs 002–007 sembrada **antes** de la migración: tras migrar, el inicio, el stock, las listas y los saldos de movimientos son idénticos; todo en el principal y los vendedores asignados a él | Integration |
| Invitaciones | La invitación guarda los locales y al aceptarla se asignan; con un solo local, el principal | Feature |
| Impresión | `PrintParityTest` con el nombre y la dirección del local | Feature |
| Contrato | `EstablishmentResource`, `establishment` en filas y `establishments` en `/auth/me` | Feature |
| Suites | SQLite, PostgreSQL y Pint; frontend completo | Suites |

## Verificación manual (antes de cerrar)

1. **CE-001:** crear un segundo local con un código de anexo de la empresa demo en beta, una serie `B002` y emitir una boleta: aceptada, con el código y la dirección del local en el XML.
2. Migración sobre la base local con datos (`DevelopmentSeeder`): mismas cifras en el inicio antes y después (CE-003).
3. Recorrido con dos locales: entrada en el principal, traslado al segundo, venta del vendedor del segundo, nota de crédito con devolución y alertas por local.
4. Con un solo local, las pantallas se ven como en `main` (RF-010).
5. Pantallas a 360 px y en la app Android (lo detalla la sección Frontend).

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Una consulta de stock o de ventas se queda sin filtro de local y mezcla cifras | Todo pasa por `EstablishmentAccess`; `CrossEstablishmentAccessTest` recorre todas las rutas desde el router con marcadores del otro local |
| La migración cambia cifras de empresas existentes | Prueba de migración con datos (CE-003) y verificación manual con la base local |
| Bloquear la fila del producto serializa ventas del mismo producto en locales distintos | Ya pasa hoy dentro de una empresa y dura milisegundos. Se mantiene a propósito: es lo que hace los traslados seguros y sin deadlocks |
| El mismo lote en dos locales confunde (mismo número) | Cada fila de lote muestra su local; el número igual es intencional para seguir el vencimiento |
| La app instalada (0.3.0) en una empresa con varios locales | El vendedor con un solo local sigue funcionando; con varios, la API responde 422 «Elige el local» y se resuelve subiendo `APP_ANDROID_MIN_VERSION` al desplegar |
| Código de anexo inexistente ⚖️ | Aviso al crear el local (A-89) y el rechazo de SUNAT se ve como hoy (spec 005) |
