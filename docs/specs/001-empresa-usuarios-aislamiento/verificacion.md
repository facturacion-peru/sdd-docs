# Verificación 001: Empresa, usuarios, roles y aislamiento

**Fecha:** 2026-09-28 · **Tarea:** T122 · **Rama:** `001-empresa-usuarios-aislamiento` (API y frontend)

Verificación contra la aplicación real (API en `:8000` y frontend de Vite en `:5173`, con las peticiones pasando por el proxy de Vite) y la base de desarrollo PostgreSQL, con datos de demostración ficticios.

## 1. Recorrido de extremo a extremo por HTTP

Script en el scratchpad de la sesión (`e2e.py`). Todas las peticiones pasan por el proxy de Vite (`http://127.0.0.1:5173/api/v1/...`), igual que las del navegador. Los enlaces de invitación y de recuperación se leen de `storage/logs/laravel.log` (`MAIL_MAILER=log`).

| # | Paso | Spec | Resultado |
|---|---|---|---|
| 1 | Login del administrador de la plataforma | HU-2.3 | ✅ 200 |
| 2 | Alta de empresa con establecimiento principal e invitación | HU-1.1 | ✅ 201 |
| 3 | Correo de invitación con enlace a `FRONTEND_URL/invitacion/{token}` | RF-010 | ✅ |
| 4 | La invitación muestra la empresa | HU-2.1 | ✅ 200 |
| 5 | Aceptar la invitación | HU-2.1 | ✅ 201, rol `company_admin` |
| 6 | Reutilizar la invitación | HU-2.2 | ✅ 410 |
| 7 | Invitar a un vendedor | HU-3.1 | ✅ 201 |
| 8 | El vendedor activa su cuenta | HU-2.1 | ✅ 201, rol `seller` |
| 9 | El vendedor pide usuarios y auditoría | HU-3.2 | ✅ 403 / 403 |
| 10 | El administrador de empresa pide rutas de plataforma | RF-021 | ✅ 403 |
| 11 | El único administrador intenta desactivarse | HU-3.3, RF-015 | ✅ 422 «La empresa debe tener al menos un administrador activo.» |
| 12 | Desactivar al vendedor corta su sesión abierta | HU-2.6, CE-004 | ✅ siguiente petición 401 |
| 13 | El vendedor desactivado intenta entrar | HU-2.6 | ✅ 422 «Tu cuenta o tu empresa están desactivadas.» |
| 14 | Recuperación con correo existente e inexistente | HU-2.7 | ✅ respuestas idénticas |
| 15 | Restablecer la contraseña y entrar con la nueva | HU-2.8 | ✅ 200 / 200 |
| 16 | La auditoría registra el recorrido | HU-6.1 | ✅ |
| 17 | Datos de la propia empresa | HU-5 | ✅ 200 |

**Resultado final: 17/17.** La primera ejecución encontró dos problemas, ya corregidos:

- **Paso 11 → 500 en PostgreSQL.** `FOR UPDATE` no se admite con `count()`. SQLite, donde corre la suite, no lo detectaba. Corregido en `UserService` y cubierto desde ahora con `composer test:pgsql`: la suite completa corre también contra PostgreSQL (`db_api_sunat_testing`) y pasa en verde.
- **Paso 15 → 429.** El límite de las rutas públicas (10 por minuto e IP) era demasiado estricto para una oficina que comparte IP pública. Subido a 30. El bloqueo por cuenta (5 fallos en 15 minutos) no cambia.

## 2. Pantallas públicas a 360 px

Capturas con Chrome headless (`--window-size=360,760`) contra el servidor de Vite.

| Pantalla | Resultado |
|---|---|
| `/login` | ✅ Legible, controles de 44 px de alto, enlace a recuperar contraseña |
| `/recuperar` | ✅ |
| `/invitacion/{token}` válido | ✅ Muestra la empresa y el rol leídos de la API por el proxy; correo de solo lectura; ayuda de contraseña visible |
| `/invitacion/{token}` inexistente | ✅ Mensaje de error claro y sin formulario |

## 3. Pantallas con sesión: pendiente de confirmación del usuario

Sin una herramienta de automatización del navegador (no se añadieron dependencias), estas pantallas las cubren las pruebas de componentes de Vitest, pero aún no se revisaron a la vista. **Falta confirmarlas a 360 px de ancho** (DevTools → modo dispositivo):

- [ ] Entrar como administrador de empresa: la cabecera muestra la empresa y el rol, y la navegación muestra Inicio, Mi empresa, Usuarios y Auditoría.
- [ ] Usuarios: invitar desde el diálogo (se cierra con Escape, el foco vuelve al botón), reenviar y cancelar una invitación, desactivar y reactivar a un usuario.
- [ ] Intentar quitarte el rol siendo el único administrador: aparece el mensaje de la API.
- [ ] Mi empresa: RUC y razón social de solo lectura; guardar el teléfono; subir un logo PNG con vista previa.
- [ ] Auditoría: filtrar por acción y por fechas; se ve bien en el celular.
- [ ] Entrar como vendedor: solo aparece «Inicio», y abrir `/usuarios` a mano lleva al inicio.
- [ ] Borrar el token en DevTools (Application → Local Storage) y navegar: lleva al login con `?redirect=` y, tras entrar, vuelve a la pantalla anterior.
- [ ] Entrar como administrador de la plataforma: aparece el aviso de usar el panel de la plataforma.

## Cuentas para la revisión

Desde el 2026-09-28 la base de desarrollo se regenera con `php artisan migrate:fresh --seed`: con `APP_ENV=local`, `DevelopmentSeeder` crea las cuentas de prueba listadas en el README de la API (contraseña `Clave-demo-123`). Para la sección 3:

- Administrador de empresa: `admin@demo.test`
- Vendedor: `vendedor@demo.test`
- Administrador de la plataforma: `plataforma@demo.test`
- Aislamiento: `admin@otra.test` (otra empresa)
