# Spec 009: Despliegue y operación

**Estado:** Borrador para aprobación · **Versión:** 0.2 · **Fecha:** 2026-09-30
**Depende de:** 001 a 007 · **Aclaraciones:** A-24 ⚖️, A-43, A-44, A-45, A-46, A-47, A-48

## Objetivo

Que el SaaS funcione **fuera de la máquina de desarrollo**, de forma segura y recuperable: la API y la aplicación web publicadas con HTTPS, la base de datos con copias de seguridad que se sabe restaurar, los reintentos a SUNAT corriendo solos, correo real para invitaciones y recuperación de contraseña, y avisos cuando algo falla. Sigue siendo el **ambiente beta de SUNAT**: el paso a producción es la spec 008.

Hoy todo corre en local: `php artisan serve`, `pnpm dev`, correo al log, sin copias de seguridad ni cron. El remoto de la API apunta al repositorio del autor original y el frontend no tiene remoto.

**Alcance del primer despliegue (A-43):** un **piloto con empresas reales sin datos reales de sus clientes**; emiten a «Cliente varios» o con datos ficticios hasta cerrar la revisión legal (A-24 ⚖️).

## Historias de usuario

### HU-1 · Publicar la aplicación (P1)

Como responsable del SaaS, quiero la API y la aplicación web publicadas en un servidor con HTTPS, para que las empresas del piloto la usen desde su celular (A-43) (A-44).

**Escenarios de aceptación**

1. **Dado** el servidor preparado, **cuando** se despliega una versión, **entonces** la aplicación web responde en su dominio con HTTPS y la API en el suyo, sin modo de depuración ni cuentas de prueba.
2. **Dado** un despliegue nuevo, **cuando** tiene migraciones, **entonces** se aplican sin perder datos y sin pedir confirmación interactiva.
3. **Dado** un despliegue que falla a mitad, **cuando** ocurre, **entonces** la versión anterior sigue funcionando.

### HU-2 · Copias de seguridad y restauración (P1)

Como responsable del SaaS, quiero copias automáticas de la base de datos y de los archivos, y saber restaurarlas, para no perder comprobantes ni configuraciones (A-45).

**Escenarios de aceptación**

1. **Dado** el servidor en marcha, **cuando** pasa un día, **entonces** hay una copia nueva, cifrada y fuera del servidor, de la base de datos y de los logos.
2. **Dado** una copia, **cuando** se restaura en un servidor limpio con la misma `APP_KEY`, **entonces** la aplicación vuelve a funcionar, incluidos los secretos cifrados de SUNAT.
3. **Dado** que una copia falla, **cuando** ocurre, **entonces** llega un aviso (A-48).

### HU-3 · Tareas programadas y correo (P1)

Como empresa, quiero que los comprobantes pendientes se reenvíen solos y que las invitaciones y recuperaciones de contraseña lleguen a mi correo (A-47).

**Escenarios de aceptación**

1. **Dado** el servidor en marcha, **cuando** un comprobante queda pendiente, **entonces** el *scheduler* lo reenvía como en la 005.
2. **Dado** una invitación o una recuperación de contraseña, **cuando** se envía, **entonces** llega al correo del destinatario desde el dominio del SaaS y no a la carpeta de spam.

### HU-4 · Avisos cuando algo falla (P1)

Como responsable del SaaS, quiero enterarme de las fallas antes que las empresas (A-48).

**Escenarios de aceptación**

1. **Dado** que la API o la web dejan de responder, **cuando** pasa un par de minutos, **entonces** recibo un aviso.
2. **Dado** que el *scheduler* deja de correr, o que hay comprobantes pendientes hace más de una hora, **cuando** ocurre, **entonces** recibo un aviso.
3. **Dado** errores 500 en la API, **cuando** ocurren, **entonces** quedan registrados con su detalle (sin secretos) y recibo un aviso si se repiten.

### HU-5 · Desplegar sin miedo (P2)

Como desarrollador, quiero que cada versión pase las pruebas antes de llegar al servidor, para no romper la aplicación de las empresas (A-46).

## Casos límite

- Se pierde `APP_KEY`: las copias no sirven para los secretos de SUNAT. `APP_KEY` se guarda aparte de las copias, y su rotación sigue el README (`APP_PREVIOUS_KEYS`).
- El disco se llena: aviso antes de que falle la base de datos.
- El certificado HTTPS vence: se renueva solo, y hay aviso si no.
- Un despliegue deja la base con una migración a medias: la migración se revisa en el ambiente de pruebas antes (HU-5).
- `APP_ENV` queda en `local` por error en el servidor: el seeder crearía cuentas con contraseña pública; el despliegue lo impide.

## Requisitos funcionales

- **RF-001:** La aplicación DEBE publicarse con HTTPS, `APP_ENV=production` y `APP_DEBUG=false`, y sin cuentas de prueba.
- **RF-002:** Los secretos (`APP_KEY`, claves de base de datos y de correo) NO DEBEN estar en el repositorio.
- **RF-003:** DEBE haber copia diaria cifrada de la base de datos y de los archivos subidos, en un almacenamiento externo, con 30 copias diarias y 12 semanales, `APP_KEY` guardada aparte y una restauración de prueba mensual (A-45).
- **RF-004:** El *scheduler* DEBE correr cada minuto en el servidor.
- **RF-005:** El correo DEBE enviarse con un proveedor transaccional y el dominio autenticado con SPF, DKIM y DMARC (A-47).
- **RF-006:** DEBE haber avisos **por correo** al responsable del SaaS por las fallas de HU-4 (A-48).
- **RF-007:** El código DEBE vivir en repositorios privados propios; cada cambio DEBE pasar las suites en CI, y el despliegue a `main` DEBE hacerse sin tiempo de caída, aplicar las migraciones sin confirmación interactiva y probarse antes en un ambiente de staging (A-46).
- **RF-008:** Los logs NO DEBEN contener secretos y DEBEN rotarse.
- **RF-009:** El primer despliegue DEBE ser un piloto sin datos reales de clientes (A-43): la aplicación muestra a las empresas del piloto un aviso de que no registren datos personales reales de sus clientes hasta nuevo aviso.

## Criterios de éxito

- **CE-001:** Una restauración completa desde la copia más reciente, en un servidor limpio, deja la aplicación funcionando en menos de 1 hora, con un procedimiento escrito.
- **CE-002:** Una caída simulada de la API y del *scheduler* genera avisos en menos de 5 minutos.
- **CE-003:** Una empresa del piloto completa el flujo del MVP (alta, productos, ticket, boleta en beta) en el servidor publicado.

## Fuera de alcance

- Ambiente de producción SUNAT (spec 008).
- Alta disponibilidad, varios servidores o balanceo de carga (volumen de A-06).
- APK en tiendas de aplicaciones.
- Cobros del SaaS.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-09-30 | Spec creada: despliegue y operación, independiente de la revisión legal salvo por el alcance del piloto (A-43) | — |
| 2026-09-30 | A-43 | Piloto con empresas reales, sin datos reales de clientes hasta A-24 |
| 2026-09-30 | A-44 | Un VPS propio: Ubuntu, Nginx, PHP 8.5-FPM, PostgreSQL local, HTTPS |
| 2026-09-30 | A-45 | Copias diarias cifradas fuera del servidor; 30 diarias y 12 semanales; `APP_KEY` aparte |
| 2026-09-30 | A-46 | Repos privados propios, GitHub Actions y staging |
| 2026-09-30 | A-47 | Proveedor de correo transaccional con dominio autenticado |
| 2026-09-30 | A-48 | Monitoreo recomendado con avisos por correo |
