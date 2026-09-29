# Verificación 004: Configuración segura de SUNAT y series

**Fecha:** 2026-09-29 · **Tarea:** T072 · **Rama:** `004-configuracion-sunat`

## Suites (T070, T071)

| Proyecto | Resultado |
|---|---|
| API, SQLite | ✅ 380 pruebas |
| API, PostgreSQL | ✅ 380 pruebas, incluida la numeración concurrente de series (8 procesos → 151–158) |
| API, Pint | ✅ |
| Frontend | ✅ `type-check`, `lint`, 114 pruebas y `build-only` |

## Recorrido de extremo a extremo (por el proxy de Vite, contra SUNAT beta real)

Hecho con la empresa «Otra Empresa S.A.C.» (sin configuración en el seeder) y certificados autofirmados generados para la prueba. 21/21 pasos correctos.

| # | Paso | Spec | Resultado |
|---|---|---|---|
| 1 | Empresa sin configurar: estado y qué falta | HU-4, HU-2.2 | ✅ «Registra el usuario y la clave SOL», «Sube el certificado digital» |
| 2 | Validar sin datos | HU-2.2 | ✅ 422 con lo que falta |
| 3 | Credenciales SOL | HU-1.1 | ✅ usuario `VE****01`, clave «registrada», estado pendiente |
| 4 | Certificado con contraseña errónea | RF-003 | ✅ «La contraseña del certificado es incorrecta.» |
| 5 | Archivo dañado | RF-003 | ✅ «El archivo no es un certificado válido o está dañado.» |
| 6 | Certificado vencido | RF-003 | ✅ «El certificado venció el 01/01/2025.» |
| 7 | Certificado de otro RUC | RF-003 | ✅ «El certificado es del RUC 20600000013, no del de la empresa (20600000021).» |
| 8 | Certificado válido (.pfx) | HU-1.4 | ✅ solo metadatos (titular, RUC, vigencia) |
| 9 | **Validación real contra el WSDL de SUNAT beta** | HU-2.1, A-31 | ✅ «Validada»; clave SOL marcada como no verificada |
| 10 | El estado permite emitir | HU-4 | ✅ |
| 11 | Reemplazo por un .pem con el RUC en `OU` | RF-012 | ✅ vuelve a pendiente; historial vigente + reemplazado |
| 12 | Serie F001 escrita en minúsculas | HU-3.1 | ✅ se normaliza; siguiente número 1 |
| 13 | Serie con último número usado 150 | HU-3.6 | ✅ siguiente 151 |
| 14 | Boleta con serie F002 | HU-3.2 | ✅ 422 con el formato esperado |
| 15 | Serie duplicada | HU-3.3 | ✅ «Ya existe la serie F001 en tu empresa.» |
| 16 | Editar el correlativo | HU-3.4 | ✅ 422 |
| 17 | Desactivar serie | HU-3.4 | ✅ |
| 18 | Vendedor | HU-3.5, HU-4 | ✅ 403 en configuración, credenciales y validar; ve series y estado |
| 19 | Otra empresa | Principio VIII | ✅ su serie da 404 y no aparece en el listado |
| 20 | Auditoría | RF-030 | ✅ credenciales, certificado subido y reemplazado, validación, series creadas y desactivadas |
| 21 | Búsqueda de secretos | CE-002 | ✅ la clave SOL, la contraseña del certificado y la clave privada no aparecen en las 29 respuestas, en `laravel.log` ni en claro en la base de datos |

El seeder también validó la empresa demo contra SUNAT beta real al ejecutar `migrate:fresh --seed`.

## Pantallas a 360 px

| Pantalla | Resultado |
|---|---|
| SUNAT (administrador) | ✅ Distintivo «PRUEBAS — sin valor legal», estado, credenciales enmascaradas con aviso de clave no verificada, metadatos del certificado e historial |
| Series (administrador) | ✅ Facturas y boletas con último y siguiente número, serie desactivada, alta con último número usado |
| Inicio (administrador) | ✅ «Emisión SUNAT: Disponible» con enlace a la configuración |
| Inicio (vendedor) | ✅ Mismo indicador, sin enlace ni aviso de vencimiento |

## Hallazgo

- **«/» mostraba un AuthLayout vacío al recargar.** Venía de la spec 001: los dos layouts cuelgan de «/» y ganaba el primero declarado. Nunca se notó porque el login navega por nombre (`home`). Se corrigió el orden en el router, con una prueba de regresión.

## Pendiente de confirmación del usuario

- **Certificado real:** el recorrido usó certificados autofirmados. Conviene subir el certificado real de una empresa para confirmar en qué campo trae el RUC (se busca en `serialNumber`, `OU` y `CN`).
- **Datos de prueba:** el recorrido dejó configuración SUNAT y las series F001 y B002 en «Otra Empresa». Se limpian con `php artisan migrate:fresh --seed`.
