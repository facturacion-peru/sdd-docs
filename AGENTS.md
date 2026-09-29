# AGENTS.md

Notas para agentes (y personas) que trabajen en este espacio. Aquí va lo importante que **no** está en los README ni se deduce leyendo el código: decisiones, reglas y trampas conocidas. Mantenerlo breve; el detalle vive en el README de cada proyecto.

## Proyectos

| Carpeta | Qué es | Stack | README |
|---|---|---|---|
| `Api-de-facturacion-electronica-sunat-Peru/` | API REST multiempresa (empresas, usuarios, auditoría; emisión SUNAT en camino). Futuro panel de plataforma en Blade/Livewire (spec 006) | Laravel 12, PHP 8.2+, Greenter 5.1, Sanctum, Pest, PostgreSQL | [README](Api-de-facturacion-electronica-sunat-Peru/README.md) |
| `frontend-api-facturacion-electronica-sunat/` | App de todos los usuarios de empresa, incluido su administrador (A-08). Futuro APK con Capacitor | Vue 3 + TS 5.9, Vite 8, Pinia, Tailwind 4, `openapi-fetch` | [README](frontend-api-facturacion-electronica-sunat/README.md) |

Hay tres repositorios git independientes: la **raíz** (solo `docs/`, `AGENTS.md` y `CLAUDE.md`; ignora las dos carpetas de proyecto), la **API** (fork de `yorchavez9/Api-de-facturacion-electronica-sunat-Peru`) y el **frontend** (desde la tarea T002 de la spec 001).

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
Primer arranque local: `php artisan migrate:fresh --seed` (crea las cuentas de prueba). Fuera de local: `php artisan platform:create-admin` y `php artisan company:create` (interactivos).

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
- El código heredado de `legacy/` no se ejecuta: antes de reincorporar algo, leer `docs/investigacion/001-evaluacion-api-existente.md`.

## Bitácora

Entradas breves y fechadas de decisiones o cambios relevantes (la más reciente arriba).

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
