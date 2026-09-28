# Plan 001: Empresa, usuarios, roles y aislamiento multiempresa

**Estado:** Aprobado · **Fecha:** 2026-09-26 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

Se reinicia el dominio de la API ([A-01](../../aclaraciones.md#a-01)) en el mismo repositorio, en una rama propia:

1. El código heredado sale del autoload a `legacy/`, como referencia para la spec 005, y se parte de un esquema de base de datos limpio ([A-02](../../aclaraciones.md#a-02)).
2. El aislamiento se implementa con `company_id` y un **global scope que falla cerrado**: consultar un modelo de empresa sin contexto de empresa lanza una excepción en lugar de devolver todo.
3. Los roles son un *enum* fijo con *Policies* nativas de Laravel. No se añaden dependencias.
4. En el frontend se crean `core/auth` (sesión y guards) y las features `auth`, `company`, `users` y `audit`, sobre una capa mínima de componentes en `shared/ui`.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | API (Laravel 12.28, PHP 8.5 local) y frontend (Vue 3) |
| Dependencias nuevas | **Ninguna** en Composer ni en pnpm |
| Base de datos | PostgreSQL en desarrollo (`db_api_sunat`); SQLite en memoria para las pruebas (ya configurado en `phpunit.xml`) |
| Autenticación | Sanctum, *personal access tokens*, caducidad de 24 h (`SANCTUM_EXPIRATION=1440`, ya configurado) |
| Correo | `MAIL_MAILER=log` en desarrollo y `array` en pruebas. El SMTP real se configura antes de producción |
| Rama de trabajo | `001-empresa-usuarios-aislamiento` en el repositorio de la API; el frontend aún no tiene git (ver [Pendientes](#pendientes-antes-de-implementar)) |

## Chequeo de constitución (v1.1.0)

| Principio | Cumple | Cómo / justificación de la excepción |
|---|---|---|
| I. Spec primero | ✅ | La spec 1.0 está aprobada; este plan no añade comportamiento que no esté en ella. |
| II. Corrección fiscal | ✅ | No hay cálculos fiscales. La validación del RUC (dígito verificador) se hace en el servidor y tiene pruebas unitarias. |
| III. Contrato de API | ⚠️ | Se regenera OpenAPI y `pnpm api:types`. **Excepción heredada:** los *response bodies* siguen sin documentarse en OpenAPI. Mitigación: los tipos de respuesta se escriben a mano en cada `features/*/api/types.ts` y cada `JsonResource` tiene una prueba de estructura en la API que detecta si cambia. |
| IV. Seguridad | ✅ | Se eliminan los endpoints `setup` públicos y se restringe CORS. **Enmienda PATCH 1.1.1 aprobada:** la excepción "login e inicialización única" pasa a "flujos de autenticación con token de un solo uso y límite de intentos: login, aceptación de invitación y recuperación de contraseña". La inicialización deja de ser un endpoint y pasa a un comando de consola. |
| V. Simplicidad | ✅ | Roles como *enum* y no como tablas; sin paquetes de permisos ni de *multi-tenancy*; controladores delgados con la lógica en `app/Services`. |
| VI. Verificable | ✅ | Cada escenario tiene su prueba (ver [Estrategia de pruebas](#estrategia-de-pruebas)); la suite heredada se retira con el código que probaba. |
| VII. Personas reales | ✅ | Vistas *mobile-first*, formularios con etiquetas, re-login que vuelve a la pantalla de origen. |
| VIII. Aislamiento y auditoría | ✅ | Global scope que falla cerrado, *route model binding* con scope (404 para otra empresa), prueba de arquitectura que obliga a cubrir cada endpoint con acceso cruzado, auditoría inmutable y saneada. |

## Estrategia sobre el código heredado

| Qué | Acción |
|---|---|
| Cambios sin commitear en la API | Se commitean tal cual en `main` como "estado previo" **antes** de crear la rama (con confirmación del usuario). Nada se descarta. |
| `app/Http/Controllers/Api/*` (salvo Ubigeo), `app/Services/*`, `app/Models/*` (salvo `Ubi*`), `app/Http/Requests/*` (salvo `UbigeoSearchRequest`), `app/Traits`, `app/Http/Controllers/Traits`, comandos de consola heredados, `routes/api_consulta_mejorada.php` | Se mueven con `git mv` a `legacy/`, fuera del autoload. `GreenterService`, `PdfService` y sus plantillas se reincorporan en la 005. |
| Migraciones heredadas (salvo las de ubigeo y las base de Laravel) | Se mueven a `legacy/database/migrations`. Las tablas nuevas tienen migraciones propias. |
| `tests/Feature/*` heredados | Se mueven a `legacy/tests`; prueban funciones que ya no están enrutadas. |
| `resources/views/pdf/*` | Se quedan (las usará la 005); nadie las invoca en 001. |
| `UbigeoController`, modelos `Ubi*`, sus migraciones y seeders | Se conservan; se revisan y reciben pruebas. |
| `package.json` y `vite.config.js` de la API, `@vite` en `welcome.blade.php`, script `composer dev` con `npx`/`npm` | Se eliminan: la API no necesita toolchain JavaScript en 001, y así se cumple "solo pnpm" sin mantener dos. `welcome` pasa a una vista estática. |
| `GenerateOpenApiSpec` y la vista `/docs` | Se conservan. |
| Base de datos local | `php artisan migrate:fresh --seed` (A-02: solo hay datos de prueba). **Borra la base local**; se confirma con el usuario al ejecutarlo. |

## Modelo de datos

Todas las tablas nuevas usan `id` autoincremental y `timestamps`, salvo que se indique otra cosa.

| Tabla | Columnas relevantes | Restricciones |
|---|---|---|
| `users` (se amplía la base de Laravel) | `name`, `email`, `password`, `is_platform_admin` (bool), `active` (bool), `last_login_at` | `email` único, guardado en minúsculas y sin espacios |
| `companies` | `ruc` (11), `razon_social`, `nombre_comercial` null, `person_type` (`natural`/`juridica`), `tax_regime` (`nrus`/`rer`/`rmt`/`general`), `email`, `phone` null, `logo_path` null, `active` | `ruc` único |
| `establishments` | `company_id`, `code` (4, `0000`), `name`, `address`, `ubigeo` → `ubi_distritos`, `is_main` | único (`company_id`, `code`) |
| `company_user` (pertenencia) | `company_id`, `user_id`, `role` (`company_admin`/`seller`), `active` | único (`user_id`): **una empresa por usuario** en el MVP ([A-04](../../aclaraciones.md#a-04)). Quitar este índice abre el multiempresa sin migrar datos. |
| `invitations` | `company_id`, `email`, `role`, `token_hash`, `expires_at` (72 h), `accepted_at` null, `invited_by` | Solo se guarda el hash (SHA-256) del token |
| `audit_logs` | `company_id` null, `actor_id` null, `action`, `auditable_type`, `auditable_id`, `changes` (json saneado), `ip`, `user_agent`, `created_at` | Sin `updated_at`; el modelo impide `update` y `delete` |
| `password_reset_tokens` | Tabla base de Laravel | — |
| `personal_access_tokens` | Tabla de Sanctum | — |

- El **domicilio fiscal** vive en el establecimiento principal. La empresa lo expone como atributo derivado.
- Los roles son el *enum* `App\Enums\CompanyRole`. El administrador de la plataforma es `users.is_platform_admin` y no tiene fila en `company_user` (RF-011).

## Aislamiento (principio VIII)

| Pieza | Responsabilidad |
|---|---|
| `App\Tenancy\TenantContext` (singleton por petición) | Guarda la empresa actual. Lo fija el middleware; en consola y colas se fija explícitamente. |
| Middleware `ResolveTenant` | Tras `auth:sanctum`: carga la pertenencia activa del usuario y fija el contexto. Si la pertenencia o la empresa están inactivas, responde 401 y revoca el token actual (RF-013, CE-004). |
| Trait `BelongsToCompany` + `CompanyScope` | Filtra por `company_id` del contexto y lo rellena al crear. **Sin contexto lanza `TenantContextMissing`**; solo `Model::withoutTenancy()` lo evita, y es de uso explícito y revisable. Al crear o actualizar, rechaza un `company_id` distinto del contexto. |
| *Route model binding* | Como el binding pasa por el scope, un recurso de otra empresa da `404` idéntico al de un recurso inexistente (RF-032). |
| Prueba de arquitectura `TenantRoutesCoverageTest` | Recorre las rutas del grupo de empresa y falla si alguna no aparece en el *dataset* de `CrossTenantAccessTest` (RF-033, CE-002). Añadir una ruta sin su prueba de acceso cruzado rompe la suite. |

Son modelos de empresa: `Establishment`, `Invitation`, `AuditLog` (con contexto) y la pertenencia. `Company` no lo es: el usuario de empresa accede a la suya vía `/company`, sin identificador en la URL.

## Autorización

- *Enum* `CompanyRole` con `company_admin` y `seller`. Middleware `role:company_admin`, más *Policies* donde hay reglas por registro (p. ej. el último administrador).
- Middleware `platform.admin` para `/platform/*`. El administrador de la plataforma **no** pasa por `ResolveTenant` y no puede llamar rutas de empresa (caso límite de la spec).
- Los `FormRequest::authorize()` delegan en la *Policy*; queda prohibido `return true` sin condición. Una prueba de arquitectura lo verifica.

## Seguridad

- **Login:** `RateLimiter` de 5 intentos por (correo + IP) cada 15 minutos (RF-012). Cada fallo queda auditado. El mensaje es el mismo si el usuario no existe.
- **Invitación y recuperación:** token aleatorio de 64 caracteres; solo se guarda el hash; un solo uso; 72 h para invitaciones y 60 min para recuperación (valor por defecto de Laravel). Al recuperar la contraseña se revocan todos los tokens del usuario (RF-016).
- **Desactivación** de un usuario o una empresa: se revocan sus tokens en el acto, y además `ResolveTenant` lo comprueba en cada petición.
- **Saneado de auditoría:** `AuditLogger` elimina claves que contengan `password`, `token`, `secret`, `clave`, `certificado` y todos los atributos `$hidden` del modelo (RF-041).
- **Logs:** ningún `Log::` recibe `getChanges()` ni `$request->all()`. Una prueba busca contraseñas conocidas en el log, las respuestas y la auditoría tras un recorrido completo (CE-003).
- **CORS:** se publica `config/cors.php` con `allowed_origins` desde `CORS_ALLOWED_ORIGINS` (RF-051).
- **Primer administrador de plataforma:** comando `php artisan platform:create-admin` (interactivo). No existe endpoint de inicialización (RF-050).
- **Errores JSON:** en `bootstrap/app.php`, todas las rutas `api/*` responden con `{ success: false, message, errors? }` para 401, 403, 404, 422 y 429, sin trazas fuera de `local`.

## Contrato de API

Prefijo `/api/v1`. Respuestas con el sobre `{ success, data, meta? }`. Cambian las rutas: las heredadas desaparecen.

**Públicas, con límite de intentos**

| Método | Ruta | Descripción | Spec |
|---|---|---|---|
| POST | `/auth/login` | Devuelve token, usuario, empresa y rol | HU-2.3/4 |
| POST | `/auth/forgot-password` | Envía el enlace; respuesta idéntica exista o no el correo | HU-2.7 |
| POST | `/auth/reset-password` | Nueva contraseña con token; revoca las sesiones | HU-2.7/8 |
| GET | `/invitations/{token}` | Datos para aceptar: empresa, correo, rol, vigencia | HU-2.1/2 |
| POST | `/invitations/{token}/accept` | Define nombre y contraseña; devuelve token de sesión | HU-2.1 |

**Usuario autenticado de empresa** (`auth:sanctum` + `ResolveTenant`)

| Método | Ruta | Rol | Descripción | Spec |
|---|---|---|---|---|
| POST | `/auth/logout` | Todos | Revoca el token actual | HU-2 |
| GET | `/auth/me` | Todos | Usuario, empresa y rol | HU-2.3 |
| GET | `/company` | Todos | Datos de la propia empresa | HU-5 |
| PATCH | `/company` | Admin | Nombre comercial, correo y teléfono; el RUC no se admite | HU-5.1/2 |
| POST | `/company/logo` | Admin | Sube el logo (PNG/JPG ≤ 1 MB) | HU-5.1 |
| GET | `/users` | Admin | Usuarios de la empresa | HU-3 |
| PATCH | `/users/{user}` | Admin | Cambia rol o estado; protege al último administrador | HU-3.3 |
| GET | `/invitations` | Admin | Invitaciones pendientes | HU-3 |
| POST | `/invitations` | Admin | Invita (correo + rol) | HU-3.1/4 |
| POST | `/invitations/{invitation}/resend` | Admin | Nuevo token y nueva vigencia | HU-2.2 |
| DELETE | `/invitations/{invitation}` | Admin | Cancela una invitación pendiente | HU-3 |
| GET | `/audit-logs` | Admin | Auditoría paginada, con filtros por acción, actor y fechas | HU-6 |
| GET | `/ubigeos/...` | Todos | Se conserva el controlador actual | HU-1 |

**Administrador de plataforma** (`auth:sanctum` + `platform.admin`)

| Método | Ruta | Descripción | Spec |
|---|---|---|---|
| GET | `/platform/companies` | Lista con estado; nunca incluye secretos | HU-4.4, HU-7 |
| POST | `/platform/companies` | Crea empresa y establecimiento principal, e invita a su administrador (una transacción) | HU-1 |
| GET | `/platform/companies/{company}` | Detalle | HU-7 |
| PATCH | `/platform/companies/{company}` | Edita, incluida la corrección del RUC | HU-5.2 |
| POST | `/platform/companies/{company}/deactivate` · `/activate` | Suspende o reactiva la empresa y revoca los tokens | HU-7 |

> **Decisión aprobada (2026-09-26):** HU-1 y HU-7 se entregan en 001 como **endpoints probados más el comando `php artisan company:create`**, pero **sin interfaz gráfica**. La interfaz de la plataforma es la spec 006 (Blade o Livewire, según A-08). Con 5 empresas iniciales, el comando cumple CE-001 (menos de 3 minutos).

## Frontend

Estructura (respeta las reglas del README: ninguna feature importa de otra):

```
src/
├── core/auth/                 # Transversal: identidad y sesión
│   ├── session-store.ts       # Pinia: usuario, empresa, rol; login/logout/me
│   ├── guards.ts              # requiresAuth, guestOnly, meta.roles
│   └── __tests__/
├── app/layouts/
│   ├── AuthLayout.vue         # Pantallas públicas
│   └── AppLayout.vue          # Navegación móvil/escritorio, empresa y rol visibles
├── features/
│   ├── auth/                  # Login, ¿olvidaste tu contraseña?, restablecer, aceptar invitación
│   ├── company/               # Datos de mi empresa y logo (HU-5)
│   ├── users/                 # Usuarios, invitar, cambiar rol, desactivar (HU-3)
│   └── audit/                 # Auditoría con filtros (HU-6)
└── shared/ui/                 # BaseButton, BaseInput, FormField, BaseAlert, BaseDialog, BaseBadge, EmptyState
```

- **Re-login (RF-014, HU-2.5):** ante un 401, el guard lleva a `/login?redirect=<ruta actual>` y, tras entrar, vuelve a esa ruta. Los formularios de 001 son cortos; si se pierde un borrador, se pierde poco. El composable `useFormDraft` se añade cuando lo necesite un formulario largo (002 en adelante).
- **Rutas por rol:** `meta.roles: ['company_admin']` oculta la navegación y bloquea la ruta. La autorización real sigue en el servidor.
- **Errores:** los `errors` de 422 se muestran junto a cada campo; los mensajes vienen en español desde la API (`laravel-lang` ya instalado).
- **Accesibilidad:** `<label>` en cada campo, foco visible, navegación por teclado en los diálogos.
- **Limpieza:** el comentario de `client.ts` que dice `npm run api:types` pasa a `pnpm api:types`.

## Estrategia de pruebas

**API (Pest).** Todas deben estar en verde con `php artisan test`.

| Escenario / requisito | Prueba |
|---|---|
| HU-1.1–1.4, RF-001–004 | `Platform/CreateCompanyTest` |
| RUC: formato, dígito verificador, prefijos | `Unit/Rules/RucTest` (casos válidos e inválidos) |
| HU-2.1/2.2, RF-010 | `Auth/InvitationAcceptanceTest` |
| HU-2.3/2.4, RF-012 | `Auth/LoginTest`, incluido el bloqueo tras 5 intentos |
| HU-2.6, RF-013, CE-004 | `Auth/InactiveAccountTest` (usuario o empresa desactivados con sesión abierta) |
| HU-2.7/2.8, RF-016 | `Auth/PasswordResetTest` |
| HU-3.1–3.4, RF-015 | `Users/InviteUserTest`, `Users/ManageUsersTest` |
| HU-4.1–4.3, RF-030–033, CE-002 | `Tenancy/CrossTenantAccessTest` (*dataset* con todas las rutas de empresa) y `Tenancy/TenantRoutesCoverageTest` |
| Scope que falla cerrado | `Tenancy/CompanyScopeTest` (sin contexto lanza excepción; `company_id` ajeno se rechaza) |
| HU-4.4, HU-7 | `Platform/ManageCompaniesTest` |
| HU-5.1/5.2 | `Company/UpdateCompanyTest` |
| HU-6.1–6.3, RF-040/041 | `Audit/AuditLogTest` (registro, inmutabilidad, saneado) |
| CE-003 | `Security/NoSecretsLeakTest` (recorrido completo buscando contraseñas en log, respuestas y auditoría) |
| RF-050/051 | `Security/NoPublicSetupRoutesTest`, `Security/CorsTest` |
| Estructura de las respuestas | `Contract/*ResourceTest` (una por `JsonResource`) |
| `authorize()` sin `return true` | `Architecture/FormRequestsAuthorizeTest` (arch testing de Pest) |

**Frontend (Vitest).** `pnpm type-check`, `pnpm lint`, `pnpm test:unit --run` y `pnpm build-only` en verde.

| Qué | Prueba |
|---|---|
| Sesión: login, logout, `me`, limpieza ante 401 | `core/auth/__tests__/session-store.spec.ts` |
| Guards: invitado, autenticado, rol, `redirect` | `core/auth/__tests__/guards.spec.ts` |
| Formularios de login, invitación y restablecimiento (validación y errores 422) | `features/auth/__tests__/*.spec.ts` |
| Usuarios: listar, invitar, proteger al último administrador (mensaje de la API) | `features/users/__tests__/*.spec.ts` |

**Verificación manual documentada** al cierre: recorrido completo en el navegador a 360 px de ancho (crear empresa por comando → aceptar invitación → invitar vendedor → vendedor sin acceso a usuarios → desactivar vendedor → auditoría).

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Una consulta con `withoutTenancy()` o SQL crudo filtra datos | `withoutTenancy()` solo en código de plataforma, fácil de buscar en revisión; SQL crudo prohibido en modelos de empresa; la prueba de cobertura de rutas obliga a probar cada endpoint. |
| Las pruebas en SQLite no detectan diferencias con PostgreSQL | En 001 no hay concurrencia crítica (la unicidad del RUC la garantiza un índice). Desde la 002 (stock) se añade una ejecución de la suite contra PostgreSQL. |
| Mover el código heredado rompe algo que se usaba | A-02: no hay datos ni usuarios reales. Todo queda en `legacy/` con historial git. |
| Los tipos de respuesta escritos a mano se desalinean de la API | Pruebas de contrato por `JsonResource`; revisar al generar tipos. Mejorar el generador de OpenAPI queda como deuda del principio III. |
| Correo no configurado en producción | Fuera del alcance del MVP en beta; el enlace de invitación también se muestra en la salida de `company:create`. |

## Aprobación

Aprobado por el usuario el 2026-09-26, junto con:

1. Entregar HU-1 y HU-7 como endpoints y comando de consola, sin interfaz gráfica (la interfaz llega en la 006).
2. La enmienda PATCH 1.1.1 del principio IV.
3. Inicializar git en el frontend (tarea T002).

El commit del "estado previo" de la API y el `migrate:fresh` de la base local se confirman al ejecutar sus tareas (T001 y T013).
