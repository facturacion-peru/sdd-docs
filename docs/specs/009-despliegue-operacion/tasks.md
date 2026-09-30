# Tareas 009: Despliegue y operación

**Estado:** Aprobado (2026-09-30) · **Fecha:** 2026-09-30 · **Plan:** [plan.md](./plan.md) (aprobado)

> Mismas convenciones que las specs anteriores: tareas pequeñas con su verificación; `[P]` = en paralelo con la anterior; 🔒 = confirmación del usuario al ejecutarla.
> 👤 = **la hace el usuario** (cuentas, pagos, accesos y secretos); el agente deja preparada la guía. 🤝 = la hacen juntos: el usuario da el acceso y el agente ejecuta y documenta.
> ⏸ = **en espera**: el despliegue al servidor se pospuso (A-46, actualización del 2026-09-30).
> **Pruebas primero** en las tareas de código. Antes de cada commit se verifica con el **código de salida** de las suites.

## Fase 1 · Preparación (sin servidor)

- [ ] **T001** Crear la rama `009-despliegue-operacion` desde `main` en la API y en el frontend. — Verificación: rama en ambos.
- [ ] **T003** Auditoría de secretos antes de publicar (A-46): historial completo de los tres repositorios (claves privadas, certificados, contraseñas, `.env`, tokens, correos de terceros). Si aparece algo, se decide con el usuario cómo limpiarlo antes de subir. — Verificación: informe de la auditoría; nada sensible en lo que se publica.
- [ ] **T002** Guía `deploy/PREPARACION.md` para el usuario: qué contratar y crear (VPS, dominio, DNS, repositorios, almacenamiento S3, correo, monitores), qué registros DNS poner (A, SPF, DKIM, DMARC), qué secretos generar y dónde guardarlos, y la lista de secretos de GitHub Actions. — Verificación: guía revisada por el usuario.

## Fase 2 · Cambios en la aplicación

- [ ] **T010** `.env.example` corregido (PostgreSQL) y `deploy/env.production.example` y `deploy/env.staging.example` con todas las variables (correo, S3, copias, avisos, CORS, `FRONTEND_URL`), sin valores secretos. — Verificación: `php artisan config:show` sin errores con cada plantilla; revisión de que no hay secretos.
- [ ] **T011** Instalar `spatie/laravel-backup` y `league/flysystem-aws-s3-v3` (con `composer update --lock --ignore-platform-req=php` si hace falta), `config/backup.php` (base y logos, zip cifrado, disco `s3`, retención 30/12, antigüedad máxima 26 h) y el disco `s3` en `config/filesystems.php`. — Verificación: `composer validate`; suites en verde.
- [ ] **T012** Escribir `Ops/BackupConfigTest` (con un disco falso): la copia incluye la base y los logos, está cifrada, y la limpieza deja 30 diarias y 12 semanales. — Verificación: en verde.
- [ ] **T013** Escribir pruebas `Ops/OpsCheckTest` (pendientes de más de 1 h, disco sobre 80 % simulado, sin repetir en 6 h, sin aviso si todo va bien) e implementar el comando `ops:check` con su notificación por correo. — Verificación: en verde en ambas bases.
- [ ] **T014** Escribir pruebas `Ops/ErrorAlertTest` (5 errores iguales en 10 min → un correo; el siguiente no repite; sin secretos) e implementar el aviso en el manejador de excepciones. — Verificación: en verde.
- [ ] **T015** *Schedule*: latido cada minuto (`OPS_HEARTBEAT_URL`), copia diaria con *ping* al terminar (`OPS_BACKUP_PING_URL`), limpieza y monitoreo de copias, `ops:check` cada 15 min. `Ops/SchedulerTest` comprueba tareas y frecuencias. — Verificación: en verde; `php artisan schedule:list`.
- [ ] **T016** [P] Frontend: franja de piloto con `VITE_PILOT_MODE` en el Inicio y en el selector de clientes. Pruebas incluidas. — Verificación: pruebas en verde.

## Fase 3 · Scripts del servidor y CI

- [ ] **T020** `deploy/provision.sh` (idempotente): usuario `deploy`, SSH solo con llave, `ufw`, `fail2ban`, `unattended-upgrades`, PHP 8.5-FPM con extensiones y OPcache, PostgreSQL con bases y usuarios por ambiente, Nginx con las plantillas de `deploy/nginx/`, Certbot, estructura `/srv/sunat/<ambiente>` y cron del *scheduler*. — Verificación: `shellcheck` sin errores; ejecución en un contenedor Ubuntu 24.04 local hasta donde no requiere DNS.
- [ ] **T021** `deploy/release.sh`, `deploy/rollback.sh` con `--dry-run`: guardia de `APP_ENV`/`APP_DEBUG`, versión nueva, enlaces compartidos, migraciones, cachés, cambio de `current`, prueba de `/up` con vuelta atrás y conservación de 5 versiones. Pruebas del modo ensayo sobre una carpeta temporal (`deploy/tests/`). — Verificación: `shellcheck`; pruebas del ensayo en verde (incluido el aborto con `APP_ENV=local`).
- [ ] **T022** `deploy/restore.sh`, `deploy/restore-test.sh` y `deploy/RESTAURAR.md` (restauración de un servidor nuevo desde cero, con `APP_KEY` como primer paso). — Verificación: `shellcheck`; restauración probada en local desde una copia de la base de desarrollo.
- [ ] **T023** Workflows de GitHub Actions: CI de la API (SQLite, PostgreSQL como servicio, Pint, pruebas de los scripts) y del frontend (pnpm con el lockfile congelado y las protecciones intactas); empaquetado; despliegue a staging en cada cambio de `main` y a producción con aprobación manual (ambiente `production`). — Verificación: `actionlint` sin errores; los jobs de CI corren en verde en cuanto existan los repositorios (T031).

## Fase 4 · Infraestructura (usuario)

- [ ] ⏸ **T030** 👤 Contratar el VPS (Ubuntu 24.04, región cercana al Perú) y el dominio; crear los registros DNS de los cuatro subdominios. — Verificación: los subdominios resuelven a la IP del VPS.
- [ ] **T031** 👤 Crear los repositorios **públicos** (API, frontend y documentación; A-46 actualizada) y subir el código; cambiar el `origin` de la API (hoy apunta al autor original). — Verificación: los workflows de CI corren en GitHub.
- [ ] ⏸ **T032** 👤 Crear el almacenamiento S3, la cuenta del proveedor de correo (con SPF, DKIM y DMARC en el DNS) y los monitores (disponibilidad y latidos); generar sus claves. — Verificación: dominio verificado en el proveedor de correo; URLs de los monitores anotadas.
- [ ] ⏸ **T033** 👤 Generar `APP_KEY` y `BACKUP_ARCHIVE_PASSWORD` de cada ambiente, guardarlas en el gestor de contraseñas y cargar los secretos en GitHub Actions y en los `.env` del servidor según `PREPARACION.md`. — Verificación: lista de secretos de la guía completa.

## Fase 5 · Primer despliegue y verificación (juntos)

- [ ] ⏸ **T040** 🤝 Ejecutar `provision.sh` en el VPS y emitir los certificados HTTPS. — Verificación: los cuatro subdominios responden con HTTPS; `ufw status`; SSH con contraseña rechazado.
- [ ] ⏸ **T041** 🤝 Primer despliegue a staging por CI, `migrate --force` y alta de la primera empresa con `platform:create-admin` y el panel. — Verificación: `/up` en verde; la web de staging entra con la cuenta de plataforma.
- [ ] ⏸ **T042** 🤝 Verificación de staging: correo de invitación recibido fuera de spam, boleta y nota de crédito aceptadas por SUNAT beta, *scheduler* corriendo, franja de piloto visible. — Verificación: resultado por paso en `verificacion.md`.
- [ ] ⏸ **T043** 🤝 Copia real a S3 y restauración completa en una base limpia, cronometrada (CE-001), más `restore-test.sh`. — Verificación: tiempo y resultado en `verificacion.md`.
- [ ] ⏸ **T044** 🤝 Caída simulada: detener PHP-FPM y el cron del *scheduler*, y crear un pendiente de más de 1 h → avisos por correo en menos de 5 minutos (CE-002). — Verificación: hora de cada aviso en `verificacion.md`.
- [ ] ⏸ **T045** 🔒 Primer despliegue a producción del piloto con aprobación manual. — Verificación: `/up` en verde; cuenta de plataforma creada; sin cuentas de prueba (`User::count()` solo con la de plataforma).
- [ ] ⏸ **T046** 🤝 Una empresa del piloto completa el flujo del MVP en producción (CE-003). — Verificación: resultado en `verificacion.md`.

## Fase 6 · Cierre

- [ ] **T050** API y frontend: suites en verde (códigos de salida) sobre la rama. — Verificación: salidas adjuntas.
- [ ] **T051** Documentación: README de la API y del frontend (operación, despliegue, copias, avisos), `AGENTS.md` y estado de la spec en `specs/README.md`. — Verificación: enlaces válidos.
- [ ] **T052** 🔒 Merge local de `009-despliegue-operacion` a `main` en ambos repositorios y commit de la raíz. — Verificación: suites en verde sobre `main`.

## Trazabilidad

| Spec | Tareas |
|---|---|
| HU-1, RF-001–002, RF-007 | T010, T020–T021, T023, T040–T041, T045 |
| HU-2, RF-003, CE-001 | T011–T012, T015, T022, T043 |
| HU-3, RF-004–005 | T015, T032, T042 |
| HU-4, RF-006, RF-008, CE-002 | T013–T015, T044 |
| HU-5 | T021, T023 |
| RF-009 (A-43) | T016, T042 |
| CE-003 | T046 |
