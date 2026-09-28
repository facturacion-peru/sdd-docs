# Tareas 001: Empresa, usuarios, roles y aislamiento multiempresa

**Estado:** Aprobado (2026-09-28) · **Fecha:** 2026-09-26 · **Plan:** [plan.md](./plan.md) (aprobado)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación. Se marcan `[x]` al terminar.
> `[P]` = puede hacerse en paralelo con la tarea anterior. 🔒 = requiere confirmación explícita del usuario al ejecutarla.
> **Pruebas primero:** las tareas "Escribir pruebas" deben terminar con las pruebas **en rojo**. La tarea siguiente las pone en verde.

**Convenciones**

- API: `cd Api-de-facturacion-electronica-sunat-Peru`. Frontend: `cd frontend-api-facturacion-electronica-sunat`.
- `pint` = `vendor/bin/pint --test`. `test` = `php artisan test`.
- Un commit por tarea (o por pareja prueba + implementación), en español, en la rama `001-empresa-usuarios-aislamiento`.

---

## Fase 1 · Preparación

- [x] **T001** 🔒 Commitear en `main` de la API los cambios sin commitear tal como están, con el mensaje "Estado previo a la spec 001". — Verificación: `git status` limpio; `git log -1` muestra el commit.
- [x] **T002** 🔒 Inicializar git en el frontend y hacer el commit inicial (el `.gitignore` ya excluye `node_modules`, `dist` y `.env`). — Verificación: `git log --oneline` muestra 1 commit; `git status` limpio; `.env` no está versionado.
- [x] **T003** Crear la rama `001-empresa-usuarios-aislamiento` en ambos repositorios. — Verificación: `git branch --show-current` en los dos.
- [x] **T004** Mover el código heredado a `legacy/` con `git mv`, según la tabla *Estrategia sobre el código heredado* del plan: controladores (salvo Ubigeo), servicios, modelos (salvo `Ubi*`), requests (salvo `UbigeoSearchRequest`), traits, comandos heredados y migraciones heredadas. `legacy/` queda fuera del autoload. — Verificación: `composer dump-autoload` sin avisos; `php artisan list` funciona; `app/Services` vacío.
- [x] **T005** Reescribir `routes/api.php` dejando solo `/v1/ubigeos/*` bajo `auth:sanctum`, y eliminar `routes/api_consulta_mejorada.php`. — Verificación: `php artisan route:list --path=api` muestra solo ubigeos; ya no existe `api/setup/*`.
- [x] **T006** [P] Eliminar la toolchain JavaScript de la API: `package.json`, `vite.config.js`, `@vite` en `welcome.blade.php` (pasa a vista estática) y el script `composer dev` con `npx`/`npm`. — Verificación: `grep -rn "npx\|npm " composer.json` vacío; `GET /` responde 200 con `php artisan serve`.
- [x] **T007** [P] Mover `tests/Feature/*` heredados a `legacy/tests` y dejar `ExampleTest`. — Verificación: `test` en verde (solo pruebas de ejemplo).

## Fase 2 · Fundamentos (bloquean todas las historias)

- [x] **T010** Escribir pruebas `Api/ErrorResponsesTest`: 401, 403, 404, 422 y 429 en `api/*` responden `{ success: false, message, errors? }` sin trazas. — Verificación: en rojo.
- [x] **T011** Configurar el render de excepciones JSON en `bootstrap/app.php`. — Verificación: T010 en verde.
- [x] **T012** Crear migraciones: ampliación de `users` (`is_platform_admin`, `active`, `last_login_at`), `companies`, `establishments`, `company_user` (único `user_id`), `invitations` y `audit_logs`, según el *Modelo de datos* del plan. — Verificación: `test` en verde (migra en SQLite); `php artisan migrate --pretend` sin errores contra PostgreSQL.
- [x] **T013** 🔒 `php artisan migrate:fresh --seed` en la base local `db_api_sunat` (**borra sus datos de prueba**, A-02). El seeder solo carga ubigeos. — Verificación: `php artisan migrate:status` muestra solo las migraciones nuevas, las base de Laravel y las de ubigeo.
- [x] **T014** Crear los *enums* `CompanyRole`, `PersonType` y `TaxRegime`, los modelos `User`, `Company`, `Establishment`, `CompanyMembership`, `Invitation` y `AuditLog`, y sus *factories*. — Verificación: `php artisan tinker --execute="App\Models\Company::factory()->make()"` sin errores; `pint`.
- [x] **T015** Escribir pruebas `Tenancy/CompanyScopeTest`: sin contexto lanza `TenantContextMissing`; con contexto filtra; al crear se rellena `company_id`; un `company_id` ajeno se rechaza; `withoutTenancy()` es la única salida. — Verificación: en rojo.
- [x] **T016** Implementar `TenantContext`, `CompanyScope`, `BelongsToCompany` y `TenantContextMissing`, y aplicarlos a `Establishment`, `CompanyMembership`, `Invitation` y `AuditLog`. — Verificación: T015 en verde.
- [x] **T017** Escribir pruebas `Tenancy/ResolveTenantTest` y `Auth/RoleMiddlewareTest`: fija el contexto según la pertenencia; 401 y revocación del token si el usuario, la pertenencia o la empresa están inactivos; `role:company_admin` rechaza al vendedor; `platform.admin` rechaza a usuarios de empresa, y el administrador de la plataforma no entra en rutas de empresa. — Verificación: en rojo.
- [x] **T018** Implementar los middlewares `ResolveTenant`, `role` y `platform.admin`, y los grupos de rutas vacíos (público, empresa, plataforma) en `routes/api.php`. — Verificación: T017 en verde.
- [x] **T019** Escribir pruebas `Audit/AuditLoggerTest`: registra actor, empresa, acción, registro, IP y agente; `update` y `delete` del modelo lanzan excepción; elimina claves sensibles y atributos `$hidden`. — Verificación: en rojo.
- [x] **T020** Implementar `AuditLogger` y la inmutabilidad de `AuditLog`. — Verificación: T019 en verde.
- [x] **T021** [P] Escribir pruebas `Unit/Rules/RucTest`: 11 dígitos, prefijos `10` y `20` admitidos (otros rechazados con mensaje), dígito verificador con pesos `5,4,3,2,7,6,5,4,3,2` y RUC reales válidos e inválidos. — Verificación: en rojo.
- [x] **T022** Implementar la regla `Ruc` y el helper que deduce `PersonType` del prefijo. — Verificación: T021 en verde.
- [x] **T023** [P] Escribir `Architecture/FormRequestsAuthorizeTest` (arch testing de Pest): ningún `FormRequest` en `app/` devuelve `true` sin condición; ningún controlador usa `Log::` con `getChanges()` ni `$request->all()`. — Verificación: en verde (aún no hay requests; debe fallar si se añade uno que incumpla, lo que se comprueba con un caso temporal).
- [x] **T024** Comando `platform:create-admin` (nombre, correo y contraseña interactivos, contraseña con confirmación) y su prueba. — Verificación: prueba en verde; ejecutarlo en local crea el administrador.

## Fase 3 · HU-1 Alta de empresa (P1)

- [x] **T030** Escribir pruebas `Platform/CreateCompanyTest`: HU-1.1 (empresa, establecimiento `0000` e invitación en una transacción; correo enviado), HU-1.2 (RUC duplicado), HU-1.3 (RUC inválido), HU-1.4 (tipo de persona deducido); campos obligatorios; solo el administrador de la plataforma. — Verificación: en rojo.
- [x] **T031** Implementar `CompanyService::create`, `Platform\CompanyController@store`, `StoreCompanyRequest`, `CompanyResource`, la notificación `InvitationNotification` (enlace a `FRONTEND_URL/invitacion/{token}`) y la auditoría del alta. Añadir `FRONTEND_URL` a `.env.example`. — Verificación: T030 en verde; `pint`.
- [x] **T032** Comando `company:create` que usa `CompanyService` y muestra el enlace de invitación en consola, con su prueba. — Verificación: prueba en verde; ejecutarlo en local crea una empresa en menos de 3 minutos (CE-001).
- [x] **T033** [P] `Contract/CompanyResourceTest`: estructura exacta de la respuesta, sin campos no declarados. — Verificación: en verde.

## Fase 4 · HU-2 Acceso de los usuarios (P1)

- [x] **T040** Escribir pruebas `Auth/InvitationAcceptanceTest`: HU-2.1 (ver datos, aceptar, obtener token, pertenencia creada con el rol invitado), HU-2.2 (vencida, usada o inexistente); límite de intentos; auditoría. — Verificación: en rojo.
- [x] **T041** Implementar `GET /invitations/{token}` y `POST /invitations/{token}/accept` (`InvitationService`, requests y resource). — Verificación: T040 en verde.
- [x] **T042** Escribir pruebas `Auth/LoginTest`: HU-2.3 (token, usuario, empresa y rol), el mismo mensaje para usuario inexistente o contraseña errónea, HU-2.4 (bloqueo tras 5 intentos en 15 min), logout revoca el token, `me`, auditoría de fallos, `last_login_at`. — Verificación: en rojo.
- [x] **T043** Implementar `POST /auth/login`, `POST /auth/logout` y `GET /auth/me` con `RateLimiter`, y `UserResource`/`SessionResource`. — Verificación: T042 en verde.
- [x] **T044** Escribir pruebas `Auth/InactiveAccountTest`: HU-2.6 y CE-004 (usuario, pertenencia o empresa desactivados con sesión abierta → la siguiente petición da 401; el login se rechaza). — Verificación: en verde si T018 lo cubre; si no, en rojo y se corrige en esta misma tarea.
- [x] **T045** Escribir pruebas `Auth/PasswordResetTest`: HU-2.7 (respuesta idéntica exista o no el correo; enlace de un solo uso a `FRONTEND_URL/restablecer`), HU-2.8 (revoca todos los tokens), token vencido, auditoría. — Verificación: en rojo.
- [x] **T046** Implementar `POST /auth/forgot-password` y `POST /auth/reset-password` con el *password broker* de Laravel y una notificación con la URL del frontend. — Verificación: T045 en verde.
- [x] **T047** [P] `Contract/SessionResourceTest` y `Contract/UserResourceTest`. — Verificación: en verde.

## Fase 5 · HU-3 Gestión de usuarios (P1)

- [x] **T050** Escribir pruebas `Users/InviteUserTest`: HU-3.1 (invitar vendedor o administrador), HU-3.4 (correo de otra empresa rechazado sin revelar cuál), correo normalizado (mayúsculas y espacios), reenviar (nuevo token, invalida el anterior), cancelar, vendedor sin acceso (HU-3.2). — Verificación: en rojo.
- [x] **T051** Implementar `GET/POST /invitations`, `POST /invitations/{invitation}/resend` y `DELETE /invitations/{invitation}`, con auditoría. — Verificación: T050 en verde.
- [x] **T052** Escribir pruebas `Users/ManageUsersTest`: listar; cambiar rol; desactivar (revoca tokens); HU-3.3 y RF-015 (no dejar la empresa sin administrador activo, ni por desactivación ni por cambio de rol); vendedor sin acceso; auditoría. — Verificación: en rojo.
- [x] **T053** Implementar `GET /users` y `PATCH /users/{user}` con `UserPolicy` y `UserService`. — Verificación: T052 en verde.
- [x] **T054** [P] `Contract/InvitationResourceTest` y `Contract/CompanyUserResourceTest`. — Verificación: en verde.

## Fase 6 · HU-5 Datos de la propia empresa (P2)

- [x] **T060** Escribir pruebas `Company/UpdateCompanyTest`: `GET /company` para cualquier rol; HU-5.1 (nombre comercial, correo y teléfono, auditado); HU-5.2 (el RUC se ignora o rechaza); logo PNG/JPG ≤ 1 MB, otros formatos rechazados; vendedor no edita. — Verificación: en rojo.
- [x] **T061** Implementar `GET /company`, `PATCH /company` y `POST /company/logo` (logo en `storage/app/public/companies/{id}/`). — Verificación: T060 en verde.

## Fase 7 · HU-7 Gestión de empresas por la plataforma (P2)

- [x] **T070** Escribir pruebas `Platform/ManageCompaniesTest`: listar y ver (HU-4.4: sin secretos); editar, incluida la corrección del RUC con validación; HU-7.1 (desactivar revoca los tokens de todos sus usuarios y conserva los datos); HU-7.2 (reactivar); auditoría; usuarios de empresa rechazados. — Verificación: en rojo.
- [x] **T071** Implementar `GET /platform/companies`, `GET/PATCH /platform/companies/{company}` y `POST .../deactivate` y `.../activate`. — Verificación: T070 en verde.

## Fase 8 · HU-6 Auditoría (P2)

- [x] **T080** Escribir pruebas `Audit/AuditLogTest`: HU-6.1 (listado paginado con filtros por acción, actor y fechas; solo de la propia empresa); HU-6.2 (no hay rutas para modificar ni borrar); HU-6.3; y una comprobación de que **cada acción de RF-040** genera su registro. — Verificación: en rojo.
- [x] **T081** Implementar `GET /audit-logs` y `AuditLogResource`, y completar las acciones de RF-040 que falten. — Verificación: T080 en verde.

## Fase 9 · HU-4 Aislamiento y seguridad transversal (P1)

- [x] **T090** Escribir `Tenancy/CrossTenantAccessTest`: un *dataset* con **todas** las rutas del grupo de empresa; para cada una, un usuario de A contra recursos de B → 404 idéntico al inexistente (HU-4.1), listados sin datos de B aunque se envíen parámetros de B (HU-4.2), creación o modificación apuntando a B rechazada (HU-4.3). — Verificación: en verde (si alguna falla, se corrige en esta tarea).
- [x] **T091** Escribir `Tenancy/TenantRoutesCoverageTest`: falla si una ruta del grupo de empresa no está en el *dataset* de T090. — Verificación: en verde; añadir una ruta temporal sin cubrir la hace fallar.
- [x] **T092** Escribir `Security/NoSecretsLeakTest` (CE-003): recorrido completo (alta, invitación, login, recuperación, cambios de usuario) con contraseñas conocidas y búsqueda de esas cadenas en el log, en todas las respuestas y en `audit_logs`. — Verificación: en verde.
- [x] **T093** [P] Escribir `Security/NoPublicSetupRoutesTest` (RF-050: toda ruta pública está en la lista blanca de flujos de autenticación) y `Security/CorsTest` (RF-051). — Verificación: CorsTest en rojo.
- [x] **T094** Publicar `config/cors.php` con `allowed_origins` desde `CORS_ALLOWED_ORIGINS`, y añadir la variable a `.env.example`. — Verificación: T093 en verde.
- [x] **T095** Regenerar el contrato: `php artisan openapi:generate`. — Verificación: `public/openapi.json` contiene las 24 rutas del plan y ninguna heredada; `/docs` carga.

## Fase 10 · Frontend: base

- [x] **T100** `pnpm api:types` y corregir en `client.ts` el comentario de `npm run api:types` a `pnpm api:types`. — Verificación: `pnpm type-check` en verde.
- [x] **T101** [P] Componentes `shared/ui`: `BaseButton`, `BaseInput`, `FormField` (label, error, ayuda), `BaseAlert`, `BaseDialog` (foco atrapado, Escape), `BaseBadge` y `EmptyState`, *mobile-first* con Tailwind. — Verificación: `pnpm type-check` y `pnpm lint`; prueba de `BaseDialog` (foco y Escape) en verde.
- [x] **T102** Escribir pruebas `core/auth/__tests__/session-store.spec.ts`: login guarda el token y la sesión; logout limpia; `me` restaura la sesión al recargar; un 401 limpia la sesión. — Verificación: en rojo.
- [x] **T103** Implementar `core/auth/session-store.ts` y los tipos de respuesta en `core/auth/types.ts`. — Verificación: T102 en verde.
- [x] **T104** Escribir pruebas `core/auth/__tests__/guards.spec.ts`: `requiresAuth` lleva a `/login?redirect=…`; `guestOnly` lleva al inicio si hay sesión; `meta.roles` bloquea al vendedor. — Verificación: en rojo.
- [x] **T105** Implementar `core/auth/guards.ts`, conectarlos al router y crear `AuthLayout` y `AppLayout` (empresa y rol visibles, navegación según el rol, cerrar sesión). — Verificación: T104 en verde; `pnpm build-only`.

## Fase 11 · Frontend: features

- [x] **T110** `features/auth`: `LoginView`, `ForgotPasswordView`, `ResetPasswordView` y `AcceptInvitationView`, con sus rutas, llamadas a la API y pruebas (validación, errores 422 por campo, bloqueo 429, redirección tras login). — Verificación: pruebas en verde.
- [x] **T111** `features/users`: listado, diálogo de invitar, invitaciones pendientes (reenviar, cancelar), cambiar rol y desactivar, mostrando el mensaje de la API al intentar dejar la empresa sin administrador. Pruebas incluidas. — Verificación: pruebas en verde.
- [x] **T112** [P] `features/company`: ver y editar los datos de la empresa, y subir el logo con vista previa. RUC y razón social de solo lectura. — Verificación: `pnpm type-check`; prueba del formulario en verde.
- [x] **T113** [P] `features/audit`: listado paginado con filtros (acción, usuario, fechas), legible en el celular. — Verificación: `pnpm type-check`; prueba de filtros en verde.

## Cierre

- [x] **T120** API: `test` completo en verde y `pint` sin cambios. — Verificación: salida de ambos comandos adjunta al cierre.
- [x] **T121** Frontend: `pnpm type-check`, `pnpm lint`, `pnpm test:unit --run` y `pnpm build-only` en verde. — Verificación: salida adjunta.
- [ ] **T122** Verificación manual a 360 px en el navegador, documentada en `verificacion.md` en esta carpeta: `company:create` → aceptar invitación → invitar vendedor → el vendedor no ve usuarios ni auditoría → desactivar vendedor (pierde la sesión) → recuperar contraseña → revisar la auditoría. — Verificación: `verificacion.md` con resultado por paso.
  - *Avance (2026-09-28):* recorrido de extremo a extremo por HTTP 17/17 y pantallas públicas a 360 px revisadas. Falta la revisión visual de las pantallas con sesión (sección 3 de `verificacion.md`).
- [x] **T123** Documentación: README de la API (instalación, `platform:create-admin`, `company:create`, variables nuevas, endpoints) y del frontend (features y flujo de sesión); retirar de la *deuda conocida* de la constitución lo que quedó resuelto (setup público, aislamiento, CORS); bitácora de `AGENTS.md`; spec 001 → **Completada** en `specs/README.md`. — Verificación: enlaces válidos; sin referencias a rutas heredadas.
- [x] **T124** 🔒 Integrar la rama: revisión final con el usuario y merge a `main` (o PR, según prefiera). — Verificación: `main` contiene el trabajo y las suites en verde.

## Añadidas durante la implementación

- [x] **T125** Cuentas de prueba en `DevelopmentSeeder`, solo con `APP_ENV=local` (pedido del usuario, 2026-09-28). — Verificación: `tests/Feature/Seeders/DevelopmentSeederTest.php` en verde; `migrate:fresh --seed` local y login real de las cinco cuentas.

## Trazabilidad

| Spec | Tareas |
|---|---|
| HU-1, RF-001–004, CE-001 | T021–T022, T030–T033 |
| HU-2, RF-010–014, RF-016, CE-004 | T040–T047, T102–T105, T110 |
| HU-3, RF-015, RF-020–021 | T017–T018, T050–T054, T111 |
| HU-4, RF-030–033, CE-002 | T015–T018, T090–T091 |
| HU-5 | T060–T061, T112 |
| HU-6, RF-040–041 | T019–T020, T080–T081, T113 |
| HU-7, RF-005 | T070–T071 |
| RF-050–051, CE-003 | T005, T092–T094 |
| Principio III (contrato) | T033, T047, T054, T095, T100 |
