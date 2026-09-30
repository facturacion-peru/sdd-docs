# Tareas 006: Panel de administración de la plataforma

**Estado:** Aprobado (2026-09-29) · **Fecha:** 2026-09-29 · **Plan:** [plan.md](./plan.md) (aprobado)

> Mismas convenciones que las specs anteriores: tareas pequeñas con su verificación; `[P]` = en paralelo con la anterior; 🔒 = confirmación del usuario al ejecutarla.
> **Pruebas primero:** las tareas «Escribir pruebas» terminan en rojo y la siguiente las pone en verde.
> Antes de cada commit se verifica con el **código de salida** de las suites de la API (`php artisan test`, `composer test:pgsql`, `vendor/bin/pint --test`) o del frontend.

## Fase 1 · Preparación y protección

- [ ] **T001** Crear la rama `006-panel-plataforma` desde `main` en la API y en el frontend. — Verificación: rama en ambos.
- [ ] **T002** Escribir e implementar `Platform/PlatformAccessTest`: recorre todas las rutas `platform/*` del router y exige 401 sin sesión y 403 para un administrador y un vendedor de empresa. — Verificación: en verde con las rutas actuales; en rojo si se quita `platform.admin` de una ruta (mutación).

## Fase 2 · Sesión (A-38)

- [ ] **T010** Escribir pruebas `Platform/PlatformSessionTest`: el token de la plataforma vence a las 8 h (401 a las 8 h 1 min) y el de empresa sigue en 24 h; el inicio de sesión de la plataforma queda auditado con IP (`auth.login`, sin empresa). — Verificación: en rojo.
- [ ] **T011** Implementar el vencimiento de 8 h y la auditoría del inicio de sesión de la plataforma. Anotar el 2FA como requisito de producción en el README. — Verificación: T010 en verde en ambas bases.

## Fase 3 · Empresas: resumen y ficha (HU-1)

- [ ] **T020** Escribir pruebas `Platform/CompanySummaryTest` con datos en tres empresas:
  - HU-1.1: por empresa, activa, estado SUNAT efectivo, usuarios, comprobantes pendientes y rechazados.
  - HU-1.2: búsqueda por RUC o razón social; filtros `status` y `issues`.
  - HU-1.3: ficha con domicilio fiscal, administrador (nombre, correo, invitación aceptada o pendiente), usuarios activos y última actividad.
  - Verificación: en rojo.
- [ ] **T021** Implementar `PlatformCompanyResource`, `PlatformCompanyDetailResource`, el request de filtros y las consultas agregadas (`withoutTenancy`). — Verificación: T020 en verde en ambas bases.

## Fase 4 · Alta, corrección, suspensión e invitación (HU-2 a HU-4)

- [ ] **T030** [P] `GET /platform/ubigeos/search` para el panel, con prueba. — Verificación: en verde.
- [ ] **T031** Escribir pruebas `Platform/CorrectLegalDataTest`:
  - Corregir el domicilio fiscal (dirección y ubigeo del establecimiento principal), auditado.
  - Cambiar el RUC → configuración SUNAT `pending` con el motivo; los comprobantes emitidos conservan su RUC.
  - RUC de otra empresa → 422.
  - `validate()` con un certificado de otro RUC → error con el motivo.
  - Verificación: en rojo.
- [ ] **T032** Implementar la corrección en `CompanyService::update`, el `UpdateCompanyRequest` ampliado y la comprobación del RUC en `SunatConfigService::validate()`. — Verificación: T031 en verde.
- [ ] **T033** Escribir pruebas: suspensión con motivo obligatorio y auditado (ampliar `ManageCompaniesTest`), y los reintentos automáticos de una empresa suspendida siguen (`sunat:send-pending`). — Verificación: en rojo.
- [ ] **T034** Implementar `DeactivateCompanyRequest` con el motivo. — Verificación: T033 en verde.
- [ ] **T035** Escribir pruebas `Platform/ResendAdminInvitationTest`: nuevo enlace, el anterior deja de valer, 422 si el administrador ya aceptó o no hay invitación, auditoría. — Verificación: en rojo.
- [ ] **T036** Implementar `POST /platform/companies/{company}/admin-invitation/resend` con `InvitationService`. — Verificación: T035 en verde.

## Fase 5 · Soporte de la emisión y auditoría (HU-5, HU-6)

- [ ] **T040** Escribir pruebas `Platform/EmissionSupportTest`:
  - HU-5.1: pendientes y rechazados de todas las empresas, con empresa, número, estado, código y mensaje de SUNAT e intentos; sin líneas, cliente ni importes; filtro por empresa y estado.
  - HU-5.2: reintento con `FakeSunatSender` dentro del contexto de la empresa; auditoría con la plataforma como autor; 422 si el estado es definitivo; 409 si hay un envío en curso.
  - Verificación: en rojo.
- [ ] **T041** Implementar `PlatformSalesDocumentResource`, el binding sin contexto de `{document}` y las dos rutas. — Verificación: T040 en verde.
- [ ] **T042** [P] Escribir e implementar `Platform/PlatformAuditTest` y `GET /platform/audit-logs` (acciones de administradores de la plataforma e inicios de sesión de la plataforma, con filtros). — Verificación: en verde.

## Fase 6 · Privacidad y contrato

- [ ] **T050** `Security/PlatformPrivacyTest` (CE-002): con productos, clientes, tickets, comprobantes y configuración SUNAT sembrados, recorre todas las rutas GET de plataforma y busca secretos, nombres de productos y clientes, líneas e importes. — Verificación: en verde, y en rojo con una mutación que añada el total al recurso de soporte.
- [ ] **T051** [P] `Contract/PlatformResourcesTest`. — Verificación: en verde.
- [ ] **T052** `php artisan openapi:generate`. README de la API: panel de plataforma, sesión de 8 h y 2FA pendiente para producción. — Verificación: OpenAPI con las rutas nuevas.

## Fase 7 · Frontend

- [ ] **T060** `pnpm api:types`; `features/platform/types.ts` y `api.ts`. — Verificación: `pnpm type-check`.
- [ ] **T061** `PlatformLayout`, rutas `/plataforma/**` y guard: el administrador de la plataforma solo entra ahí y un usuario de empresa vuelve al Inicio. Se retira `PlatformAdminView`. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T062** Empresas: lista con búsqueda, filtros e indicadores; alta con buscador de ubigeo y errores por campo. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T063** Ficha: resumen, administrador con «Reenviar invitación», «Corregir datos legales» (aviso al cambiar el RUC), «Suspender» con motivo y «Reactivar». Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T064** [P] Soporte de la emisión (filtros y «Reintentar») y auditoría de la plataforma. Pruebas incluidas. — Verificación: pruebas en verde.

## Fase 8 · Cierre

- [ ] **T070** API: `php artisan test`, `composer test:pgsql` y `vendor/bin/pint --test` en verde (códigos de salida). — Verificación: salidas adjuntas.
- [ ] **T071** Frontend: `pnpm type-check`, `pnpm lint`, `pnpm test:unit --run` y `pnpm build-only` en verde. — Verificación: salidas adjuntas.
- [ ] **T072** Verificación de extremo a extremo por el proxy de Vite:
  - Alta de una empresa con su administrador, cronometrada (CE-001) y aceptación de la invitación.
  - Corrección de datos legales y cambio de RUC.
  - Suspender (el usuario de la empresa pierde el acceso) y reactivar.
  - Identificar empresas con problemas y reintentar un comprobante desde soporte (CE-003).
  - Un usuario de empresa no entra al panel; búsqueda de secretos y datos de negocio en las respuestas.
  - Capturas a 360 px. Resultado en `verificacion.md`.
  - Verificación: `verificacion.md` con resultado por paso.
- [ ] **T073** Documentación: README de la API y del frontend, `AGENTS.md` y estado de la spec en `specs/README.md`. — Verificación: enlaces válidos.
- [ ] **T074** 🔒 Merge local de `006-panel-plataforma` a `main` en ambos repositorios y commit de la raíz. — Verificación: suites en verde sobre `main`.

## Trazabilidad

| Spec | Tareas |
|---|---|
| HU-1, CE-003 | T020–T021, T062, T072 |
| HU-2, CE-001 | T030, T035–T036, T062, T072 |
| HU-3 | T031–T032, T063 |
| HU-4 | T033–T034, T063 |
| HU-5 | T040–T041, T064 |
| HU-6, RF-003 | T042, T064 |
| RF-001 | T002, T061 |
| RF-002, CE-002 | T050 |
| RF-006 (A-38) | T010–T011 |
| Principio III | T051–T052, T060 |
