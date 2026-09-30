# Plan 006: Panel de administración de la plataforma

**Estado:** Borrador · **Fecha:** 2026-09-29 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

1. **API:** se amplía el grupo `platform` de la 001 (`auth:sanctum` + `platform.admin`) con:
   - resúmenes por empresa (estado SUNAT y contadores), búsqueda y filtro «con problemas»;
   - corrección del domicilio fiscal, desactivación con motivo y reenvío de la invitación del administrador;
   - soporte de la emisión (pendientes y rechazados de todas las empresas, con reintento);
   - auditoría de la plataforma.

   Todo con **recursos propios de plataforma**, que exponen solo metadatos (A-37).
2. **Lecturas entre empresas:** el administrador de la plataforma no tiene contexto de empresa. Las lecturas agregadas usan `withoutTenancy()` (la salida prevista para código de plataforma, principio VIII). Las escrituras sobre datos de una empresa (reintento) corren dentro de `TenantContext::run($empresa)`.
3. **Sesión:** los tokens de la plataforma vencen a las 8 h y cada inicio de sesión de la plataforma se audita (A-38).
4. **Frontend:** una feature nueva, `platform`, con su propio layout en `/plataforma` (A-35). El aviso actual que manda a la consola desaparece.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos | API y frontend |
| Dependencias nuevas | Ninguna |
| Migraciones | Ninguna tabla nueva. El motivo de la suspensión va en la auditoría (`company.deactivated`), no en `companies` |
| APK | Las pantallas de plataforma se cargan con `import()` diferido; al empaquetar con Capacitor se pueden excluir. Queda anotado en el README |

## Chequeo de constitución (v1.1.2)

| Principio | Cumple | Cómo / justificación |
|---|---|---|
| I. Spec primero | ✅ | Spec 1.0 aprobada; A-35 a A-38 resueltas. |
| II. Corrección fiscal | ✅ | Cambiar el RUC invalida la configuración SUNAT; los comprobantes ya emitidos conservan su copia (RF-006 de la 005). |
| III. Contrato de API | ⚠️ | Entradas por FormRequest (incluidos los filtros); respuestas protegidas con pruebas de contrato (excepción heredada). |
| IV. Seguridad | ✅ | Recursos de plataforma con lista blanca de campos; prueba que busca secretos y datos de negocio en todas las respuestas del panel (CE-002); tokens de 8 h; 2FA registrado como requisito de producción. |
| V. Simplicidad | ✅ | Sin tecnología nueva (A-35) ni tablas nuevas; contadores calculados con subconsultas. |
| VI. Verificable | ✅ | Pruebas por historia, de privacidad, de sesión y de contrato; recorrido de extremo a extremo. |
| VII. Personas reales | ✅ | Filtro «con problemas de emisión» para responder en menos de 1 minuto (CE-003); motivos en español. |
| VIII. Aislamiento y auditoría | ✅ | Las rutas de plataforma quedan fuera del grupo de empresa; los usuarios de empresa reciben 403. Toda acción queda auditada con su autor. |

## Contrato de API (`/api/v1/platform`, solo administrador de la plataforma)

| Método | Ruta | Descripción | Spec |
|---|---|---|---|
| GET | `/companies?search=&status=&issues=` | Lista paginada con resumen: activa, estado SUNAT efectivo, usuarios, comprobantes pendientes y rechazados. `issues=1` deja solo las que tienen pendientes, rechazados o SUNAT sin validar | HU-1.1/1.2, CE-003 |
| GET | `/companies/{company}` | Ficha: datos legales, domicilio fiscal, administrador (nombre, correo, invitación aceptada o pendiente), usuarios activos, última actividad (último ticket o comprobante) y resumen | HU-1.3 |
| POST | `/companies` | Alta (sin cambios respecto a la 001) | HU-2.1/2.2 |
| PATCH | `/companies/{company}` | Datos legales, ahora también `fiscal_address.address` y `fiscal_address.ubigeo`. Si cambia el RUC, la configuración SUNAT vuelve a `pending` con el motivo | HU-3 |
| POST | `/companies/{company}/deactivate` | Ahora exige `{ reason }`, que queda en la auditoría | HU-4.1 |
| POST | `/companies/{company}/activate` | Sin cambios | HU-4.2 |
| POST | `/companies/{company}/admin-invitation/resend` | Reenvía la invitación pendiente del administrador (el enlace anterior deja de valer); 422 si ya la aceptó | HU-2.3 |
| GET | `/ubigeos/search?q=` | Búsqueda de ubigeo para el alta y la corrección (el de empresa exige contexto de empresa) | HU-2, HU-3 |
| GET | `/sales-documents?status=&company_id=` | Pendientes y rechazados de todas las empresas: empresa (RUC y razón social), número, tipo, estado, código y mensaje de SUNAT, intentos y fechas. Sin líneas, cliente ni importes | HU-5.1 |
| POST | `/sales-documents/{document}/retry` | Reintento con `SunatDispatcher` dentro del contexto de la empresa; se audita `sales_document.retry_requested` con la plataforma como autor | HU-5.2 |
| GET | `/audit-logs?action=&company_id=` | Acciones de administradores de la plataforma e inicios de sesión de la plataforma | HU-6 |

- **Binding sin contexto:** `{document}` en las rutas de plataforma se resuelve con un binding propio (`withoutTenancy`), igual que `{company}`, para no disparar `TenantContextMissing`.
- **Validación con RUC cambiado:** `SunatConfigService::validate()` comprueba además que el RUC del certificado vigente coincida con el de la empresa. Si no coincide, la configuración queda en error con el motivo «El certificado es del RUC X; sube uno del RUC actual.».
- **Privacidad:** `PlatformCompanyResource`, `PlatformCompanyDetailResource` y `PlatformSalesDocumentResource` son listas blancas. `CompanyResource`, que usa la empresa, no se reutiliza para las respuestas nuevas.

## Sesión (A-38)

- Al iniciar sesión, un administrador de la plataforma recibe un token con `expires_at` a las 8 h (Sanctum lo respeta aunque la expiración global sea de 24 h). Los usuarios de empresa no cambian.
- Se audita `auth.login` para la plataforma (sin empresa), con IP y agente. Los inicios fallidos ya se auditaban (001).
- El segundo factor queda anotado en A-24 como requisito previo a producción.

## Frontend: feature `platform`

| Ruta | Pantalla |
|---|---|
| `/plataforma` | **Empresas:** búsqueda, filtros (activas, suspendidas, «con problemas») y, por empresa, estado SUNAT y pendientes o rechazados |
| `/plataforma/empresas/nueva` | **Alta:** datos legales, régimen, domicilio fiscal con buscador de ubigeo y correo del administrador |
| `/plataforma/empresas/:id` | **Ficha:** datos y resumen; administrador con «Reenviar invitación»; «Corregir datos legales» (con aviso si se cambia el RUC); «Suspender» (pide motivo) o «Reactivar» |
| `/plataforma/soporte` | **Soporte de la emisión:** pendientes y rechazados de todas las empresas, filtro por empresa y «Reintentar» |
| `/plataforma/auditoria` | **Auditoría de la plataforma** |

- `PlatformLayout` propio (cabecera «Plataforma», navegación Empresas · Soporte · Auditoría, cerrar sesión), sin el menú de las empresas.
- **Guard:**
  - el administrador de la plataforma solo entra a `/plataforma/**` (hoy ya se le redirige);
  - un usuario de empresa que intente `/plataforma/**` vuelve al Inicio;
  - un token vencido (401) lleva al login como en el resto de la app.
- La feature no importa de otras features (regla del frontend): sus tipos y su API son propios.

## Estrategia de pruebas

**API** (SQLite y PostgreSQL):

| Escenario / requisito | Prueba |
|---|---|
| HU-1 | `Platform/CompanySummaryTest`: resumen y contadores correctos con datos en varias empresas, búsqueda, filtro `issues`, ficha con administrador y última actividad |
| HU-2.3 | `Platform/ResendAdminInvitationTest`: nuevo enlace, el anterior deja de valer, 422 si ya aceptó, auditoría |
| HU-3 | `Platform/CorrectLegalDataTest`: domicilio fiscal; cambio de RUC → SUNAT `pending` con motivo; RUC de otra empresa → 422; `validate()` con certificado de otro RUC → error; los comprobantes emitidos conservan el RUC |
| HU-4 | `ManageCompaniesTest` ampliado: motivo obligatorio y auditado; los reintentos de una empresa suspendida siguen |
| HU-5 | `Platform/EmissionSupportTest`: lista entre empresas sin líneas, cliente ni importes; reintento con `FakeSunatSender` y auditoría con la plataforma como autor |
| HU-6 | `Platform/PlatformAuditTest` |
| A-38 | `Platform/PlatformSessionTest`: token de la plataforma vence a las 8 h (401 a las 8 h 1 min) y el de empresa no; inicio de sesión auditado |
| CE-002, A-37 | `Security/PlatformPrivacyTest`: recorre todas las rutas de plataforma con datos sembrados y busca secretos, nombres de productos, clientes, líneas e importes de comprobantes |
| Acceso | `Platform/PlatformAccessTest`: toda ruta `platform/*` responde 403 a usuarios de empresa y 401 sin sesión (lista generada desde el router, como `TenantRoutesCoverageTest`) |
| Contratos | `Contract/PlatformResourcesTest` |

**Frontend (Vitest):** guard de `/plataforma`; lista con filtros e indicadores; alta con errores por campo; ficha (reenviar invitación, corregir datos con aviso de RUC, suspender con motivo, reactivar); soporte con reintento; auditoría.

**Verificación manual:** recorrido de extremo a extremo por el proxy de Vite (alta de una empresa en menos de 3 minutos, CE-001; identificar empresas con problemas, CE-003; privacidad) y capturas a 360 px.

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Filtrar datos de negocio o secretos por un recurso mal hecho | Recursos de plataforma con lista blanca y `PlatformPrivacyTest` sobre todas las rutas |
| Una ruta de plataforma olvida el middleware | `PlatformAccessTest` recorre todas las rutas `platform/*` del router |
| Consultas agregadas lentas con muchas empresas | Subconsultas con los índices existentes (`status`, `company_id`); volumen de A-06. Se revisa si pasa de cientos de empresas |
| Cambiar el RUC deja una configuración SUNAT incoherente | Vuelve a `pending` y `validate()` exige que el RUC del certificado coincida |
| El código de plataforma termina dentro del APK | Carga diferida; exclusión documentada para la spec del APK |

## Aprobación

Pendiente.
