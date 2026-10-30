# Tareas 018: Consulta de DNI y RUC

**Estado:** Aprobadas (2026-10-30) · **Fecha:** 2026-10-30 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior. 🔒 = requiere confirmación explícita del usuario. 🔑 = necesita `APISPERU_TOKEN` en el `.env` del usuario.
> Reparto: las tareas **T** (API y cierre) las lleva la sesión de backend; las **F** (frontend), la sesión del frontend.

## Fase 1 · Preparación (API)

- [x] **T001** Rama `018-consulta-dni-ruc` desde `main` en la API. `APISPERU_TOKEN` y `APISPERU_BASE_URL` en `config/services.php`, `.env.example` y `deploy/env.*.example` (vacías) — Verificación: `git branch --show-current` y suites en verde
- [x] **T002** 🔑 Confirmar la forma real de las respuestas: un comando de un solo uso (fuera del repositorio) consulta un DNI y un RUC conocidos y un número inexistente, e imprime los campos y el código HTTP, **nunca el token**. Si cambia lo que supone el plan, se corrige el plan primero — Verificación: forma anotada en `verificacion.md`

## Fase 2 · API: servicio y datos

- [ ] **T003** Migraciones `customers.verified_at`, `document_lookups` y `document_lookup_requests`; modelos `DocumentLookup` (sin scope de empresa) y `DocumentLookupRequest` — Verificación: migran y revierten en SQLite y PostgreSQL
- [ ] **T004** Pruebas de `ApisPeruClient` con `Http::fake` (deben fallar):
  - DNI → nombre «APELLIDO_PATERNO APELLIDO_MATERNO NOMBRES»;
  - RUC → razón social, dirección, ubigeo si viene, y estado y condición normalizados;
  - sin datos o 404 → `not_found`; 5xx o tiempo agotado → `unavailable`; 429 o mensaje de límite → `quota`; sin token → `disabled`, sin llamar;
  - el token va solo en la URL al servicio y nunca en el log (`Log::spy`).

  Verificación: en rojo
- [ ] **T005** `App\DocumentLookup\ApisPeruClient` y su resultado — Verificación: T004 en verde

## Fase 3 · API: búsqueda y clientes (HU-1, HU-2)

- [ ] **T006** Pruebas de `POST /customers/lookup` y entrada en `TenantRoutes.php` (deben fallar):
  - formato inválido, RUC con dígito malo y carné de extranjería: 422 sin llamadas;
  - el orden empresa → tabla común → servicio, con `source`;
  - una segunda empresa no gasta consulta (CE-002);
  - un manual encontrado pasa a verificado con `customer.verified`;
  - un manual no encontrado vuelve con `reason`;
  - cada falla con su `reason` y una fila de uso por llamada real;
  - el vendedor también busca.

  Verificación: en rojo
- [ ] **T007** `LookupCustomerRequest`, `DocumentLookupService::lookup` (con `Cache::lock`), la ruta y los campos nuevos de `CustomerResource` — Verificación: T006 en verde
- [ ] **T008** [P] Pruebas y cambios: PATCH de nombre o dirección de un verificado da 422; la importación de clientes no cambia un verificado y avisa (RF-012); un RUC de baja o no habido se emite en factura (A-95) — Verificación: en verde
- [ ] **T009** CE-004 en `tests/Integration` (PostgreSQL, con forks): dos búsquedas simultáneas del mismo documento nuevo hacen una sola llamada y dejan un solo cliente. Mutación: sin el lock, falla — Verificación: en verde y en rojo con la mutación

## Fase 4 · API: actualizar y uso (HU-3, HU-4)

- [ ] **T010** Pruebas y `POST /customers/{customer}/refresh`: llama siempre al servicio y actualiza el cliente y la tabla común con `customer.refreshed`; si falla, 503 sin cambios; un comprobante ya emitido no cambia — Verificación: en verde
- [ ] **T011** [P] Pruebas y `GET /platform/document-lookups/usage` (14 días, sin números de documento); `PlatformAccessTest` y `PlatformPrivacyTest` en verde — Verificación: en verde
- [ ] **T012** Contrato en `tests/Feature/Contract` (`CustomerResource` y el `meta` de `lookup`), `CrossTenantAccessTest` con `lookup` y `refresh` de B, `php artisan openapi:generate`, suites SQLite y PostgreSQL y Pint — Verificación: códigos de salida 0 y cifras anotadas

## Fase 5 · Frontend

> Las lleva la sesión del frontend, según la sección «Frontend» del plan.
> - **Dependencias de la API** (por el `pnpm api:types` de las rutas nuevas):
>   - F004 y F006 necesitan T007;
>   - F007 necesita T008;
>   - F008 necesita T010;
>   - F009 necesita T011;
>   - F011 necesita el `openapi.json` final de T012.
> - **En paralelo con la API:** F001–F003 y F005 no dependen de ella. Las respuestas se tipan a mano y las pruebas simulan la API, así que pueden empezar enseguida.
> - **Verificación completa:** `pnpm type-check && pnpm lint && pnpm test:unit --run && pnpm build-only`.

- [x] **F001** Rama `018-consulta-dni-ruc` desde `main` en el frontend — Verificación: `git branch --show-current`
- [x] **F002** [P] Pruebas y `shared/utils/documents.ts`:
  - `isDni` (8 dígitos);
  - `isRuc` (11 dígitos y dígito verificador, con los mismos casos que `App\Rules\Ruc`);
  - `documentKind(text)`: `dni`, `ruc` o `null` para texto o números de otra largura.

  Verificación: en verde
- [x] **F003** [P] Tipos a mano: `Customer` con `verified`, `verified_at`, `ruc_status` y `ruc_condition`, y `LookupResult` en `features/sales/types.ts`. Luego `CustomerVerifiedBadge.vue` y `RucStatusAlert.vue`, con sus pruebas:
  - «Verificado» con la fecha y «Manual»;
  - aviso con BAJA o NO HABIDO, sin aviso con ACTIVO y HABIDO ni si no es RUC.

  Verificación: en verde y `theme.spec.ts` en verde
- [x] **F004** (después de T007) `pnpm api:types`. `customersApi.lookup(type, number, signal)` y sus pruebas (cuerpo, `AbortSignal`, `data` nulo con `reason`, 422 de formato) — Verificación: `pnpm type-check` y pruebas en verde
- [x] **F005** [P] Pruebas y `useDocumentLookup` (con `lookup` simulado):
  - consulta al completar 8 u 11 dígitos tras 300 ms;
  - aborta la anterior al seguir escribiendo; una respuesta vieja no pisa a la nueva;
  - formato inválido → `invalid` sin pedir nada; con `requireRuc`, un DNI da «Para factura se necesita un RUC.»;
  - sin conexión → `offline`, sin pedir nada.

  Verificación: en verde
- [x] **F006** (después de T007) Pruebas y cambios en `CustomerPicker` (HU-1):
  - un documento encontrado queda elegido solo, con la insignia y el estado del RUC;
  - cada `reason` muestra su motivo y «Registrar a mano» abre el diálogo con el tipo y el número;
  - un manual que no se pudo verificar queda elegido, con el motivo debajo;
  - «Buscando…» y el resultado con `aria-live`;
  - la venta en curso no cambia si la búsqueda falla;
  - el texto sigue buscando por nombre.

  Verificación: en verde
- [ ] **F007** (después de T008) Pruebas y cambios en `CustomersView` (HU-2): insignia y aviso de RUC en cada fila; en un verificado, «Nombre» y «Dirección» deshabilitados con su ayuda; un 422 llega a su campo; un manual se edita como siempre — Verificación: en verde
- [ ] **F008** (después de T010) `customersApi.refresh(id)` y «Actualizar datos» en `CustomerPicker` y `CustomersView` (HU-3), con pruebas: si funciona, reemplaza el cliente (también el de la venta en curso) y avisa «Datos actualizados.»; un 503 muestra el `message` y no cambia nada — Verificación: en verde
- [ ] **F009** [P] (después de T011) Pruebas y `LookupUsageView` en `/plataforma/consultas`, con su entrada en el menú del panel: tabla de 14 días con desplazamiento a 360 px y el total de hoy; ruta solo de plataforma (`router.spec.ts`) — Verificación: en verde
- [ ] **F010** README del frontend: `useDocumentLookup`, `shared/utils/documents`, insignia y aviso del RUC, y la vista de uso de la plataforma — Verificación: revisado
- [ ] **F011** (después de T012) `pnpm api:types` con el `openapi.json` final. `version` 0.5.0 en `package.json` — Verificación: verificación completa del frontend con código de salida 0 y cifras anotadas, y el APK *debug* compila (JDK 21)
- [ ] **F012** Verificación manual del frontend (plan):
  - **CE-001 en el teléfono real con datos móviles:** menos de 3 s desde el último dígito de un RUC nuevo, medido con las herramientas del WebView;
  - CE-003: el servicio caído o sin token, y una boleta con un cliente registrado a mano;
  - 360 px y modo oscuro en el selector, la tarjeta con el aviso, «Clientes» y la tabla de la plataforma;
  - TalkBack anuncia «Buscando…» y el resultado;
  - modo avión con «Vender» abierto.

  Verificación: resultados en `verificacion.md`

## Cierre

- [ ] **T013** 🔑 Verificación manual (plan, pasos 1 a 4): CE-001 cronometrado en el teléfono con datos móviles, una factura con un RUC traído del servicio aceptada en SUNAT beta y el aviso con un token inválido — Verificación: resultados en `verificacion.md`
- [ ] **T014** Comprobaciones del principio VI en verde (API SQLite y PostgreSQL, Pint; frontend completo) — Verificación: códigos de salida 0 y cifras anotadas
- [ ] **T015** Documentación: README de la API (consulta de documentos, variables y uso), trampas en `AGENTS.md`, bitácora y estado en `docs/specs/README.md`. El README del frontend lo actualiza su sesión — Verificación: revisada
- [ ] **T016** 🔒 Merge local de `018-consulta-dni-ruc` a `main` en ambos repositorios y commit de la raíz — Verificación: suites en verde sobre `main`
