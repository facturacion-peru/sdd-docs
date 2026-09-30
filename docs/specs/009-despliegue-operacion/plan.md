# Plan 009: Despliegue y operación

**Estado:** Aprobado (2026-09-30) · **Fecha:** 2026-09-30 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

1. **Un VPS** (Ubuntu 24.04 LTS) preparado con un script idempotente del repositorio: Nginx, PHP 8.5-FPM, PostgreSQL local, HTTPS de Let's Encrypt, firewall y actualizaciones de seguridad automáticas (A-44).
2. **Dos ambientes en el mismo servidor**, cada uno con su base de datos, su `.env` y sus subdominios:
   - **staging**: se despliega solo con cada cambio en `main`;
   - **producción del piloto**: se despliega la misma versión, con una aprobación manual en GitHub.

   Los dos usan SUNAT beta: el ambiente de producción de SUNAT es la spec 008.
3. **CI/CD con GitHub Actions** en repositorios privados propios (A-46):
   - las suites completas en cada cambio;
   - la API se empaqueta sin dependencias de desarrollo y la web se compila en CI;
   - el despliegue va por SSH con carpetas de versión y un enlace simbólico: sin tiempo de caída y con vuelta atrás inmediata.
4. **Copias** con `spatie/laravel-backup`: base de datos y logos, cifradas, en almacenamiento compatible con S3, con 30 diarias y 12 semanales, más restauración de prueba mensual con un script (A-45).
5. **Avisos por correo** (A-48):
   - monitores externos de disponibilidad y del latido del *scheduler*;
   - un comando `ops:check` cada 15 minutos (pendientes SUNAT, copias, disco);
   - aviso de errores 500 repetidos, con límite de frecuencia.
6. **Correo** por el SMTP de un proveedor transaccional, sin dependencias nuevas (A-47).
7. **Aviso de piloto** en la aplicación: no registrar datos reales de clientes (A-43, RF-009).

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos | API, frontend y un nuevo `deploy/` en el repositorio de la API (scripts del servidor, plantillas de Nginx y de `.env`) |
| Dependencias nuevas | `spatie/laravel-backup` (copias, retención por días y semanas, cifrado del zip y monitoreo de antigüedad; MIT, muy usada) y `league/flysystem-aws-s3-v3` (destino S3). Justificación en el chequeo de constitución |
| Migraciones | Ninguna tabla nueva |
| Servicios externos | Proveedor de VPS, DNS del dominio, GitHub (repositorios y Actions), almacenamiento S3 (p. ej. Backblaze B2, Cloudflare R2 o AWS S3), proveedor de correo (p. ej. SES, Resend o Brevo) y monitores gratuitos (p. ej. Healthchecks.io para latidos y UptimeRobot para disponibilidad) |

## Quién hace qué

| Paso | Responsable |
|---|---|
| Contratar el VPS, el dominio y el DNS | **Tú** |
| Crear los repositorios privados en GitHub y cargar los secretos de Actions (claves SSH, `.env`) | **Tú**, con la lista que preparo |
| Abrir las cuentas de almacenamiento S3, correo y monitores, y crear sus claves | **Tú** |
| Guardar `APP_KEY` y la clave de cifrado de copias en tu gestor de contraseñas | **Tú** |
| Scripts de preparación y despliegue, plantillas, workflows de CI, cambios en la aplicación, procedimiento de restauración y README | **Yo** |
| Primer despliegue, restauración de prueba y verificación de avisos | **Juntos**: tú das el acceso; yo ejecuto y documento |

## Chequeo de constitución (v1.1.2)

| Principio | Cumple | Cómo / justificación |
|---|---|---|
| I. Spec primero | ✅ | Spec 1.0 aprobada; A-43 a A-48 resueltas. |
| II. Corrección fiscal | ✅ | Sigue en SUNAT beta; nada cambia en el cálculo ni en la emisión. |
| III. Contrato de API | ✅ | Sin cambios de contrato, salvo el endpoint de salud, sin datos. |
| IV. Seguridad | ✅ | HTTPS, `APP_DEBUG=false`, secretos fuera del repositorio (secretos de Actions y `.env` solo en el servidor), SSH solo con llave, firewall, PostgreSQL sin acceso externo, copias cifradas y `APP_KEY` aparte, y el despliegue aborta si `APP_ENV` no es `production` o `staging`. |
| V. Simplicidad | ⚠️ | **Dos dependencias nuevas** para copias. Hacerlo a mano (`pg_dump`, cifrado, subida y retención por días y semanas, más aviso por antigüedad) es más código propio y menos probado. Un solo servidor, sin contenedores ni orquestadores. |
| VI. Verificable | ✅ | CI obligatoria antes de desplegar, staging antes de producción, restauración de prueba mensual (CE-001) y caída simulada (CE-002). |
| VII. Personas reales | ✅ | Aviso de piloto claro; correos desde el dominio del SaaS, fuera de spam. |
| VIII. Aislamiento y auditoría | ✅ | Sin cambios en el aislamiento; la auditoría viaja en las copias. |

## Servidor

| Pieza | Decisión |
|---|---|
| Sistema | Ubuntu 24.04 LTS, usuario `deploy` sin contraseña (solo llave SSH), `root` sin acceso por SSH |
| Seguridad | `ufw` (22, 80, 443), `fail2ban`, `unattended-upgrades` |
| PHP | 8.5-FPM con `bcmath`, `gd`, `openssl`, `pgsql`, `intl`, `mbstring`, `xml`, `zip`, `curl`; OPcache activo |
| Base de datos | PostgreSQL 16, solo en `localhost`; bases `sunat_production` y `sunat_staging`, con usuarios separados |
| Web | Nginx: `app.<dominio>` y `staging-app.<dominio>` sirven los archivos compilados de la web (con recarga de SPA); `api.<dominio>` y `staging-api.<dominio>` van a PHP-FPM. Cabeceras de seguridad y HTTPS con Certbot (renovación automática) |
| Estructura | `/srv/sunat/<ambiente>/{api,web}/releases/<fecha>`, un enlace `current`, y `shared/` con `.env` y `storage/` |
| Tareas | Cron del usuario `deploy`: `schedule:run` cada minuto por ambiente |
| Logs | Canal `daily` de Laravel con 14 días; `logrotate` para Nginx |

El script `deploy/provision.sh` deja el servidor en este estado y se puede volver a ejecutar sin daño.

## Despliegue (A-46)

| Paso | Detalle |
|---|---|
| CI de la API | En cada *push* y *pull request*: `php artisan test`, las mismas pruebas en PostgreSQL (servicio de Actions) y `vendor/bin/pint --test` |
| CI del frontend | `pnpm install --frozen-lockfile`, `type-check`, `lint`, `test:unit --run` y `build-only`, con las protecciones de `pnpm-workspace.yaml` intactas |
| Empaquetado | API: `composer install --no-dev --optimize-autoloader`. Web: compilada con `VITE_API_BASE_URL` y `VITE_PILOT_MODE` del ambiente |
| Staging | Con cada cambio en `main` que pase CI: sube el paquete y ejecuta `deploy/release.sh staging` |
| Producción | El mismo paquete, con aprobación manual (ambiente `production` de GitHub con revisor requerido) |
| `release.sh` | 1) verifica `APP_ENV` (`production` o `staging`) y `APP_DEBUG=false` en el `.env` compartido, o aborta; 2) descomprime en `releases/<fecha>` y enlaza `shared`; 3) `migrate --force`; 4) cachés de configuración, rutas y vistas; 5) cambia `current` y recarga PHP-FPM; 6) prueba `/up`, y si falla vuelve a la versión anterior; 7) conserva las últimas 5 versiones |
| Vuelta atrás | `deploy/rollback.sh <ambiente>`: vuelve `current` a la versión anterior. Regla: las migraciones deben ser compatibles con la versión anterior (añadir antes de quitar) |

## Copias (A-45)

- `spatie/laravel-backup` todos los días a las 3:00: vuelca PostgreSQL e incluye `storage/app/public`, en un zip cifrado con AES-256 (`BACKUP_ARCHIVE_PASSWORD`) y subido al disco `s3`.
- **Retención:** 30 copias diarias y 12 semanales (estrategia de limpieza del paquete). **Monitoreo:** si la última copia tiene más de 26 horas, se avisa.
- `APP_KEY` y `BACKUP_ARCHIVE_PASSWORD` **no** se guardan en las copias ni en el servidor de copias: van en tu gestor de contraseñas.
- **Restauración:** `deploy/restore.sh <archivo> <ambiente>` baja, descifra y restaura la base y los logos. Un procedimiento escrito (`deploy/RESTAURAR.md`) cubre el caso de un servidor nuevo desde cero (CE-001).
- **Prueba mensual:** `deploy/restore-test.sh` restaura la copia más reciente en una base temporal del servidor, verifica que la aplicación descifra un secreto SUNAT y cuenta comprobantes, e informa el resultado por correo.

## Monitoreo y avisos (A-48)

| Qué | Cómo |
|---|---|
| Web y API responden | Monitor externo de disponibilidad sobre `app.<dominio>` y `api.<dominio>/up`, cada 1 a 5 minutos |
| El *scheduler* corre | Tarea de cada minuto que hace *ping* a un monitor de latidos; si deja de llegar, el monitor avisa |
| La copia diaria terminó | *Ping* al monitor de latidos al terminar la copia; más el monitoreo de antigüedad del paquete |
| Pendientes SUNAT de más de 1 h, disco sobre 80 % | Comando `ops:check` cada 15 minutos; avisa por correo con un límite de un aviso por problema cada 6 horas |
| Errores 500 repetidos | En el manejador de excepciones: se registran con su detalle (sin secretos) y, si un mismo error se repite 5 veces en 10 minutos, se envía un correo; máximo uno por error cada hora |
| Canal | Correo a `OPS_ALERT_EMAIL` (el responsable del SaaS) |

## Aplicación

| Cambio | Dónde |
|---|---|
| Comando `ops:check` y sus avisos | API |
| Aviso de errores 500 repetidos, con límite de frecuencia | API (`bootstrap/app.php`) |
| Latido del *scheduler* y *ping* de copia | API (`routes/console.php`, configuración `OPS_HEARTBEAT_URL`, `OPS_BACKUP_PING_URL`) |
| Configuración de copias (`config/backup.php`) y disco `s3` | API |
| `.env.example` corregido (PostgreSQL, que hoy dice MySQL) y `deploy/env.<ambiente>.example` con todo lo necesario | API |
| Aviso de piloto (`VITE_PILOT_MODE`): franja en el Inicio y en el selector de clientes, «Piloto: no registres datos reales de tus clientes» | Frontend |
| CORS y enlaces: `FRONTEND_URL` y `CORS_ALLOWED_ORIGINS` por ambiente | `.env` del servidor |

## Estrategia de pruebas

**API (SQLite y PostgreSQL):**
- `Ops/OpsCheckTest`: avisa por pendientes de más de 1 h y por disco (simulado); no repite el aviso dentro de 6 h; no avisa si todo está bien.
- `Ops/ErrorAlertTest`: 5 errores iguales en 10 minutos → un correo; el sexto no repite; el correo no lleva secretos.
- `Ops/BackupConfigTest`: la copia incluye la base y los logos, está cifrada y usa la retención de A-45 (con un disco falso).
- `Ops/SchedulerTest`: el *schedule* incluye el latido, la copia, la limpieza, el monitoreo y `ops:check`, con sus frecuencias.

**Scripts:** `deploy/release.sh` y `rollback.sh` con un modo de ensayo (`--dry-run`) probado en CI sobre una carpeta temporal: aborta con `APP_ENV=local` o `APP_DEBUG=true`, vuelve atrás si `/up` falla y conserva las 5 últimas versiones.

**Frontend (Vitest):** franja de piloto visible solo con `VITE_PILOT_MODE`.

**Verificación en el servidor (juntos):** primer despliegue a staging y a producción, restauración completa en un servidor o base limpia cronometrada (CE-001), caída simulada de la API y del *scheduler* con los avisos (CE-002), correo de invitación recibido en una bandeja real y fuera de spam, y flujo del MVP completo por una empresa del piloto (CE-003).

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Perder `APP_KEY` o la clave de las copias | Guardadas aparte en tu gestor de contraseñas; el procedimiento de restauración lo exige como primer paso |
| Un despliegue rompe producción | CI obligatoria, staging antes, aprobación manual, prueba de `/up` con vuelta atrás automática y `rollback.sh` |
| Migración incompatible con la versión anterior | Regla de «añadir antes de quitar» documentada; se prueba en staging |
| Servidor único como punto de falla | Copias externas diarias y restauración de un servidor nuevo en menos de 1 h (CE-001); alta disponibilidad fuera de alcance |
| Correos en spam | Proveedor transaccional con SPF, DKIM y DMARC; se verifica con una bandeja real |
| Cuentas de prueba en el servidor | `DevelopmentSeeder` ya se niega fuera de `local`, y `release.sh` aborta si `APP_ENV` no es `production` o `staging` |
| Datos reales de clientes antes de la revisión legal | Aviso de piloto en la aplicación y acuerdo con las empresas (A-43) |

## Aprobación

Aprobado por el usuario el 2026-09-30, incluidas las dependencias `spatie/laravel-backup` y `league/flysystem-aws-s3-v3`, y el reparto de responsabilidades.
