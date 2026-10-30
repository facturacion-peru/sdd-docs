# Plan 018: Consulta de DNI y RUC

**Estado:** Borrador · **Fecha:** 2026-10-30 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

> Reparto: la **API** la escribe la sesión de backend (este documento). La sección **Frontend** la completa la sesión del frontend antes de la aprobación.

## Resumen técnico

- **Una sola ruta de búsqueda**, `POST /customers/lookup` `{document_type, document_number}`. Busca primero entre los clientes de la empresa, luego en la tabla común `document_lookups` y al final en apisperu. Si el documento se encuentra, el cliente queda guardado en la empresa y la ruta lo devuelve con de dónde vino (`source`). Es POST porque puede crear o cambiar un cliente.
- **Cliente verificado** (A-92): `customers.verified_at`. Con fecha, el nombre y la dirección no se editan a mano (422). Una búsqueda que encuentra un cliente manual lo reemplaza y lo verifica.
- **Tabla común de la plataforma** (A-93 ⚖️): `document_lookups`, sin `company_id`. Guarda solo lo necesario para el comprobante (nombre o razón social, dirección, ubigeo, estado y condición del RUC) y la fecha. No guarda la respuesta cruda.
- **Servicio:** `App\DocumentLookup\ApisPeruClient` con el cliente HTTP de Laravel, así `Http::fake()` lo intercepta en las pruebas. Tiempo límite de 6 s, sin reintentos y con el token de `APISPERU_TOKEN` en la URL. Las respuestas se traducen a un resultado: `found`, `not_found`, `unavailable`, `quota` o `disabled`.
- **Sin topes** (A-96). Cada llamada real al servicio deja una fila en `document_lookup_requests` (empresa, usuario, tipo, resultado y hora; sin el número), que la plataforma suma por día.
- **Una búsqueda simultánea gasta una consulta** (CE-004): un `Cache::lock` por documento (el caché es la base de datos, compartido entre procesos) envuelve el «¿ya está? si no, consultar y guardar».
- **Actualizar** (A-94): `POST /customers/{customer}/refresh` consulta siempre y actualiza el cliente y la tabla común.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | API y frontend (incluida la app) |
| Ramas | `018-consulta-dni-ruc` en API y frontend, desde `main` |
| Dependencias nuevas | Ninguna |
| Migraciones | `customers.verified_at`; tablas `document_lookups` y `document_lookup_requests` |
| Variables nuevas | `APISPERU_TOKEN` (vacía en `.env.example` y en los ejemplos de `deploy/`; sin token, la búsqueda solo mira lo guardado) y `APISPERU_BASE_URL` (por defecto `https://dniruc.apisperu.com/api/v1`) |
| Forma de la respuesta | La documentación oficial no está publicada en texto. Lo que usan otros proyectos: `GET /dni/{n}?token=` con `nombres`, `apellidoPaterno` y `apellidoMaterno`, y `GET /ruc/{n}?token=` con `razonSocial`, `direccion`, `departamento`, `provincia`, `distrito`, `estado` y `condicion` (quizá `ubigeo`). **T002 lo confirma con el token del usuario** antes de escribir el cliente |

## Chequeo de constitución

| Principio | Cumple | Cómo |
|---|---|---|
| I. Spec primero | Sí | Spec 018 v1.0 aprobada, con A-92 a A-96 |
| II. Corrección fiscal | Sí | El comprobante sigue copiando los datos del cliente al emitir (spec 005), así que actualizar un cliente no cambia lo emitido. Un RUC de baja o no habido solo avisa (A-95 ⚖️) |
| III. Contrato de API | Sí | `LookupCustomerRequest` en OpenAPI. Los campos nuevos de `CustomerResource` se fijan en `tests/Feature/Contract` |
| IV. Seguridad | Sí | El token solo vive en el servidor: nunca va en respuestas, logs ni en la app (se prueba). El número se valida antes de consultar. La app nunca llama a apisperu |
| V. Simplicidad | Sí | Sin colas, sin topes y sin dependencias. Una ruta de búsqueda y otra de actualización |
| VI. Verificable | Sí | `Http::fake` para las respuestas, prueba concurrente en PostgreSQL (CE-004) y verificación con el token real |
| VII. Personas reales | Sí | Si el servicio falla, se sigue vendiendo con el registro manual (RF-010). La búsqueda de lo guardado no espera a la red |
| VIII. Aislamiento y auditoría | Sí, con ⚖️ | Los clientes siguen siendo de cada empresa; la tabla común solo tiene datos públicos y no se lista. Auditoría: `customer.verified` y `customer.refreshed`. ⚖️ Guardar DNI y nombres de forma central entra en A-24 |

## Modelo de datos

**`customers`**: se agrega `verified_at` (nulo). El cliente está verificado si tiene fecha.

**`document_lookups`** (de la plataforma, sin scope de empresa):
- campos: `document_type` (`1` o `6`), `document_number`, `name`, `address` (nulo), `ubigeo` (nulo), `ruc_status` (nulo), `ruc_condition` (nulo) y `fetched_at`;
- único `(document_type, document_number)`.

Solo se escribe desde `DocumentLookupService`. Ninguna ruta la lista.

**`document_lookup_requests`**: `company_id`, `user_id` (nulo), `document_type`, `outcome` (`found`, `not_found`, `unavailable` o `quota`) y `created_at`. Una fila por llamada real al servicio, sin el número del documento. Índice `(created_at)`.

**Nombre de una persona (DNI):** «APELLIDO_PATERNO APELLIDO_MATERNO NOMBRES», en mayúsculas, como SUNAT nombra a las personas con RUC 10. Así el mismo cliente se ve igual con DNI o con RUC.

## Búsqueda (`App\DocumentLookup\DocumentLookupService`)

`lookup(User $actor, CustomerDocumentType $type, string $number): LookupResult`

1. **Validar:** DNI de 8 dígitos; RUC de 11 dígitos con dígito verificador (`App\Rules\Ruc`). Si no cumple, 422 sin consultar (RF-005). Otros tipos de documento: 422 «Este documento se registra a mano».
2. **En la empresa:** si hay un cliente con ese documento, se devuelve con `source: company`. Si es manual, sigue al paso 3 para intentar verificarlo, pero sin bloquear: si el servicio falla, se devuelve el manual.
3. Con `Cache::lock("document-lookup:{type}:{number}", 10)->block(8)`:
   - **En la tabla común:** si está, se crea o actualiza el cliente de la empresa con esos datos y `verified_at` (`source: platform`, sin gastar consulta).
   - **Si no está:** `ApisPeruClient::find()`. Se registra la fila de uso y, si encuentra el documento, se guarda en la tabla común y en el cliente (`source: service`).
4. **Resultado:** `{customer: Customer|null, source: company|platform|service|null, reason: not_found|unavailable|quota|disabled|null}`.

Un cliente manual que pasa a verificado deja `customer.verified` en la auditoría, con el nombre anterior y el nuevo.

`refresh(Customer $customer, User $actor)`: igual que el paso 3, pero siempre consulta el servicio (A-94). Si falla, el cliente no cambia y la ruta responde 503 con el motivo. Si encuentra el documento, actualiza la tabla común y el cliente, y deja `customer.refreshed` en la auditoría.

**`ApisPeruClient`**: `GET {base}/{dni|ruc}/{number}?token=…`, con `timeout(6)` y sin reintentos.
- `found`: 200 con datos.
- `not_found`: 200 o 404 sin datos o con `success: false`.
- `quota`: 429, o un mensaje de límite (se confirma en T002).
- `unavailable`: 5xx, tiempo agotado o error de red.
- `disabled`: sin token, y no se llama al servicio.

El token nunca se registra en el log. Los errores se registran con el número enmascarado.

## Clientes

- **`CustomerResource`** agrega `verified` (bool), `verified_at`, `ruc_status` y `ruc_condition`. Los dos últimos salen de `document_lookups` con un `left join` y son nulos si no es RUC o si es manual.
- **`PATCH /customers/{customer}`** (administrador): con el cliente verificado, `name` y `address` dan 422 «Cliente verificado: sus datos vienen de SUNAT/RENIEC. Usa “Actualizar datos”.» `document_type` y `document_number` ya no se editan hoy (spec 005).
- **`POST /customers`** (registro manual): no cambia y crea clientes manuales. El frontend busca antes con `lookup`.
- **Importación** (spec 014, RF-012): `CustomerImport` no toca `name` ni `address` de un cliente verificado y deja una advertencia en la vista previa: «Cliente verificado: no se cambia su nombre ni dirección».

## Contrato de API

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | `/customers/lookup` | empresa | `{document_type: '1'\|'6', document_number}` → `{data: CustomerResource\|null, meta: {source, reason}}`. 200 si encontró o si el servicio falló (con `reason`); 422 si el formato no es válido |
| POST | `/customers/{customer}/refresh` | empresa | Vuelve a consultar → `CustomerResource`; 503 con `message` si el servicio no responde, no hay cupo o no hay token; 422 si el documento no es DNI ni RUC |
| GET | `/platform/document-lookups/usage` | plataforma | Últimos 14 días: `[{date, total, found, not_found, failed}]` (HU-4) |

Las dos rutas de empresa van en `TenantRoutes.php`. La de plataforma la cubren `PlatformAccessTest` y `PlatformPrivacyTest`, y no devuelve números de documento.

## Frontend

> Lo escribe la sesión del frontend. Rama `018-consulta-dni-ruc` del frontend, desde `main`. **Sin dependencias nuevas.** Versión 0.5.0 (de ahí sale `versionCode`).

### Lo que el frontend necesita del contrato

1. **Valores de `ruc_status` y `ruc_condition` normalizados**, por ejemplo siempre en mayúsculas y sin espacios de más (`ACTIVO`, `BAJA DE OFICIO`, `HABIDO`, `NO HABIDO`…). La app avisa cuando `ruc_status !== 'ACTIVO'` o `ruc_condition !== 'HABIDO'` (A-95) y muestra el texto tal cual.
2. **`verified_at`** en ISO 8601, como el resto de fechas.
3. **Un cliente manual que el servicio no pudo verificar** (paso 2 de la búsqueda) llega con `data` igual al cliente manual y `reason` igual al motivo. La app lo elige igual y muestra el motivo debajo.

### Tipos y API (`features/sales`)

- **`Customer`:** agrega `verified: boolean`, `verified_at: string | null`, `ruc_status: string | null` y `ruc_condition: string | null`. Se tipan a mano, como el resto de las respuestas.
- **`LookupResult`:** `{ customer: Customer | null; source: 'company' | 'platform' | 'service' | null; reason: 'not_found' | 'unavailable' | 'quota' | 'disabled' | null }`.
- **`customersApi.lookup(type, number, signal)`:** `POST /customers/lookup` con el `AbortSignal` de `openapi-fetch`; cancelar no muestra error.
- **`customersApi.refresh(id)`:** `POST /customers/{id}/refresh`. Un 503 llega como `ApiError` con su `message`.
- **`shared/utils/documents.ts`:** `isDni`, `isRuc` (11 dígitos y dígito verificador, el mismo cálculo que `App\Rules\Ruc`) y `documentKind(text)`. Sirven para avisar del formato **antes** de pedir nada (RF-005, HU-1 esc. 6). La API valida igual.

### `useDocumentLookup` (`features/sales/composables`)

Recibe el texto que se escribe y devuelve un estado:
- `idle` (no parece documento);
- `invalid` (con el mensaje: «El DNI tiene 8 dígitos.», «El RUC no es válido: revisa los números.»);
- `searching`;
- `found` (`customer`, `source`);
- `missing` (`reason`);
- `offline`.

**Reglas:**
- **Cuándo consulta:** al completar 8 dígitos (DNI) u 11 (RUC) y con 300 ms de espera sin teclear.
- **Una búsqueda a la vez:** cada búsqueda nueva aborta la anterior (`AbortController`). Una respuesta tardía nunca pisa a una más nueva: se compara el número.
- **Sin conexión** (`device().onConnectionChange`): no consulta. Busca solo entre los clientes ya cargados y queda en `offline` (caso límite).
- **Factura (`requireRuc`):** un DNI queda en `invalid` con «Para factura se necesita un RUC.», sin consultar.

### «Vender»: `CustomerPicker`

- **El campo de búsqueda es el mismo de hoy.** Si lo escrito es un DNI o un RUC, se usa `useDocumentLookup`; si es texto, la búsqueda por nombre de siempre.
- **Encontrado (HU-1 esc. 1 a 3):** el cliente **queda elegido solo**, sin un toque más (CE-001). En la tarjeta del cliente elegido se ve:
  - nombre o razón social y dirección;
  - la insignia «Verificado» (con la fecha en `title` y en texto para el lector) o «Manual»;
  - el estado del RUC.
  
  «Cambiar» vuelve a la búsqueda, como hoy.
- **RUC de baja o no habido (A-95 ⚖️):** `BaseAlert` de aviso dentro de la tarjeta, «Este RUC figura como BAJA DE OFICIO / NO HABIDO en SUNAT. Puedes emitir, pero confírmalo con el cliente.». No bloquea «Emitir».
- **No encontrado o servicio caído (HU-1 esc. 5, RF-010):** debajo del campo se muestra el motivo y el botón «Registrar a mano», que abre el diálogo de hoy con el tipo y el número ya puestos. La venta en curso no se toca. Los motivos:
  - `not_found`: «No encontramos ese DNI en RENIEC» o «ese RUC en SUNAT»;
  - `unavailable`: «El servicio de consulta no responde.»;
  - `quota`: «Se acabaron las consultas automáticas de hoy.»;
  - `disabled`: «La consulta automática no está disponible.».
- **Mientras busca:** «Buscando en RENIEC…» o «…en SUNAT…» con `aria-live="polite"`, así el lector de pantalla anuncia el resultado sin mover el foco.
- **«Actualizar datos» (HU-3):** botón chico en la tarjeta de un cliente con DNI o RUC, para el vendedor y el administrador. Si funciona, reemplaza el cliente de la venta en curso y avisa «Datos actualizados.»; si da 503, muestra el `message` y el cliente queda como estaba.
- **Carné de extranjería y otros (HU-1 esc. 7):** nada cambia; se registran a mano.
- **El ticket** (nombre y documento opcionales, en texto libre) no usa la búsqueda: no es un cliente guardado (spec 003).

### «Clientes» (`CustomersView`)

- Cada fila muestra la insignia y, si es RUC, el aviso de estado cuando corresponde.
- **El diálogo de edición de un cliente verificado** (HU-2 esc. 1) deshabilita «Nombre» y «Dirección», con la ayuda «Viene de RENIEC o SUNAT. Usa «Actualizar datos».». Si la API responde 422 igual (por ejemplo, otro usuario lo verificó mientras tanto), el mensaje se muestra en el campo.
- **«Actualizar datos»** en cada fila con DNI o RUC; al terminar, recarga la fila.

### Componentes compartidos de la feature

| Componente | Papel |
|---|---|
| `CustomerVerifiedBadge.vue` | «Verificado» (`success`, con «Consultado el 30 oct. 2026») o «Manual» (`neutral`) |
| `RucStatusAlert.vue` | El aviso de A-95; no se pinta si el RUC está activo y habido o si no es RUC |

Colores solo del tema (`theme.spec.ts`).

### Plataforma: «Consultas DNI/RUC» (HU-4)

- **Ruta y menú:** `/plataforma/consultas` (`platform-lookups`), en el menú del panel junto a «Soporte».
- **Tabla de los 14 días:** fecha, total, encontrados, no encontrados y fallidos, dentro de un `div` con desplazamiento horizontal a 360 px. Arriba, el total de hoy.
- Sin números de documento: la API no los manda.
- Sin gráfico en el MVP: la tabla responde la pregunta de cuándo pagar el plan.

### Pruebas del frontend (Vitest)

| Requisito / escenario | Prueba |
|---|---|
| RF-005, HU-1 esc. 6 | `documents.spec.ts`: DNI de 8 dígitos; RUC con dígito verificador válido e inválido (casos de la API); texto con letras, que no es documento |
| HU-1 esc. 1 a 3, CE-001 | `useDocumentLookup.spec.ts`: consulta al completar 8 u 11 dígitos tras la espera; aborta la anterior al seguir escribiendo; una respuesta vieja no pisa a la nueva; sin conexión, no consulta |
| HU-1 esc. 3 y 5, RF-010 | `CustomerPicker.spec.ts`: encontrado → queda elegido con «Verificado»; cada `reason` muestra su motivo y «Registrar a mano» abre el diálogo con el número; la venta no se toca |
| A-95 | `RucStatusAlert`: avisa con BAJA o NO HABIDO, no con ACTIVO y HABIDO; «Emitir» sigue habilitado |
| HU-2 esc. 1 y 4 | `CustomersView.spec.ts`: verificado → nombre y dirección deshabilitados y la insignia; manual → se editan; 422 en el campo |
| HU-3 | «Actualizar datos»: éxito reemplaza el cliente de la venta y avisa; 503 deja el cliente como estaba y muestra el `message` |
| Factura | Con `requireRuc`, un DNI da el error de formato sin pedir nada |
| HU-4 | `LookupUsageView.spec.ts`: tabla de 14 días y total de hoy; ruta solo de plataforma (`router.spec.ts`) |
| Tema | `theme.spec.ts` en verde |

**Verificación manual del frontend** (además de la de la API):
- **CE-001 en el teléfono real con datos móviles:** desde el último dígito de un RUC nuevo hasta ver la razón social, menos de 3 s, medido con las herramientas del WebView;
- servicio sin token o caído: emitir una boleta con un cliente registrado a mano (CE-003);
- 360 px y modo oscuro en el selector de cliente, la tarjeta con el aviso, «Clientes» y la tabla de la plataforma;
- TalkBack anuncia «Buscando…» y el resultado;
- modo avión con «Vender» abierto.

## Estrategia de pruebas

| Requisito / escenario | Prueba | Tipo |
|---|---|---|
| T002 | Script de prueba con el token real (DNI y RUC conocidos): imprime los campos que llegan, nunca el token | Manual, antes de escribir el cliente |
| RF-005, HU-1 esc. 6 | DNI de 7 dígitos, RUC con dígito verificador malo y carné de extranjería: 422 sin llamadas (`Http::assertNothingSent`) | Feature |
| HU-1 esc. 1–3, RF-001, CE-002 | El orden empresa → tabla común → servicio. Una segunda empresa que busca el mismo documento no llama al servicio | Feature |
| HU-1 esc. 4, A-95 | Un RUC «BAJA DE OFICIO» o «NO HABIDO» devuelve `ruc_status` y `ruc_condition`, y la factura con ese cliente se emite | Feature |
| HU-1 esc. 5, RF-010, CE-003 | 404 o sin datos → `not_found`; 500 o tiempo agotado → `unavailable`; 429 → `quota`; sin token → `disabled`. En todos los casos, `POST /customers` manual y la boleta siguen funcionando | Feature |
| HU-2, RF-002/003 | PATCH de nombre o dirección de un verificado: 422. Un manual encontrado pasa a verificado con auditoría. Uno manual no encontrado sigue editable | Feature |
| RF-012 | Importar un verificado no cambia su nombre ni su dirección y avisa | Feature |
| HU-3, A-94 | `refresh` llama siempre al servicio y actualiza; si falla, 503 sin cambios. Un comprobante ya emitido no cambia | Feature |
| HU-4, RF-013 | Una fila de uso por llamada real (ninguna con lo guardado), suma por día en la plataforma, sin números de documento | Feature |
| CE-004 | Dos procesos buscan a la vez el mismo documento nuevo (fake que cuenta llamadas en la base): una sola llamada y un solo cliente | Integration (PostgreSQL) |
| RF-011 | El token no aparece en ninguna respuesta ni en el log (`Log::spy`); la consulta lleva el token solo en la URL al servicio | Feature |
| Aislamiento | `CrossTenantAccessTest`: `lookup` y `refresh` de B; `document_lookups` no es de empresa y no se expone | Feature |
| Contrato | `CustomerResource` con los campos nuevos; `meta` de `lookup` | Feature |
| Suites | SQLite, PostgreSQL y Pint; frontend completo | Suites |

## Verificación manual (antes de cerrar)

Con `APISPERU_TOKEN` en el `.env` del usuario:
1. **T002:** confirmar la forma de las respuestas (DNI y RUC conocidos).
2. **CE-001:** en el teléfono con datos móviles, cronometrar desde terminar de escribir un RUC nuevo hasta ver la razón social (menos de 3 s).
3. Una factura con un RUC traído del servicio, aceptada en SUNAT beta.
4. Con un token inválido: aviso claro y registro manual.

## Riesgos

| Riesgo | Mitigación |
|---|---|
| apisperu cambia la forma de la respuesta o se cae | Un solo punto de traducción (`ApisPeruClient`), con pruebas de cada forma conocida. Si falla, el registro manual sigue (RF-010) |
| El cupo gratuito se agota en días de mucho uso | Lo guardado se reutiliza entre empresas. El uso por día está a la vista de la plataforma, y pasar al plan pagado es solo cambiar el token (A-96) |
| Datos personales en una tabla común ⚖️ | Solo los campos del comprobante, sin respuesta cruda, sin listado y sin el número en los registros de uso. Va a la revisión de A-24 antes de producción |
| El token en un log o en una respuesta | Prueba con `Log::spy` y con las respuestas. El cliente nunca registra la URL completa |
| Respuesta lenta del servicio | Tiempo límite de 6 s. Lo guardado responde sin red |
