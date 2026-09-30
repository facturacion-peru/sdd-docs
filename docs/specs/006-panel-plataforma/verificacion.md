# Verificación 006: Panel de administración de la plataforma

**Fecha:** 2026-09-29 · **Tarea:** T072 · **Rama:** `006-panel-plataforma`

## Suites (T070, T071)

| Proyecto | Resultado |
|---|---|
| API, SQLite | ✅ 555 pruebas |
| API, PostgreSQL | ✅ 555 pruebas |
| API, Pint | ✅ |
| Frontend | ✅ `type-check`, `lint`, 152 pruebas y `build-only` |

Pruebas con mutación:

| Protección | Mutación | Resultado |
|---|---|---|
| Rol de plataforma en todas las rutas `platform/*` | Quitar `platform.admin` de una ruta | ❌ un usuario de empresa obtiene 200 |
| Recursos de plataforma con lista blanca | Añadir el total al recurso de soporte | ❌ la prueba encuentra el importe en `/platform/sales-documents` |

## Recorrido de extremo a extremo (por el proxy de Vite)

Sobre una base recién sembrada. El comprobante pendiente se creó con una segunda instancia de la API (puerto 8001) con SUNAT inalcanzable. 19/19 pasos correctos.

| # | Paso | Spec | Resultado |
|---|---|---|---|
| 1 | Lista con estado SUNAT y contadores | HU-1.1 | ✅ |
| 2 | Filtro «con problemas de emisión» | HU-1.2, CE-003 | ✅ una consulta de 41 ms |
| 3 | Alta de empresa con su administrador | HU-2.1 | ✅ 174 ms de API |
| 4 | RUC repetido | HU-2.2 | ✅ 422 |
| 5 | Reenvío de la invitación | HU-2.3 | ✅ enlace nuevo en el correo |
| 6 | El administrador acepta y entra | HU-2.1 | ✅ |
| 7 | La ficha muestra la invitación aceptada | HU-1.3 | ✅ |
| 8 | Corrección del domicilio fiscal | HU-3 | ✅ |
| 9 | Cambio de RUC → SUNAT pendiente con el motivo | HU-3.2 | ✅ |
| 10 | Validar con el certificado del RUC anterior | HU-3.2 | ✅ error explicado |
| 11 | Con el RUC original, validación real contra SUNAT beta | HU-3.2 | ✅ validada |
| 12 | Suspender sin motivo | HU-4.1 | ✅ 422 |
| 13 | Suspendida: su administrador pierde la sesión y no entra | HU-4.1 | ✅ 401 y login rechazado |
| 14 | Reactivada: vuelve a entrar | HU-4.2 | ✅ |
| 15 | Soporte: el pendiente con su empresa, sin cliente ni importes | HU-5.1 | ✅ |
| 16 | Reintento desde la plataforma, aceptado por SUNAT beta | HU-5.2 | ✅ |
| 17 | Un usuario de empresa no entra al panel | RF-001 | ✅ 403 |
| 18 | Auditoría de la plataforma | HU-6 | ✅ inicio de sesión, alta, invitación, correcciones, suspensión, reactivación y reintento |
| 19 | Búsqueda de secretos y datos de negocio en 16 respuestas del panel | CE-002 | ✅ sin coincidencias |

## Pantallas a 360 px

| Pantalla | Resultado |
|---|---|
| Empresas | ✅ Búsqueda, filtros, estado SUNAT e indicadores |
| Ficha | ✅ Datos legales, estado, administrador con invitación y suspensión |
| Nueva empresa | ✅ Formulario con buscador de distrito |
| Auditoría | ✅ Acciones con empresa, autor e IP |

## Hallazgos corregidos

- **El administrador de la plataforma perdía la sesión al recargar.** `/auth/me` estaba en el grupo de empresa y le respondía 403. Venía de la 001 y no se notaba, porque tras el login la sesión sale de la respuesta del login. Ahora `/auth/me` deja pasar a la plataforma, conservando las comprobaciones de cuenta y empresa activas.
- **Un usuario de empresa podía saber si un id de empresa existía** (404 frente a 403 en las rutas de plataforma): el *binding* se resolvía antes que el rol. Ahora el rol se comprueba primero. Lo detectó `PlatformAccessTest` en PostgreSQL.
- **OpenAPI:** una regla `array` con claves con nombre (`fiscal_address.address`) se documentaba como lista. Se corrigió el generador, con prueba.
- La auditoría mostraba `sales_document.status_changed` sin traducir.

## Pendiente de confirmación del usuario

- **CE-001:** cronometrar el alta de una empresa desde el panel (objetivo: menos de 3 minutos). La API responde en 174 ms; falta el tiempo de una persona llenando el formulario.
- **Antes de producción** (A-24, A-38): segundo factor para los administradores de la plataforma.
- **Datos de prueba:** el recorrido dejó la empresa «Librería Delta S.A.C.» y un comprobante en la base local. Se limpian con `php artisan migrate:fresh --seed`.
