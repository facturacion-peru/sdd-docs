# AGENTS.md

Notas para agentes (y personas) que trabajen en este espacio. Aquí va lo importante que **no** está en los README ni se deduce leyendo el código: decisiones, reglas y trampas conocidas. Mantenerlo breve; el detalle vive en el README de cada proyecto.

## Proyectos

| Carpeta | Qué es | Stack | README |
|---|---|---|---|
| `Api-de-facturacion-electronica-sunat-Peru/` | API REST multiempresa (empresas, usuarios, inventario, tickets y emisión SUNAT en beta). El panel de plataforma irá en el frontend Vue (spec 006, A-35) | Laravel 12, PHP 8.2+, Greenter 5.1, Sanctum, Pest, PostgreSQL | [README](https://github.com/facturacion-peru/Api-de-facturacion-electronica-sunat-Peru#readme) |
| `frontend-api-facturacion-electronica-sunat/` | App de todos los usuarios de empresa, incluido su administrador (A-08), y del administrador de la plataforma en `/plataforma` (A-35). Futuro APK con Capacitor | Vue 3 + TS 5.9, Vite 8, Pinia, Tailwind 4, `openapi-fetch` | [README](https://github.com/facturacion-peru/frontend-api-facturacion-electronica-sunat#readme) |

Hay tres repositorios git independientes ([sdd-docs](https://github.com/facturacion-peru/sdd-docs), [API](https://github.com/facturacion-peru/Api-de-facturacion-electronica-sunat-Peru) y [frontend](https://github.com/facturacion-peru/frontend-api-facturacion-electronica-sunat)): la **raíz** (solo `README.md`, `docs/`, `AGENTS.md` y `CLAUDE.md`; ignora las dos carpetas de proyecto y `respaldos/`), la **API** (fork de `yorchavez9/Api-de-facturacion-electronica-sunat-Peru`, remoto `upstream`) y el **frontend** (desde la tarea T002 de la spec 001). Son **públicos** (A-46): nada secreto entra en ellos. Cómo clonarlos juntos: [README](README.md).

## Reglas

- **Spec-Driven Development.** Los principios no negociables están en [`docs/constitution.md`](docs/constitution.md); leerla antes de proponer cualquier cambio. Cada funcionalidad pasa por `docs/specs/NNN-nombre/` (`spec.md` → `plan.md` → `tasks.md`) con aprobación del usuario entre pasos.
  - Punto de entrada: [`docs/specs/README.md`](docs/specs/README.md) (estado de cada spec). Contexto de negocio: [`docs/producto/vision.md`](docs/producto/vision.md).
  - Las dudas se registran en [`docs/aclaraciones.md`](docs/aclaraciones.md) con un identificador `A-NN`; las decide el usuario, nunca el agente.
  - **No escribir código de producción** para una spec que no esté en estado *Lista*.
- **Solo pnpm en el frontend.** Nunca `npm`, `npx` ni `yarn` (riesgo de ataques de supply-chain). Usar `pnpm add`, `pnpm dlx`, `pnpm <script>`.
  - Las protecciones están en `frontend-.../pnpm-workspace.yaml` (`minimumReleaseAge`, `strictDepBuilds`, `blockExoticSubdeps`, `trustPolicy`). No desactivarlas para "hacer que funcione".
  - Si una dependencia necesita scripts de instalación, aprobarla con `pnpm approve-builds` tras revisarla.
  - El `preinstall` de `package.json` rechaza npm/yarn/bun, pero npm lo ejecuta *después* de descargar: es un aviso, no un bloqueo total.
- **El frontend se construye paso a paso**, un módulo (`src/features/<dominio>`) a la vez, a pedido del usuario.
- **Contrato con la API vía OpenAPI.** Si cambian rutas o `FormRequest` en Laravel: `php artisan openapi:generate` en la API y luego `pnpm api:types` en el front. Nunca editar `src/core/api/schema.d.ts` a mano.
- **Idioma:** documentación, commits y mensajes al usuario en español.

## Cómo levantar todo

```bash
# API
cd Api-de-facturacion-electronica-sunat-Peru && php artisan serve --host=127.0.0.1 --port=8000
# Frontend (proxy /api → :8000)
cd frontend-api-facturacion-electronica-sunat && pnpm dev
```

Verificación del frontend: `pnpm type-check && pnpm lint && pnpm test:unit --run && pnpm build-only`.
Pruebas de la API: `php artisan test` (SQLite) **y** `composer test:pgsql` (PostgreSQL, base `db_api_sunat_testing`), más `vendor/bin/pint --test` (excluye `legacy/` y `lang/`).
Primer arranque local: `php artisan migrate:fresh --seed` (crea las cuentas de prueba). Para los reintentos de envío a SUNAT, `php artisan schedule:work` en otra terminal. Fuera de local: `php artisan platform:create-admin` y `php artisan company:create` (interactivos).

## Trampas conocidas

- pnpm 11 no define `npm_config_user_agent` en los scripts: no sirve para detectar pnpm.
- TypeScript fijado en `~5.9` porque `openapi-typescript` aún no soporta TS 6. Revisar cuando lo haga.
- El spec OpenAPI documenta *requests* pero no *responses*: las respuestas se tipan a mano en el `types.ts` de cada feature y las protegen las pruebas de `tests/Feature/Contract`.
- **Toda ruta de empresa nueva** va en `tests/Feature/Tenancy/TenantRoutes.php` con su tipo de prueba; si no, `TenantRoutesCoverageTest` falla.
- `ResolveTenant` está en la lista de prioridad **antes de `SubstituteBindings`**: sin eso, el *route model binding* consultaría sin contexto de empresa (500).
- `User` no tiene scope de empresa: `{user}` en rutas se resuelve con un binding propio en `AppServiceProvider`, solo entre miembros de la empresa del contexto.
- SQLite (suite rápida) no detecta todo: `FOR UPDATE` con `count()` falla solo en PostgreSQL. Correr también `composer test:pgsql`.
- En pruebas, el guard de Laravel conserva el usuario entre peticiones de una misma prueba: usar `app('auth')->forgetGuards()` para simular una petición nueva.
- Pint: `vendor/bin/pint --test` revisa el código mantenido. Para formatear, pasar solo los archivos de la tarea.
- `migrate:fresh --seed` en local regenera ubigeos y las cuentas de prueba de `DevelopmentSeeder` (solo `APP_ENV=local`, contraseña `Clave-demo-123`). El usuario autorizó refrescar la base local (2026-09-28); los datos que no vengan del seeder se pierden. Nunca ejecutarlo fuera de local.
- Las pruebas de PHP salen marcadas como *deprecated* por `PDO::MYSQL_ATTR_SSL_CA` dentro de `vendor/` (Laravel 12.28 + PHP 8.5). Se resuelve actualizando Laravel dentro de la 12.x.
- `tests/Integration` corre sin la transacción envolvente de `RefreshDatabase`: úsalo para límites de transacción y concurrencia. Dentro de `tests/Feature`, `DB::transactionLevel()` ya vale 1.
- Las pruebas que llaman a modelos de empresa fuera de una petición (servicios, recursos, relaciones) necesitan `app(TenantContext::class)->set($empresa)`. Si falla con `TenantContextMissing` o `TenantMismatch`, casi siempre es la prueba y no el código.
- En SQLite, `SUM()` sobre columnas decimales devuelve coma flotante y `bcadd` trunca (12.3449999 → 12.344). Sumar en PHP con `App\Support\Decimal::sum()`, no con `->sum()`.
- El generador de OpenAPI solo documenta parámetros de consulta y cuerpos que vienen de un **FormRequest**. Si la validación está en línea (`$request->validate`), el frontend queda sin tipos. Los numéricos se documentan como cadenas decimales.
- Antes de un commit, verificar con el **código de salida** de las suites (`php artisan test; echo $?`), nunca con `| grep … &&`: `grep` tiene éxito aunque la suite falle.
- La factory de establecimientos no genera el código `0000` (es el del principal): colisionaba con el índice único y daba pruebas intermitentes.
- El `TenantContext` persiste entre peticiones de una misma prueba: si la prueba crea datos de **otra** empresa después de una petición, limpiarlo antes con `app(TenantContext::class)->clear()`.
- Las ventas de un ticket solo se revierten anulando el ticket (`InventoryService::reverse(..., fromSource: true)` desde `TicketService`).
- Chrome headless (`--print-to-pdf`) ignora `@page size`: para comprobar los 80 mm hace falta el diálogo de impresión real.
- Los dos layouts del router cuelgan de «/» con la misma prioridad: gana el primero declarado. `AppLayout` debe ir antes que `AuthLayout` (si no, recargar en «/» muestra un AuthLayout vacío); lo protege `app/router/__tests__/router.spec.ts`.
- `DevelopmentSeeder` es idempotente: `php artisan db:seed --class=DevelopmentSeeder` agrega lo que falte (p. ej. productos nuevos del surtido) sin borrar datos locales; no hace falta `migrate:fresh`.
- `DevelopmentSeeder` intenta validar la empresa demo contra SUNAT beta (red real). Sus pruebas simulan la respuesta con `Http::fake`; sin red, la configuración demo queda pendiente y el seeder no falla.
- Certificados de prueba: `Tests\Support\TestCertificates::make()` (autofirmados con el RUC en `serialNumber`, `OU` o `CN`). Para uno ya vencido fuera de las pruebas, `openssl req -x509 … -not_before … -not_after …` (OpenSSL 3.5).
- **SUNAT beta limita la frecuencia:** si se envía un documento a los pocos segundos de otro, responde `HTTP 401 Unauthorized`; a los 5 s lo acepta. El envío al emitir reintenta una vez (`SUNAT_RATE_LIMIT_PAUSE`, 0 en `phpunit.xml`). Beta tampoco detecta reenvíos ni ofrece la consulta de CDR.
- Greenter usa `SoapClient`, no el cliente HTTP de Laravel: `Http::fake()` no lo intercepta. En pruebas se reemplaza `SunatSender` por `FakeSunatSender` (`app()->instance(...)`).
- `tests/Beta/` (spike y `SunatBetaTest`) va contra SUNAT beta real y no está en `phpunit.xml`: `php artisan test tests/Beta/SunatBetaTest.php --group=sunat-beta`.
- Para simular una caída de SUNAT sin tocar `.env`: `SUNAT_BETA_ENDPOINT=https://127.0.0.1:9/billService php artisan serve --port=8001 --no-reload`. Sin `--no-reload`, `artisan serve` no pasa las variables de entorno al servidor.
- `composer update --lock` tras cambiar `require` necesita `--ignore-platform-req=php`: algunas dependencias aún no declaran PHP 8.5.
- `php artisan tinker archivo.php` se queda en modo interactivo; usar `tinker --execute="require '...';"`.
- Las rutas de plataforma resuelven el rol **antes** del *route model binding* (`EnsurePlatformAdmin` en la lista de prioridad, como `ResolveTenant`); si no, un usuario de empresa distinguiría ids existentes (404) de inexistentes (403).
- `/auth/me` usa `tenant:allow-platform`: deja pasar al administrador de la plataforma (sin empresa) y conserva la revocación de tokens de cuentas o empresas desactivadas.
- **Toda ruta nueva de plataforma** queda cubierta por `PlatformAccessTest` y `PlatformPrivacyTest` (se generan desde el router). Sus respuestas usan recursos `Platform*Resource` de lista blanca, nunca los de empresa.
- **Notas de crédito:** la plantilla `notacr2.1` de Greenter no admite descuentos de línea; `UblBuilder` expresa esas líneas por sus valores netos. No volver a poner `setDescuentos` en líneas de nota.
- Las ventas y reposiciones de un comprobante no se revierten desde el inventario: solo con una nota de crédito o descartando el rechazado (`reverse(..., fromSource: true)`).
- Los errores por línea de la API usan claves planas (`"lines.0.quantity"`): en las pruebas, `->json('errors')['lines.0.quantity']`, no `assertJsonPath` (que trata los puntos como niveles).
- El historial de la API se reescribió el 2026-09-30 para quitar un `.env.beta` del autor original con sus credenciales (auditoría de la 009). Los *hashes* de la API anteriores a esa fecha que aparezcan en documentos ya no existen; la copia está en `respaldos/`.
- **Colores solo del tema:** las pantallas usan los tokens de `main.css` (`brand-*`, `success-*`, `warning-*`, `danger-*`, `ink`, `line`, `subtle`…), nunca `red-*`, `green-*`, `amber-*` ni `slate-*`; `shared/ui/__tests__/theme.spec.ts` lo comprueba (excepción: `TicketReceipt`, que se imprime en blanco y negro). Si cambia el logo (`LogoApp.vue`), actualizar también `public/favicon.svg`.
- **Campos:** `BaseInput`, `BaseSelect` y `SearchInput` comparten `shared/ui/field.ts`; ninguna vista escribe clases propias en un `<input>` de texto ni usa `<select>` directo (lo comprueba `theme.spec.ts`). Las búsquedas usan `SearchInput` con `@clear` apuntando al mismo manejador que recarga la lista.
- **Modo oscuro:** `main.css` redefine los mismos tokens bajo `:root[data-theme='dark']`; `useTheme` siempre deja `data-theme` en `<html>`. Cada tono tiene un papel fijo (50–200 fondos tenues, 300–400 gráficos, 600 relleno con texto blanco, 700 texto de acento, 800 *hover* del relleno, 900–950 siempre oscuros): no usar `700` como fondo ni `600` como texto, o se rompe en oscuro.
- En Vue, `:inert="cond"` con `false` deja `inert="false"` en el DOM y el navegador lo trata como inerte: usar `:inert="cond || undefined"`.
- **Venta en curso:** `sunat.sale-draft.<empresa>.<usuario>` en `localStorage`. Solo `AppLayout.logout()` la borra al salir; no engancharla a `onTokenCleared` (también salta con un 401 y se perdería la venta, principio VII).
- `sr-only` sobre un `<table>` no la oculta bien (ignora el ancho y el `caption` puede verse): ponerlo en un `div` que la envuelva.
- En un grid de una columna con texto truncado, usar `grid-cols-1` (`minmax(0,1fr)`): la columna implícita crece con el texto y desborda a 360 px.
- `trustPolicyExclude` en `pnpm-workspace.yaml` admite solo versiones exactas revisadas y aprobadas por el usuario (hoy `semver@6.3.1`, falso positivo). Ante un `ERR_PNPM_TRUST_DOWNGRADE` nuevo: revisar, preguntar y documentar; nunca desactivar `trustPolicy`.
- El código heredado de `legacy/` no se ejecuta: antes de reincorporar algo, leer `docs/investigacion/001-evaluacion-api-existente.md`.

## Bitácora

Entradas breves y fechadas de decisiones o cambios relevantes (la más reciente arriba).

- **2026-10-03** — Spec 012 implementada en la rama `012-vender-mejorado` del frontend (desde la 011): venta en curso en el store `sales/stores/sale-draft` guardada por empresa y usuario (se borra al cobrar, vaciar o cerrar sesión; un 401 no la borra), catálogo para tocar con «Cargar más», «Vaciar carrito» con confirmación, dos columnas desde `lg` y barra con hoja en el celular. v1.1: «Vender» a pantalla completa en escritorio (`meta.fullWidth`, catálogo y carrito con desplazamiento propio) y barra lateral contraíble a íconos (spec 010 v1.4, recordada en `sunat.sidebar.collapsed`). v1.2–1.3: el selector Ticket / Boleta / Factura va dentro del carrito, en el orden del cobro (productos → comprobante → cliente → medio de pago). `DevelopmentSeeder` suma 41 productos de bodega para ver la paginación (rama `012-vender-mejorado` de la API). 211 pruebas; CE-002 verificado contra la API local. Falta commit y merge.

- **2026-10-03** — Specs 010 (identidad visual, v1.3) y 011 (panel de inicio) implementadas y confirmadas; falta el merge. 010 en la rama `010-identidad-visual` del frontend (desde la 009); 011 en `011-panel-inicio` de la API (desde la 009) y del frontend (desde la 010). Spec 012 (Vender: carrito persistente y dos columnas, A-56 a A-59) en estado *Lista*. Detalle de la 010: tema «Petróleo» con tokens de marca, neutros y estados, logo provisional `LogoApp.vue` y favicon SVG, Inter autoalojada (`semver@6.3.1` excluido de `trustPolicy` tras revisarlo), campos unificados (`BaseSelect`, `SearchInput` con lupa y botón de limpiar) modo claro/oscuro con selector (A-55, fondo claro `#eef2f6`) y barra lateral en `AppLayout` (panel desplegable en el celular; desaparece el menú «Más»). 011: `GET /api/v1/dashboard` en la rama `011-panel-inicio` de la API (A-52 a A-54; 639 pruebas SQLite y 650 PostgreSQL) y el nuevo Inicio en el frontend (195 pruebas). Pantallas a 360 px y en escritorio revisadas.

- **2026-09-30** — Repositorios publicados en GitHub (organización `facturacion-peru`, públicos; T031): `sdd-docs` (raíz, rama renombrada de `master` a `main`), API y frontend con todas sus ramas. README en la raíz y enlaces absolutos entre los tres. Spec 009 avanzada hasta T023 (copias, alertas, scripts de servidor, restauración y GitHub Actions); el despliegue sigue apagado (`DEPLOY_ENABLED`) hasta tener servidor.

- **2026-09-30** — Spec 007 integrada en `main` de la API y del frontend (merge local, T075); suites en verde sobre `main`. Siguientes candidatas: 008 (paso a producción SUNAT, tras la revisión legal A-24) y 009 (despliegue y operación).
- **2026-09-30** — Spec 007 implementada (T001–T074) en la rama `007-notas-credito` de API y frontend: 609 pruebas de API (SQLite y PostgreSQL, con devoluciones concurrentes y consistencia aleatoria) y 162 de frontend. SUNAT beta aceptó sin observaciones notas de anulación, devolución parcial y total; recorrido de extremo a extremo 15/15 y pantallas a 360 px revisadas. Aclaraciones A-39 ⚖️ a A-42. Hallazgos: descuentos de línea en notas (Greenter), reversión manual de ventas de comprobantes, y `required_if` en el generador de OpenAPI. Pendiente: CE-003 en el celular, revisión contable (A-39) y el merge (T075).
- **2026-09-29** — Spec 006 integrada en `main` de la API y del frontend (merge local, T074); suites en verde sobre `main`. Todas las capacidades del mapa de la visión (001–006) están integradas. Queda lo previo a producción: A-24 ⚖️ (datos personales, retención y 2FA de la plataforma) y la revisión contable de A-11, A-17 y A-22.
- **2026-09-29** — Spec 006 implementada (T001–T073) en la rama `006-panel-plataforma` de API y frontend: 555 pruebas de API (SQLite y PostgreSQL, dos mutaciones) y 152 de frontend; recorrido de extremo a extremo 19/19 y pantallas a 360 px revisadas. Aclaraciones A-35 a A-38 (panel en Vue, A-08 actualizada). La verificación corrigió que la plataforma perdiera la sesión al recargar (`/auth/me`, desde la 001), una fuga 404/403 en rutas de plataforma y el generador de OpenAPI con objetos anidados. Pendiente: CE-001 cronometrado, 2FA antes de producción y el merge (T074).
- **2026-09-29** — Spec 005 integrada en `main` de la API y del frontend (merge local, T095); suites en verde sobre `main`. Las specs 001–005 del MVP están completas; queda la spec 006 (panel de plataforma) y lo pendiente de producción (A-24 ⚖️).
- **2026-09-29** — Spec 005 implementada (T001–T094) en la rama `005-emision-comprobantes` de API y frontend: 520 pruebas de API (SQLite y PostgreSQL, con emisión y envío concurrentes y tres mutaciones) y 137 de frontend. SUNAT beta real aceptó sin observaciones los 8 casos de CE-001; recorrido de extremo a extremo 16/16 con caída simulada y pantallas a 360 px revisadas. Aclaraciones A-22 ⚖️, A-23 y A-32 a A-34. La verificación corrigió las observaciones 4096–4098 (ubigeo del emisor) y el menú en rutas hermanas. Requisito `ext-gd`. Pendiente: CE-003 en el celular, revisión contable (A-22) y el merge (T095).
- **2026-09-29** — Spec 004 integrada en `main` de la API y del frontend (merge local, T074); suites en verde sobre `main`. Siguiente: aclaraciones de la spec 005 (A-22 ⚖️, A-23).
- **2026-09-29** — Spec 004 implementada (T001–T073) en la rama `004-configuracion-sunat` de API y frontend: 380 pruebas de API (SQLite y PostgreSQL, con correlativo concurrente y prueba de no filtración de secretos) y 114 de frontend; recorrido de extremo a extremo 21/21 con validación real contra SUNAT beta y pantallas a 360 px revisadas. Secretos cifrados con `APP_KEY` (requisito `ext-openssl`). La verificación encontró que «/» mostraba un AuthLayout vacío al recargar (desde la 001) y se corrigió. Pendiente: probar con un certificado real y el merge (T074).
- **2026-09-29** — Spec 003 integrada en `main` de la API y del frontend (merge local, T084); suites en verde sobre `main`. Siguiente: aclaraciones de la spec 004 (A-20, A-21).
- **2026-09-28** — Spec 003 implementada (T001–T083) en la rama `003-tickets-venta`: 317 pruebas de API (SQLite y PostgreSQL, con numeración concurrente real) y 87 de frontend; recorrido de extremo a extremo 13/13 y pantallas a 360 px revisadas. Aclaraciones A-17 a A-19, A-29 y A-30. Navegación con menú «Más». Pendiente: cronometrar una venta (CE-001), probar la impresión en una térmica real y el merge (T084).
- **2026-09-28** — Spec 002 integrada en `main` de la API y del frontend (merge local, T094); suites en verde sobre `main`. Siguiente: aclaraciones de la spec 003 (A-15, A-17 ⚖️, A-18, A-19).
- **2026-09-28** — Spec 002 implementada (T001–T093) en la rama `002-productos-inventario` de API y frontend: 276 pruebas de API (SQLite y PostgreSQL, con concurrencia real y consistencia aleatoria) y 70 de frontend en verde; recorrido de extremo a extremo 16/16 y pantallas a 360 px revisadas. Aclaraciones A-12 a A-16 y A-26 a A-28. Requisito de plataforma `ext-bcmath`. Pendiente: cronometrar la entrada rápida (CE-003) y el merge (T094).
- **2026-09-28** — Spec 001 integrada en `main` de la API y del frontend (merge local, T124). Queda pendiente la revisión visual de las pantallas con sesión (T122). `pint.json` excluye `legacy/` y `lang/`.
- **2026-09-28** — Cuentas de prueba en `DevelopmentSeeder` (solo `APP_ENV=local`); el usuario aprobó los cambios de la 001.
- **2026-09-28** — Spec 001 implementada (T001–T123) en la rama `001-empresa-usuarios-aislamiento` de API y frontend: 182 pruebas de API (SQLite y PostgreSQL) y 46 de frontend en verde; recorrido de extremo a extremo 17/17 (`verificacion.md`). La verificación encontró y corrigió un 500 exclusivo de PostgreSQL y un límite de rutas públicas demasiado estricto (ahora 30/min e IP). Pendiente: revisión visual de las pantallas con sesión (T122) y merge (T124). Constitución 1.1.2.
- **2026-09-28** — Tareas 001 aprobadas: la spec 001 pasa a *Lista* y empieza la implementación.
- **2026-09-26** — Plan 001 aprobado (entrega de HU-1/HU-7 sin interfaz, git en el frontend); constitución 1.1.1 (principio IV: flujos de autenticación públicos); `tasks.md` de 001 redactado (66 tareas), pendiente de aprobación.
- **2026-09-26** — Spec 001 aprobada (v1.0) tras resolver A-01 a A-11 y A-25; constitución 1.1.0 (principio VIII, aislamiento y auditoría); `plan.md` de 001 redactado, pendiente de aprobación.
- **2026-09-26** — Reorganizada la documentación SDD: la idea del SaaS pasa a `docs/producto/vision.md`; evaluación de la API en `docs/investigacion/`; 26 preguntas en `docs/aclaraciones.md`; specs 001–004 reescritas y 005 creada; se retiraron los `plan.md`/`tasks.md` prematuros; plantillas en `docs/plantillas/`. Constitución 1.0.1.
- **2026-09-23** — Adoptado SDD: creada `docs/constitution.md` v1.0.0 (7 principios; 8 desde 1.1.0).
- **2026-09-23** — Frontend migrado a pnpm 11.1.2: eliminados `node_modules`, `package-lock.json` y `dist`; añadidos `packageManager`, `pnpm-workspace.yaml` con protecciones de supply-chain, `.npmrc` (`engine-strict`) y lockfiles ajenos en `.gitignore`.
- **2026-09-23** — Creado el proyecto frontend (Vue 3 + Vite) con arquitectura por features; `features/` aún vacío.
