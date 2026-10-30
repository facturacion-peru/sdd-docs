# Verificación 016: Imagen del producto y logo en S3

## F002 · Cámara y galería en el WebView de Android (2026-10-29)

Emulador `gama-baja` (Android 15, API 35, cámara trasera emulada), APK *debug* de la rama con dos cambios **temporales**, sin commit. La prueba se hizo con un `<input type="file">` inyectado por DevTools del WebView (Chrome 124), sin tocar pantallas.

**Lectura del código de Capacitor 8** (`BridgeWebChromeClient.onShowFileChooser`):
- Abre la cámara **solo** si el `<input>` tiene `capture` **y** su `accept` incluye literalmente `image/*`. Con `accept="image/jpeg,image/png,image/webp"` y `capture`, abre el selector de fotos.
- Si la app **no declara** `CAMERA`, lanza la cámara del sistema sin pedir permiso. Si la declarara sin tenerlo concedido, lo pediría, y negarlo devuelve una cancelación, sin forma de distinguirla.
- La foto tomada se guarda en `getExternalFilesDir(Pictures)` y se entrega con el `FileProvider` de la app.

| Prueba | Resultado |
|---|---|
| Cámara, sin cambios en Android | ❌ Abre el selector de fotos. Log: `Media capture intent could not be launched. Falling back to default file picker`. Android 11+ oculta la app de cámara sin `<queries>` para `IMAGE_CAPTURE` (además, `file_paths.xml` solo permite la caché) |
| Cámara con `<queries>` (`IMAGE_CAPTURE`) y `<external-files-path path="Pictures/">` | ✅ Abre la cámara **sin pedir permiso** y sin `CAMERA` en el manifiesto. Llega un `File` `image/jpeg` (1392×1856, 74 KB, `ffd8ffe1`). `createImageBitmap` lo decodifica |
| «Atrás» en la cámara | ✅ Vuelve a la app, el `<input>` emite `cancel` y la ruta no cambia |
| Galería: `accept="image/jpeg,image/png,image/webp"` sin `capture` | ✅ Abre el selector de fotos del sistema (Android 13+), «This app can only access the photos you select», **sin permiso de almacenamiento**. Llega un `File` `image/jpeg` |
| «Atrás» en el selector de fotos | ✅ Emite `cancel`; la app sigue en la misma pantalla |

**Decisión:** no hace falta `@capacitor/camera` ni declarar `CAMERA`. F007 lleva:
- `<queries>` con `android.media.action.IMAGE_CAPTURE` en el manifiesto;
- `<external-files-path name="captured_photos" path="Pictures/" />` en `file_paths.xml`;
- `pickImage('camera')` con `accept="image/*"` y `capture="environment"`; el formato lo revisa `prepareProductPhoto` por los *magic bytes*;
- borrar la foto temporal de `Pictures/` después de leerla. Capacitor no la borra, y la cámara del teléfono puede haberle puesto GPS.

**Queda para F017 (teléfono real, Poco X3 con Android 12):** sin el selector de fotos de Android 13+, la galería abre otra app (Documentos o Google Fotos). También queda comprobar la orientación de una foto vertical real (CE-004).

## Teléfono real con disco local (2026-10-29)

Poco X3 NFC (Android 12) por depuración inalámbrica, APK *debug* 0.3.0 de la rama. La API de la rama corre en la PC con `IMAGES_DISK=public` (sin AWS: aún no existe el bucket) y `PHP_CLI_SERVER_WORKERS=4`, por wifi.

| Prueba | Resultado |
|---|---|
| Subida desde la PC con `curl` (subir, ver y quitar; JPEG con orientación 6) | ✅ WebP de 800 y 200 px, la foto queda vertical |
| Primera prueba del usuario: cámara y galería | ❌ Sale «Foto guardada» y la foto queda en el disco (producto «Aceite vegetal 1 L»), pero la app no la muestra. Logcat: `Mixed Content … requested an insecure image … blocked` |
| Tras el cambio: la miniatura vista por DevTools del WebView | ✅ La app carga como `http://localhost`, la miniatura carga (200×200) y la sesión sigue abierta |
| Segunda prueba del usuario | ✅ «Funciona bien» |

**Causa:** el WebView de la app *debug* cargaba como `https://localhost`. Chromium bloquea las imágenes HTTP de la API aunque `allowMixedContent` deje pasar `fetch` y XHR, por eso la subida funcionaba y la foto no se veía. **Corrección** (frontend `dc4a368`): `server.androidScheme: 'http'` solo con `CAP_ANDROID_DEV=1`. La API de desarrollo necesita entonces `http://localhost` en `CORS_ALLOWED_ORIGINS`. La app de producción no cambia: carga como `https://localhost` y las fotos vienen del bucket por HTTPS.

**Queda para F017 y T020, con el bucket:** CE-001 cronometrado con datos móviles, CE-002, 360 px y modo oscuro, TalkBack y modo avión con el formulario abierto. El usuario confirmó el funcionamiento en general, sin detallar cada punto de la lista (CE-004 incluido); conviene repetirlos contra S3.
