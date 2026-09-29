# Plan 004: Configuración segura de SUNAT y series

**Estado:** Borrador (pendiente de aprobación) · **Fecha:** 2026-09-29 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

Sobre la base de las specs 001 a 003 (aislamiento, auditoría, `bcmath`):

1. Tres tablas nuevas:
   - `sunat_settings`: una fila por empresa, con estado, ambiente y usuario y clave SOL.
   - `certificates`: el vigente y los reemplazados.
   - `series`: con su correlativo.
2. Los secretos (clave SOL, certificado en PEM y su contraseña) se guardan con el cast `encrypted` de Laravel (AES-256, clave `APP_KEY`) **en la base de datos**, nunca en el disco público. Se marcan como `$hidden` y ninguna respuesta ni log los incluye (principio IV, RF-001/002).
3. El certificado se lee con la extensión `openssl` de PHP: acepta `.pfx`/`.p12` (con contraseña) o `.pem`. Se comprueba la contraseña, la vigencia y que el RUC del titular coincida con el de la empresa. Solo se exponen metadatos: titular, RUC y vigencia.
4. **Validar** (A-31) = certificado vigente y válido + conexión con SUNAT beta, comprobada leyendo el WSDL de `SunatEndpoints::FE_BETA`. En beta los envíos usan las credenciales genéricas de prueba de SUNAT. La clave SOL real queda guardada y marcada como «no verificada».
5. Las series validan el formato SUNAT y el régimen (el Nuevo RUS no emite facturas). Su correlativo solo avanza con la emisión (spec 005), con el mismo patrón de bloqueo que la numeración de tickets.
6. En el frontend, una feature nueva `sunat` (solo el administrador) y un indicador de estado de la emisión para todos los usuarios.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos | API y frontend |
| Dependencias nuevas | Ninguna. Se usan `ext-openssl` (declarada en `composer.json` como requisito de plataforma) y Greenter, ya instalado, solo para la URL de beta |
| Cifrado | Cast `encrypted` de Laravel con `APP_KEY`. Rotar la clave exige `APP_PREVIOUS_KEYS` para poder descifrar lo existente; queda documentado en el README |
| Conexión con SUNAT | Cliente HTTP de Laravel con tiempo límite; en pruebas se simula con `Http::fake()` |
| Pruebas | Certificados autofirmados generados en la propia prueba con `openssl` (válido, vencido, otro RUC, contraseña errónea) |

## Chequeo de constitución (v1.1.2)

| Principio | Cumple | Cómo / justificación |
|---|---|---|
| I. Spec primero | ✅ | Spec 1.0 aprobada; A-20, A-21 y A-31 resueltas. |
| II. Corrección fiscal | ✅ | Formato de series SUNAT (4 caracteres, `F` o `B`) y regla del Nuevo RUS en el servidor; en beta nunca se usa producción (RF-013). |
| III. Contrato de API | ⚠️ | OpenAPI y tipos regenerados; entradas por FormRequest. Las respuestas se protegen con pruebas de contrato (excepción heredada). |
| IV. Seguridad | ✅ | Secretos cifrados en la base de datos, fuera de respuestas, logs y auditoría (prueba específica); solo el administrador los gestiona; archivo subido validado y descartado tras procesarlo. |
| V. Simplicidad | ✅ | Un certificado vigente, un ambiente y sin OSE/PSE; la validación de conexión es una lectura del WSDL, sin enviar documentos. |
| VI. Verificable | ✅ | Pruebas por escenario, prueba de no filtración de secretos (CE-002) y de correlativo concurrente (CE-003). |
| VII. Personas reales | ✅ | Estados y motivos en español, aviso de vencimiento del certificado y distintivo de «PRUEBAS» siempre visible. |
| VIII. Aislamiento y auditoría | ✅ | Modelos con `BelongsToCompany`; rutas en `TenantRoutes.php`; se auditan registro y reemplazo de secretos, validaciones y series, sin valores. |

## Modelo de datos

| Tabla | Columnas relevantes | Restricciones |
|---|---|---|
| `sunat_settings` | `company_id` (único), `environment` (`beta`), `status` (`not_configured`/`pending`/`validated`/`error`/`inactive`), `sol_user`, `sol_password` (**cifrado**), `sol_verified_at` nulo, `last_validated_at`, `last_validation_error`, `validated_by` | una por empresa |
| `certificates` | `company_id`, `pem` (**cifrado**: certificado + clave privada), `password` (**cifrado**), `subject`, `ruc`, `serial_number`, `valid_from`, `valid_to`, `status` (`current`/`replaced`), `uploaded_by`, `replaced_at` | a lo sumo un `current` por empresa |
| `series` | `company_id`, `establishment_id`, `document_type` (`01` factura, `03` boleta), `code` (`F001`), `last_number`, `active` | único (`company_id`, `code`) |

- **Estados** (RF-010/012): cualquier cambio de clave SOL o de certificado lleva la configuración a `pending`; `validate()` la lleva a `validated` o `error`; la empresa desactivada la muestra como `inactive`.
- **Serie:** el formato se valida por tipo (`^F[A-Z0-9]{3}$` o `^B[A-Z0-9]{3}$`). `last_number` es el último número usado, 0 por defecto. `nextNumber()` bloquea la fila, igual que la secuencia de tickets; lo usa la spec 005 y se prueba ya con concurrencia.

## Servicios

| Pieza | Responsabilidad | Spec |
|---|---|---|
| `CertificateInspector` | Lee `.pfx`/`.p12`/`.pem` con `openssl` y devuelve PEM y metadatos, o un error concreto: contraseña incorrecta, archivo dañado, vencido, sin clave privada, RUC distinto | HU-1.2/1.3, RF-003 |
| `SunatConfigService` | Guarda credenciales y certificado (el anterior pasa a `replaced`, RF-012), valida (certificado + conexión) y calcula el estado; audita sin valores | HU-1, HU-2 |
| `SunatConnectionChecker` | Lee el WSDL de beta con tiempo límite; «SUNAT no responde» no marca la configuración como errónea (HU-2.3) | HU-2 |
| `SeriesService` | Alta con último número usado, desactivación, `nextNumber()` bloqueante, regla del Nuevo RUS | HU-3 |

- El RUC del titular se busca en los campos del certificado donde SUNAT lo pone (`serialNumber`, `OU`, `CN`).
- En beta, las credenciales de envío (usuario `{RUC}MODDATOS`, clave `moddatos`) salen de configuración, no de la base de datos (RF-014).

## Contrato de API (`/api/v1`, grupo de empresa)

| Método | Ruta | Rol | Descripción | Spec |
|---|---|---|---|---|
| GET | `/sunat/status` | Todos | Estado, ambiente, si se puede emitir, qué falta, días para vencer el certificado. Sin secretos | HU-4 |
| GET | `/sunat/settings` | Admin | Configuración: usuario SOL enmascarado, `has_sol_password`, `sol_verified`, metadatos del certificado vigente, historial de certificados, última validación | HU-1.4 |
| PUT | `/sunat/credentials` | Admin | `{ sol_user, sol_password }` → estado `pending` | HU-1.1 |
| POST | `/sunat/certificate` | Admin | Multipart `{ certificate, password }` → valida y reemplaza → `pending` | HU-1.1–1.3 |
| POST | `/sunat/validate` | Admin | Valida y devuelve el estado con el motivo o lo que falta | HU-2 |
| GET | `/series` | Todos | Series con último y siguiente número (el vendedor las necesitará para emitir en la 005) | HU-3.5 |
| POST | `/series` | Admin | `{ document_type, code, last_number? }` | HU-3.1–3.3, 3.6 |
| PATCH | `/series/{series}` | Admin | `{ active }`. No admite `last_number`: el correlativo no se edita | HU-3.4 |

## Frontend: feature `sunat`

| Ruta | Rol | Pantalla |
|---|---|---|
| `/sunat` | Admin | Configuración: estado con motivo, credenciales SOL (la clave nunca se muestra; «registrada»), subida del certificado con contraseña y sus metadatos, botón «Validar», aviso de que en beta la clave SOL real no está verificada, e historial de certificados |
| `/series` | Admin | Series por tipo, con último y siguiente número, alta (con último número usado) y desactivación |

- **Indicadores:** «Emisión SUNAT» en el Inicio para todos los usuarios (disponible o qué falta), y un aviso al administrador si el certificado vence en 30 días o menos (HU-4.1, HU-4.3).
- **Distintivo «PRUEBAS — sin valor legal»** (HU-4.2): un componente compartido, `EnvironmentBadge`, que se muestra en la pantalla SUNAT y que usará la emisión de la 005.
- **Navegación:** «SUNAT» y «Series» dentro del menú «Más».

## Estrategia de pruebas

**API** (SQLite y PostgreSQL):

| Escenario / requisito | Prueba |
|---|---|
| RF-003: contraseña errónea, dañado, vencido, sin clave privada, otro RUC | `Unit/Sunat/CertificateInspectorTest` (certificados generados en la prueba) |
| HU-1.1–1.5 | `Sunat/SunatCredentialsTest`, `Sunat/CertificateUploadTest` |
| HU-2.1–2.5, RF-010–015 | `Sunat/ValidateConfigurationTest` (con `Http::fake`: SUNAT responde, falla o no responde) |
| HU-3.1–3.6, RF-020–025 | `Sunat/SeriesTest` |
| CE-003 correlativo concurrente | `Integration/Sunat/ConcurrentSeriesNumberTest` (solo PostgreSQL) |
| HU-4 | `Sunat/SunatStatusTest` |
| CE-002 | `Security/NoSunatSecretsLeakTest`: recorrido completo buscando la clave SOL, la contraseña y el PEM en respuestas, logs, auditoría y (en claro) en la base de datos |
| RF-030 | Auditoría en cada prueba de escritura |
| Aislamiento | Rutas en `TenantRoutes.php`; configuración, certificado y serie de B en `CrossTenantAccessTest` |
| Contratos | `Contract/SunatResourcesTest` |

**Frontend (Vitest):** configuración (la clave nunca aparece, subida con errores del certificado, validar con estados y aviso de clave no verificada), series (alta con último número, formato, desactivar), indicador de estado y aviso de vencimiento, y `EnvironmentBadge`.

**Verificación manual:** recorrido de extremo a extremo con un certificado de prueba y validación real contra el WSDL de SUNAT beta; capturas a 360 px.

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Filtración de secretos | Cast `encrypted` + `$hidden` + recursos que exponen solo metadatos + saneado de la auditoría + prueba de filtración |
| Pérdida de `APP_KEY` o rotación sin plan | README: `APP_PREVIOUS_KEYS` para rotar; perder la clave obliga a volver a subir certificados y clave SOL |
| Certificados reales con el RUC en campos distintos | Se buscan `serialNumber`, `OU` y `CN`; si no se encuentra, el error lo explica |
| SUNAT beta caída durante la validación | Estado aparte, «SUNAT no responde»: no marca la configuración como errónea |
| Correlativos duplicados en la 005 | `nextNumber()` bloqueante ya probado con concurrencia en esta spec |
