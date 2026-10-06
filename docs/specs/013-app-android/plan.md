# Plan 013: Aplicación Android

**Estado:** Aprobado (2026-10-06) · **Fecha:** 2026-10-06 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

> Cómo. Solo se redacta cuando la spec está **Aprobada**.

## Resumen técnico

El frontend Vue se empaqueta con **Capacitor 8** en un proyecto Android dentro del repositorio del frontend (`android/`). La interfaz viaja dentro del APK (A-67) y habla con la API por HTTPS (A-62). Todo lo que depende del dispositivo pasa por un módulo nuevo y aislado, `core/device/`, con una implementación web (la de hoy) y otra nativa. Así la web no cambia (CE-004) y el resto del código no sabe si corre en Android.

Hay dos cambios pequeños en la API. El primero es el **PDF de 80 mm del ticket**, porque en la app no hay impresión del navegador y los formatos de impresión pertenecen a la API (principio II). El segundo es el **control de versión mínima** de la app con la cabecera `X-App-Version` (A-67).

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | Frontend (principal) y API (PDF del ticket, versión mínima, CORS) |
| Rama | `013-app-android` en el frontend y en la API, desde `main` |
| Nombre e identificador | **Provisionales** (A-61): nombre «Facturación SUNAT» e identificador `io.github.facturacion_peru.app` (deriva de la organización de GitHub; los guiones no se admiten en un identificador de Android). Se cambian antes de la primera entrega a una empresa; mientras nadie lo haya instalado, cambiarlo no tiene costo |
| Android mínimo | El de Capacitor 8 (`minSdk` 24, Android 7.0); se apunta al SDK que pida Capacitor 8 (`targetSdk` 36) |
| Herramientas | SDK en `~/Android/sdk` (ya existe). Capacitor 8 pide **JDK 21**; en `~/Android` hay JDK 17, así que se descarga JDK 21 (Temurin) junto a él, sin tocar el Java del sistema. Se compila con Gradle desde la consola (`./gradlew`); Android Studio no hace falta |
| Migraciones | Ninguna |

### Dependencias nuevas (frontend)

Todas se instalan con `pnpm add`, respetan `minimumReleaseAge` y las demás protecciones, y se revisan antes de aprobarlas si piden scripts de instalación.

| Paquete | Para qué | Por qué este |
|---|---|---|
| `@capacitor/core`, `@capacitor/android`, `@capacitor/cli` (dev) | Empaquetar la app y su puente con Android | El que prevé la constitución (principio VII); mantenido por Ionic, mismo código web |
| `@capacitor/app` | Botón «Atrás», versión instalada (`getInfo`) y volver del segundo plano | Oficial |
| `@capacitor/filesystem` y `@capacitor/share` | Guardar el PDF, XML o CDR en la caché de la app y abrir «Compartir / Abrir con» (A-63) | Oficiales; es el camino estándar en Android (`FileProvider`) |
| `@capacitor/network` (corrección del 2026-10-06, T022) | Saber si hay conexión en la app | Oficial. El WebView de Android **no** actualiza `navigator.onLine` por sí solo: lo debe avisar la app, y Capacitor no lo hace. Se descubrió en el emulador (modo avión con `navigator.onLine` todavía en `true`) |
| `@aparajita/capacitor-secure-storage` | Token en el almacén de claves de Android (A-64) | Capacitor no tiene almacenamiento seguro oficial (el de Ionic es de pago). Este cifra con Android Keystore, va al día con Capacitor 8 y tiene licencia MIT. Alternativa revisada: `capacitor-secure-storage-plugin` (menos activo). Se lee su código nativo antes de adoptarlo |

**No se agregan** plugins de barras del sistema. La conexión se detecta con `navigator.onLine` en la web y con `@capacitor/network` en la app (ver la corrección arriba), ambos detrás de `core/device`. Los márgenes de la muesca y de las barras se resuelven con `env(safe-area-inset-*)`, porque Capacitor 8 dibuja de borde a borde. Los íconos se escriben a mano como *vector drawable* adaptativo a partir del SVG del logo, así no hace falta un generador con dependencias nativas.

## Chequeo de constitución

| Principio | Cumple | Cómo / justificación de la excepción |
|---|---|---|
| I. Spec primero | Sí | Spec 013 aprobada con A-61 a A-67 |
| II. Corrección fiscal | Sí | La app no calcula nada nuevo. El PDF del ticket lo genera la API, igual que los de comprobantes |
| III. Contrato de API | Sí | `GET /tickets/{ticket}/pdf` va al OpenAPI y a los tipos. La cabecera `X-App-Version` y la respuesta 426 se documentan en la spec y en `types.ts` |
| IV. Seguridad | Sí | Token en Keystore y fuera de las copias (`allowBackup=false` y reglas de extracción). La versión que se entrega solo admite HTTPS; el texto plano se permite solo en la variante *debug*. La clave de firma vive fuera de los repositorios. CORS admite el origen de la app de forma explícita. Las dependencias se justifican arriba |
| V. Simplicidad | Sí | Un solo código y un solo módulo `core/device/`. Sin plugins de red ni de barras. Sin panel de plataforma ni modo sin conexión |
| VI. Verificable | Sí | Vitest para `core/device` (las dos implementaciones con dobles), Pest para el PDF del ticket y la versión mínima, y verificación manual en un Android real (CE-001 a CE-003) |
| VII. Personas reales | Sí | El módulo de dispositivo queda aislado y es sustituible. Botón «Atrás», aviso sin conexión y la venta en curso se conserva |
| VIII. Aislamiento y auditoría | Sí | El PDF del ticket usa la misma política que `GET /tickets/{ticket}`: otra empresa recibe 404 y el vendedor solo ve sus tickets (A-30). Se registra en `TenantRoutes.php` |

## Contrato de API

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/api/v1/tickets/{ticket}/pdf` | Admin y vendedor (solo sus tickets) | PDF de 80 mm del ticket, con el mismo contenido y la misma leyenda «no es comprobante de pago» que `TicketReceipt`. Usa dompdf, como `DocumentPdf` |
| — | Cabecera `X-App-Version` en toda petición | — | La envía solo la app. Un *middleware* la compara con `APP_ANDROID_MIN_VERSION` (`.env`; vacía = sin control). Si es menor, responde **426** con `{ success: false, message, min_version }`. Sin la cabecera (web), no hace nada |

CORS: `allowed_headers` suma `X-App-Version`. El origen de la app (`https://localhost`, el esquema de Capacitor en Android) se agrega a `CORS_ALLOWED_ORIGINS` en el `.env` del servidor y del entorno local. No se usa «*».

## Frontend

### Módulo `core/device/` (principio VII)

Una sola interfaz, elegida al arrancar según `Capacitor.isNativePlatform()`:

| Función | Web (hoy) | Android |
|---|---|---|
| `secureStore` (get, set, remove) | `localStorage` | Secure Storage (Keystore) |
| `deliverFile(blob, filename, { open })` | Enlace con *blob*, como hoy en `download.ts` | Escribe en la caché con Filesystem y abre «Compartir» |
| `onBackButton(handler)` | No hace nada | `App.addListener('backButton')` |
| `onConnectionChange(handler)` | `navigator.onLine` y eventos `online`/`offline` | `Network.getStatus()` y `networkStatusChange` |
| `appVersion()` | `null` | `App.getInfo().version` |
| `isNative` | `false` | `true` |

- **Token** (`core/api/token-storage.ts`): la API pública sigue síncrona porque la usa el cliente HTTP, pero ahora lee de una caché en memoria. `initTokenStorage()` la carga desde `secureStore` en `main.ts` antes de montar la app. `setToken` y `clearToken` escriben en la caché y en `secureStore`.
- **Descargas** (`core/api/download.ts`): cambia solo la entrega final, que pasa por `deliverFile`.
- **Imprimir en la app:** `TicketView` y `SaleDoneDialog` piden el PDF de 80 mm del ticket a la API y lo comparten, igual que ya se hace con los comprobantes. En la web sigue `window.print()` con `TicketReceipt`.
- **Cliente HTTP:** cuando `appVersion()` existe, se agrega `X-App-Version`. Un 426 lleva a una pantalla «Actualiza la app» con la versión mínima (HU-5), sin borrar la sesión ni la venta en curso.
- **Botón «Atrás»** (`app/`): primero cierra el `BaseDialog` abierto más reciente, con una pila pequeña en `shared/ui` que ya cubre las hojas del celular. Si no hay ninguno y el router tiene historial, ejecuta `router.back()`. En el Inicio o en el login, `App.minimizeApp()`: la app se va al fondo, como se espera en Android, y la venta en curso queda guardada.
- **Sin conexión** (`app/layouts/AppLayout`): un aviso fijo según `device().onConnectionChange` (`navigator.onLine` y sus eventos en la web; `@capacitor/network` en la app). El cobro ya falla con el mensaje de conexión y conserva la venta y la clave de idempotencia (spec 012).
- **Plataforma fuera de la app (RF-003):** si `isNative` y la sesión es de plataforma, se cierra la sesión y el login muestra «El panel de la plataforma se usa desde la web». Las rutas `/plataforma` también quedan bloqueadas en la app.
- **Márgenes de borde a borde:** `env(safe-area-inset-*)` en `AppLayout`, la barra fija de «Vender» y las hojas de `BaseDialog`.
- **Teclado:** el ajuste del WebView (`adjustResize`) mantiene visibles el campo y el botón de cobrar. Se verifica en el teléfono.

### Proyecto Android y compilación

- `capacitor.config.ts`: `appId`, `appName` y `webDir: 'dist'`. No usa `server.url` (la interfaz va dentro, A-67).
- Modos de Vite:
  - `.env.android` con `VITE_API_BASE_URL` del servidor HTTPS de la 009 (vacío hasta que exista).
  - `.env.android-dev` con `http://<IP de la PC>:8000`, que no se versiona.
  - La API se sirve con `php artisan serve --host=0.0.0.0` para la red local.
- **Texto plano solo en debug:** `network_security_config` en `android/app/src/debug/res/xml/`; la variante *release* no lo tiene (RF-007).
- **Copias de seguridad:** `android:allowBackup="false"` y `dataExtractionRules` que excluyen todo (RF-002).
- **Firma:** un *keystore* en `~/Android/keys/` (fuera de los repositorios). Gradle lo lee de `android/keystore.properties`, que está en `.gitignore`. Se documenta que perder la clave impide actualizar la app instalada.
- **Versión:** `versionName` toma la `version` de `package.json` (hoy `0.0.0`; arranca en `0.1.0`). `versionCode` sube en cada entrega.
- Scripts `pnpm`:
  - `android:dev`: compila en modo `android-dev`, sincroniza e instala el APK de debug con `adb`.
  - `android:release`: compila en modo `android`, sincroniza y genera el APK firmado con `assembleRelease`.
- **CI:** el CI actual sigue igual. Compilar el APK en GitHub Actions queda fuera de esta spec, porque la firma necesita secretos.

## Estrategia de pruebas

| Requisito / escenario | Prueba | Tipo |
|---|---|---|
| RF-001, CE-004 | Las suites actuales sin cambios de comportamiento; `core/device` en web se comporta como hoy | Unitaria |
| RF-002, HU-2 | `token-storage.spec.ts`: `initTokenStorage` carga de `secureStore`; `setToken` y `clearToken` escriben en él; con la implementación nativa no hay nada en `localStorage` | Unitaria |
| RF-003, HU-1 esc. 4 | `LoginView`/`router` en modo nativo simulado: la cuenta de plataforma ve el aviso y no entra a `/plataforma` | Unitaria |
| RF-004, HU-3 esc. 1–2 | `download.spec.ts` y `SaleDoneDialog`/`TicketView` en modo nativo: piden el PDF y llaman a `deliverFile`, sin `window.print` | Unitaria |
| RF-005, HU-3 esc. 3 | `back-button.spec.ts`: cierra el diálogo superior, si no vuelve, y en el Inicio minimiza | Unitaria |
| RF-006, HU-4 | `AppLayout.spec.ts`: el evento `offline` muestra el aviso y `online` lo quita | Unitaria |
| RF-008, HU-5 | API: `AppVersionTest` (sin cabecera pasa, menor da 426, igual o mayor pasa, sin mínimo configurado pasa). Frontend: un 426 muestra «Actualiza la app» sin borrar la sesión | Pest y unitaria |
| PDF del ticket | `TicketPdfTest`: contenido y leyenda; vendedor ajeno 403/404; otra empresa 404 (más `TenantRoutes.php`) | Pest (SQLite y PostgreSQL) |
| RF-007 | Inspección del APK de release (`aapt dump xmltree` del manifiesto y de la configuración de red): sin texto plano y `allowBackup=false` | Manual documentada |
| CE-001, CE-002, CE-003 | En un Android de gama baja real: instalar, entrar, ticket y boleta (SUNAT beta), compartir el PDF de 80 mm, «Atrás», sin conexión, cerrar sesión; tiempo de apertura; `adb backup` y el almacenamiento web sin token | Manual documentada en `verificacion.md` |

## Riesgos

| Riesgo | Mitigación |
|---|---|
| La interfaz dentro del APK queda vieja frente a la API | Versión mínima con 426 (A-67); los cambios incompatibles de la API ya exigen versión nueva (principio III) |
| Plugin de almacenamiento seguro de un tercero | Revisión del código nativo y versión exacta en el *lockfile*. Queda aislado en `core/device`, así que cambiarlo no toca el resto |
| Perder la clave de firma | *Keystore* fuera de los repositorios con copia en `respaldos/` y fuera de la máquina. Se documenta en el README |
| Probar sin servidor HTTPS | Variante *debug* por la red local (A-62); la entrega real espera a la 009 |
| Teléfonos de gama baja lentos | Medir CE-002 en un teléfono real; el *bundle* ya está dividido por rutas |
| El WebView de Android no imprime HTML | Imprimir siempre por el PDF de la API (A-63), que ya funciona para comprobantes |
| Identificador provisional | Cambiarlo es una tarea antes de la primera entrega (A-61); ningún usuario lo habrá instalado |
