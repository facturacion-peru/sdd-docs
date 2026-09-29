# Tareas 004: Configuración segura de SUNAT y series

**Estado:** Aprobado (2026-09-29) · **Fecha:** 2026-09-29 · **Plan:** [plan.md](./plan.md) (aprobado)

> Mismas convenciones que las specs anteriores: tareas pequeñas con su verificación; `[P]` = en paralelo con la anterior; 🔒 = confirmación del usuario al ejecutarla.
> **Pruebas primero:** las tareas «Escribir pruebas» terminan en rojo y la siguiente las pone en verde.
> Antes de cada commit se verifica con el **código de salida** de las suites de la API (`php artisan test`, `composer test:pgsql`, `vendor/bin/pint --test`) o del frontend.

## Fase 1 · Preparación

- [ ] **T001** Crear la rama `004-configuracion-sunat` desde `main` en la API y en el frontend, y declarar `ext-openssl` en `composer.json` (solo cambia el hash del lock). — Verificación: rama en ambos; `composer validate`.

## Fase 2 · Fundamentos

- [ ] **T010** Enums `SunatStatus`, `SunatEnvironment`, `DocumentType` (`01`, `03`, con prefijo de serie `F`/`B`) y `CertificateStatus`, con pruebas. — Verificación: pruebas en verde.
- [ ] **T011** Migraciones `sunat_settings`, `certificates` y `series`, según el plan. — Verificación: `test` y `test:pgsql` en verde; `php artisan migrate` en local.
- [ ] **T012** Modelos `SunatSetting`, `Certificate` y `Series` con `BelongsToCompany`, casts `encrypted` para los secretos y `$hidden`, y *factories*. Prueba: en la base de datos los secretos no aparecen en claro. — Verificación: prueba en verde.

## Fase 3 · Certificado (HU-1)

- [ ] **T020** Escribir pruebas `Unit/Sunat/CertificateInspectorTest` con certificados generados en la prueba: `.pfx` válido (metadatos y PEM), contraseña errónea, archivo dañado, vencido, sin clave privada, RUC distinto, `.pem` válido, y RUC en `serialNumber`, `OU` o `CN`. — Verificación: en rojo.
- [ ] **T021** Implementar `CertificateInspector`. — Verificación: T020 en verde.
- [ ] **T022** Escribir pruebas `Sunat/CertificateUploadTest` y `Sunat/SunatCredentialsTest`:
  - HU-1.1: credenciales y certificado se guardan; estado `pending`.
  - HU-1.2/1.3: errores concretos del certificado.
  - HU-1.4: la configuración expone usuario enmascarado, `has_sol_password`, titular, RUC y vencimiento, nunca secretos.
  - HU-1.5: el vendedor no ve ni cambia la configuración.
  - Reemplazar el certificado conserva el historial.
  - Auditoría sin valores.
  - Verificación: en rojo.
- [ ] **T023** Implementar `SunatConfigService` (credenciales y certificado), `GET /sunat/settings`, `PUT /sunat/credentials` y `POST /sunat/certificate`, con requests y recursos. — Verificación: T022 en verde en ambas bases.

## Fase 4 · Validación y estado (HU-2, HU-4)

- [ ] **T030** Escribir pruebas `Sunat/ValidateConfigurationTest` (con `Http::fake`):
  - HU-2.1: certificado válido y SUNAT responde → `validated`.
  - HU-2.2: incompleta → lista de lo que falta.
  - HU-2.3: SUNAT no responde → no queda en `error`, con motivo «SUNAT no responde».
  - HU-2.4: cambiar la clave o el certificado vuelve a `pending`.
  - HU-2.5 y RF-015: `sol_verified` falso en beta.
  - Certificado vencido después de validar → `error`.
  - Verificación: en rojo.
- [ ] **T031** Implementar `SunatConnectionChecker`, `POST /sunat/validate` y el cálculo de estado. — Verificación: T030 en verde.
- [ ] **T032** Escribir y poner en verde `Sunat/SunatStatusTest` (HU-4): `GET /sunat/status` para cualquier rol, con estado, ambiente, `can_issue`, lo que falta y los días para vencer el certificado; empresa desactivada → `inactive`. — Verificación: en verde.

## Fase 5 · Series (HU-3)

- [ ] **T040** Escribir pruebas `Sunat/SeriesTest`:
  - HU-3.1: serie con último número usado; el siguiente es el que sigue.
  - HU-3.2: formato por tipo.
  - HU-3.3: duplicada en la empresa, rechazada (en otra empresa se permite).
  - HU-3.4: desactivar sí; bajar el correlativo o borrar, no.
  - HU-3.5: listado con último y siguiente número, visible para el vendedor.
  - HU-3.6: el Nuevo RUS no crea series de factura.
  - El vendedor no crea ni desactiva; auditoría.
  - Verificación: en rojo.
- [ ] **T041** Implementar `SeriesService`, `GET/POST /series` y `PATCH /series/{series}`. — Verificación: T040 en verde.
- [ ] **T042** Escribir `Integration/Sunat/ConcurrentSeriesNumberTest` (solo PostgreSQL, `pcntl_fork`): `nextNumber()` desde N procesos da números consecutivos sin repetir (CE-003). Verificar con una mutación sin bloqueo que la prueba falla. — Verificación: en verde en `test:pgsql` y omitida en `test`.

## Fase 6 · Seguridad, aislamiento y contrato

- [ ] **T050** Escribir y poner en verde `Security/NoSunatSecretsLeakTest` (CE-002): recorrido completo (credenciales, certificado, validación, series) buscando la clave SOL, la contraseña del certificado y fragmentos del PEM en respuestas, logs, auditoría y, en claro, en toda la base de datos. — Verificación: en verde, y una mutación que devuelve la clave en la respuesta la hace fallar.
- [ ] **T051** Añadir las rutas a `TenantRoutes.php` y configuración, certificado y serie de B en `CrossTenantAccessTest`. — Verificación: cobertura y acceso cruzado en verde.
- [ ] **T052** [P] `Contract/SunatResourcesTest`. — Verificación: en verde.
- [ ] **T053** `php artisan openapi:generate`; en `DevelopmentSeeder`, series demo (F001, B001) y un certificado autofirmado de prueba para la empresa demo, que quedará validada si SUNAT beta responde. README de la API: cifrado y rotación de `APP_KEY` con `APP_PREVIOUS_KEYS`. — Verificación: OpenAPI con las rutas nuevas; `migrate:fresh --seed` en local (autorizado).

## Fase 7 · Frontend

- [ ] **T060** `pnpm api:types`; `features/sunat/types.ts` y `api.ts`. — Verificación: `pnpm type-check`.
- [ ] **T061** Pantalla `/sunat`: estado con motivo, credenciales (la clave nunca se muestra), subida del certificado con contraseña y metadatos, «Validar», aviso de clave SOL no verificada en beta, historial de certificados y `EnvironmentBadge` («PRUEBAS — sin valor legal»). Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T062** Pantalla `/series`: por tipo, con último y siguiente número, alta (con último número usado) y desactivación; errores de formato y del Nuevo RUS. Pruebas incluidas. — Verificación: pruebas en verde.
- [ ] **T063** [P] Indicador «Emisión SUNAT» en el Inicio para todos, aviso de certificado por vencer para el administrador, y «SUNAT» y «Series» en el menú «Más». Pruebas incluidas. — Verificación: pruebas en verde.

## Cierre

- [ ] **T070** API: `php artisan test`, `composer test:pgsql` y `vendor/bin/pint --test` en verde (códigos de salida). — Verificación: salidas adjuntas.
- [ ] **T071** Frontend: `pnpm type-check`, `pnpm lint`, `pnpm test:unit --run` y `pnpm build-only` en verde. — Verificación: salidas adjuntas.
- [ ] **T072** Verificación de extremo a extremo por el proxy de Vite:
  - Credenciales, certificado de prueba y errores del certificado.
  - Validación real contra el WSDL de SUNAT beta.
  - Series, vendedor sin acceso a la configuración, y otra empresa.
  - Búsqueda de secretos en las respuestas.
  - Capturas a 360 px. Resultado en `verificacion.md`.
  - Verificación: `verificacion.md` con resultado por paso.
- [ ] **T073** Documentación: README de la API y del frontend, `AGENTS.md` y estado de la spec en `specs/README.md`. — Verificación: enlaces válidos.
- [ ] **T074** 🔒 Merge local de `004-configuracion-sunat` a `main` en ambos repositorios y commit de la raíz. — Verificación: suites en verde sobre `main`.

## Trazabilidad

| Spec | Tareas |
|---|---|
| HU-1, RF-001–005 | T012, T020–T023, T061 |
| HU-2, RF-010–015 | T030–T031, T061 |
| HU-4 | T032, T063 |
| HU-3, RF-020–025 | T040–T041, T062 |
| CE-003 | T042 |
| CE-002, RF-002 | T012, T050 |
| RF-030 | T022, T030, T040 |
| Principio VIII | T051 |
| Principio III | T052–T053, T060 |
