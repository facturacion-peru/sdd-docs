# Verificación 013: Aplicación Android

## Preparación (T002, T003)

### Herramientas (T002, 2026-10-06)

- Capacitor 8.5 (`@capacitor/android`): `compileSdk` y `targetSdk` 36, `minSdk` 24 (Android 7.0), Java 21 y Android Gradle Plugin 8.13.
- JDK 21: Temurin 21.0.12.1+1 en `~/Android/jdk-21.0.12.1+1`, descargado de la API de Adoptium y verificado con su SHA-256. El JDK 17 y el Java del sistema no se tocaron.
- SDK en `~/Android/sdk`: `platforms;android-36`, `build-tools;36.0.0`, `platform-tools` y `cmdline-tools` ya estaban instalados. No faltaba nada.

### Revisión de dependencias (T003, 2026-10-06)

**`@aparajita/capacitor-secure-storage@8.0.1`** (MIT, <https://github.com/aparajita/capacitor-secure-storage>; publicada el 2026-09-23):

- **Código nativo de Android:** `SecureStorage.java` y `KeyStoreException.java`, unas 440 líneas.
  - Cifra cada valor con AES/GCM/NoPadding, usando una clave generada dentro de Android Keystore que no sale del dispositivo. El IV es aleatorio en cada escritura.
  - El texto cifrado se guarda en `SharedPreferences` privadas de la app (`MODE_PRIVATE`).
- **Manifiesto:** vacío, sin permisos. El código no hace llamadas de red, no tiene telemetría ni dependencias nativas extra; solo usa `appcompat` y Capacitor.
- **JS (`dist/`):** el puente de Capacitor y una implementación web sobre `localStorage`. La app no la usa, porque en la web sigue el módulo propio.
- **Scripts de instalación:** ninguno.
- **A tener en cuenta:** declara `@capacitor/ios` y `@capacitor/keyboard` como `dependencies` y no como *peer*. Se instalan como JS sin uso. `cap sync` solo registra los plugins de primer nivel de `package.json`, así que no se agregan al APK.
- **Conclusión:** apto. La versión queda fijada en el *lockfile*.

**Capacitor** (`@capacitor/core`, `cli`, `android`, `app`, `filesystem` y `share`; Ionic, MIT): ninguno tiene `preinstall`, `install` ni `postinstall`.

## Proyecto Android (T018–T021, 2026-10-06)

- **Compilación:** `./gradlew assembleDebug` y `pnpm android:release` (con una URL https de prueba) compilan con JDK 21 y SDK 36. Sin `VITE_API_BASE_URL` https, `vite build --mode android` falla a propósito (RF-007).
- **APK de release** (`aapt2 dump`):
  - `versionName` 0.1.0, `versionCode` 100, `targetSdk` 36, `minSdk` 24.
  - `allowBackup=false`, `fullBackupContent=false` y `dataExtractionRules`.
  - Sin `usesCleartextTraffic` y con `allowMixedContent: false`.
  - Único permiso: `INTERNET`.
- **Firma:** `apksigner verify` OK (esquema v2) con la clave `CN=Facturacion SUNAT, O=facturacion-peru`. `git status` no muestra la clave, `keystore.properties` ni `local.properties`.
- **APK de debug:** `usesCleartextTraffic=true` y `debuggable=true`. Es la única variante que habla HTTP con la PC.
- **`FileProvider`:** limitado a la caché de la app (la plantilla de Capacitor exponía también el almacenamiento externo).
- **CORS local:** el preflight desde `https://localhost` contra la IP de la red (`192.168.1.12`) permite el origen y `x-app-version`, y el login responde 200.

## Verificación en emulador (T022 parcial, 2026-10-06)

Emulador Android 15 (API 35, x86_64, 360×800 dp, 3 GB de RAM y GPU por software), con la variante de desarrollo contra la API local por `10.0.2.2`. No reemplaza al teléfono real de CE-001 y CE-002, pero cubre el comportamiento.

| Comprobación | Resultado |
|---|---|
| Ícono, nombre y pantalla completa (HU-1 esc. 1) | Ícono adaptativo del logo y nombre «Facturación SUNAT» |
| Entrar como vendedor (HU-1 esc. 2) | Inicio «Hola, Luis» |
| Primer arranque sin sesión | Login sin «Tu sesión terminó». Se corrigió el guard: al inicio ya no agrega `redirect`, un detalle que venía de la 001 |
| Ticket e «Imprimir» (HU-3 esc. 1) | T-000010 y modal. «Imprimir» abre «Compartir» con `T-000010-80mm.pdf` (opciones Imprimir, Drive, Bluetooth…) |
| Boleta real en SUNAT beta | B001-00000012 aceptada. El botón decía «Emitir boleta · B001» (spec 012 v1.5) |
| Descargas (HU-3 esc. 2) | PDF A4, XML y CDR abren «Compartir». Los archivos quedan en la caché de la app |
| Botón «Atrás» (HU-3 esc. 3, RF-005) | Cierra «Compartir», luego el modal, vuelve al Inicio y en el Inicio manda la app al fondo |
| Arranque en frío con sesión (HU-1 esc. 3) | Entra directo; la venta en curso (1 producto) sigue ahí |
| Sin conexión (HU-4) | Con modo avión aparece el aviso y desaparece al volver la red. Cobrar sin red muestra el error de conexión y conserva el carrito |
| Versión mínima (HU-5 esc. 1) | Con `APP_ANDROID_MIN_VERSION=0.2.0` aparece «Actualiza la app», con 0.2.0 y la versión instalada 0.1.0 |
| Instalar encima (HU-5 esc. 2) | La 0.2.0 sobre la 0.1.0 conserva la sesión y la venta en curso |
| Cuenta de plataforma (HU-1 esc. 4, RF-003) | «El panel de la plataforma se usa desde la web, no desde la app». El token creado se revoca: los tokens de la cuenta siguen en 3 |
| Cerrar sesión (HU-2 esc. 2) | Borra la venta en curso y conserva la preferencia de comprobante (spec 012) |
| Sesión segura (CE-003, en el dispositivo) | Fuera del almacenamiento web. En `shared_prefs` solo hay texto cifrado (AES-GCM, clave en Keystore); el token en claro no aparece en ningún archivo de la app |
| Modo oscuro del sistema | La app lo sigue (`data-theme=dark`) |
| Teclado | Al editar la cantidad, la vista pasa de 752 a 521 px y el campo y «Cobrar» siguen visibles |

**Hallazgos:**

- **El WebView de Android no actualiza `navigator.onLine`** (Capacitor no le avisa del estado de la red). Por eso el aviso sin conexión no aparecía. Se corrigió el plan y se agregó `@capacitor/network` detrás de `core/device` (`onConnectionChange`).
- **Arranque en frío lento en el emulador.** El Inicio tarda unos 6 s en aparecer (primera pintura con contenido a los 6,3 s), por el arranque del WebView con GPU por software. En una recarga en caliente, la primera pintura llega ~0,25 s después de cargar el HTML, sin tareas largas.
- **`php artisan serve` atiende de a una conexión.** Desde el emulador, dos llamadas simultáneas tardaban 4,7 s; con `PHP_CLI_SERVER_WORKERS=4`, 0,1 s. Es solo del servidor de desarrollo.
- **Ticket de prueba.** Un toque automático registró por error el ticket T-000011 en la base local; se anuló como administrador (motivo «Prueba automática en el emulador»).

**Pendiente en un teléfono real de gama baja:** CE-001 completo y CE-002 (apertura en menos de 3 s), además de probar «Compartir» con una impresora o una app de impresión reales.

## Teléfono real por Wi-Fi (2026-10-06)

Poco X3 NFC (M2007J20CG), Android 12 (SDK 31), 5,5 GB de RAM. Variante *debug* instalada por depuración inalámbrica, contra la API de la PC (`192.168.1.12:8001`, `PHP_CLI_SERVER_WORKERS=4`).

- **Instalación:** MIUI rechaza `adb install` con `INSTALL_FAILED_USER_RESTRICTED` hasta activar «Instalar vía USB» y aceptar el aviso en el teléfono, que dura pocos segundos. Quedó documentado en el README.
- **Red:** desde la app, `GET /api/v1/auth/me` sin sesión responde 401 en unos 100 ms y CORS acepta el origen `https://localhost`.
- **Primera apertura sin sesión:** el login se pinta a 1,06 s.
- **Sesión:** se guarda en el Keystore al entrar y se borra al cerrar sesión. El registro mostró `internalRemoveItem` tras «Cerrar sesión».

**CE-002: apertura en frío con la sesión guardada, del toque al Inicio pintado.** Medido con el reloj del teléfono, `performance.timeOrigin` y el LCP:

| Arranque | Actividad (`am start -W`) | WebView listo | Inicio pintado |
|---|---|---|---|
| 1 | 998 ms | 887 ms | 2175 ms |
| 2 | 979 ms | 861 ms | 1937 ms |
| 3 | 975 ms | 873 ms | 2025 ms |

Promedio: **~2,0 s**, menos de los 3 s que pide CE-002. Unos 0,9 s son el arranque del WebView y ~0,7 s la restauración de la sesión (`auth/me`) más el Inicio (`dashboard` y `sunat/status`, en paralelo).

El Poco X3 NFC es de **gama media de 2020**, no de gama baja. Para cerrar CE-001 y CE-002 tal como están escritos, falta repetir en un equipo de 2–3 GB de RAM. Si no se consigue uno, se puede dar por aceptado con este margen (1 s por debajo del límite).

> **Decisión (2026-10-06, del usuario):** CE-002 se acepta con esta medición en gama media, por el margen de ~1 s.

### CE-001 en el teléfono real (2026-10-06, probado por el usuario)

| Comprobación | Resultado |
|---|---|
| Entrar | Sí, con la cuenta de administrador de la empresa demo |
| Ticket | T-000012 y T-000013 |
| Boleta en SUNAT beta | B001-00000013 y B100-00000304, aceptadas |
| «Imprimir» | Genera y comparte el PDF de 80 mm del ticket (`GET /tickets/13/pdf`) y del comprobante (`/sales-documents/27/pdf`). No se probó con una impresora física porque no había una al alcance |
| Botón «Atrás» | Correcto |
| Modo avión | Aparece el aviso «Sin conexión» |
| Cerrar sesión | Borra el token del Keystore |

**Queda fuera de esta verificación:** imprimir en una térmica real. Las térmicas suelen imprimir desde su propia app o desde el servicio de impresión de Android, recibiendo el PDF por «Compartir». La impresión Bluetooth directa sigue siendo una spec aparte (A-63).
