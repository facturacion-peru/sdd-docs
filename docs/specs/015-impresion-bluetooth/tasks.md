# Tareas 015: Impresión Bluetooth en la app Android

**Estado:** En curso: falta T033 (impresora) y T036 (merge) · **Aprobadas:** 2026-10-23 · **Fecha:** 2026-10-23 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior. 🔒 = requiere confirmación explícita del usuario. 🖨️ = necesita la impresora física (A-75): queda pendiente hasta tenerla.

## Fase 1 · Preparación

- [x] **T001** Rama `015-impresion-bluetooth` desde `main` en la API y en el frontend — Verificación: `git branch --show-current` en ambos
- [x] **T002** Comprobar que `endroid/qr-code` (la versión instalada) expone la matriz del QR sin GD y que `mb_convert_encoding(..., 'CP850', 'UTF-8')` está disponible en la CI (`mbstring`) — Verificación: script de prueba en el *scratchpad*, sin commit. Si falta algo, detenerse y consultar (riesgo del plan) Hecho: `Endroid\QrCode\Bacon\MatrixFactory` da la matriz sin GD; CP850 convierte á é í ó ú ñ Ñ ¿ ¡ « » y la CI ya tiene `mbstring`.
## Fase 2 · API: librería ESC/POS

- [x] **T003** `tests/Support/EscposText`: decodificador de pruebas que convierte los bytes en el texto impreso (CP850 → UTF-8, una línea por salto), con marcas de estilo (`[B]`, `[2x]`, `[QR:…]`, `[IMG wxh]`, `[CUT]`), y sus pruebas — Verificación: en verde
- [x] **T004** Pruebas de `EscposBuilder` (deben fallar):
  - secuencias exactas de iniciar, PC850, alineación, negrita, `GS !`, avance y corte;
  - QR nativo con `pL pH` correctos para 100 y 300 bytes, módulo 5 (58 mm) y 6 (80 mm);
  - tildes, ñ, ¿, ¡, « y » en CP850; `—`, `−`, `×` y comillas tipográficas reemplazados; emojis omitidos;
  - `line` y `row(left, right)` parten por palabras, importes a la derecha, palabras largas partidas;
  - ninguna línea supera 32 o 48 caracteres con textos de 100 caracteres.

  Verificación: en rojo
- [x] **T005** `App\Printing\Escpos\EscposBuilder` y `App\Printing\PrintOptions` — Verificación: T004 en verde
- [x] **T006** Pruebas de `QrRaster` (deben fallar): ancho múltiplo de 8 puntos, cabe en 384 y 576, bandas de 24 filas en `GS v 0` y la matriz decodificada igual a la de endroid — Verificación: en rojo Hecho: las pruebas se escribieron junto con el código (T007), sin pasar antes por rojo.
- [x] **T007** `App\Printing\Escpos\QrRaster` y `EscposBuilder::qr(data, 'image')` — Verificación: T006 en verde

## Fase 3 · API: documentos y rutas (HU-2, HU-3)

- [x] **T008** Pruebas de acceso de las tres rutas y entradas en `TenantRoutes.php` (deben fallar):
  - el vendedor recibe 404 por el ticket de otro vendedor y el administrador 200;
  - otra empresa, 404;
  - `width=70` y `qr=x` dan 422; por defecto, 80 y `native`;
  - `Content-Type: application/octet-stream`.

  Verificación: en rojo
- [x] **T009** `PrintRequest`, `PrintController` (`ticket`, `salesDocument`, `testPage`) y las rutas; `TestPagePrint` (nombre de la empresa, ancho, regla, tildes, «S/ 1,234.50» y QR de prueba) — Verificación: T008 en verde, salvo el contenido de ticket y comprobante Hecho: `ticket` y `salesDocument` van en sus controladores de siempre (`TicketController::print`, `SalesDocumentController::print`), para reusar el mismo control de acceso que el PDF; la hoja de prueba, en `Printing\PrintingController`. El título y las etiquetas del PDF pasaron a `DocumentPdf` (`title()`, `CUSTOMER_DOCUMENT_LABELS`, `REPRESENTATION_NAMES`) para que PDF e impresión usen los mismos.
- [x] **T010** Pruebas de paridad y contenido del ticket (deben fallar): todo el texto del PDF de 80 mm aparece en la impresión decodificada (normal y «ANULADO»), ticket de 50 líneas completo con el TOTAL al final — Verificación: en rojo
- [x] **T011** `TicketPrint` — Verificación: T010 en verde
- [x] **T012** Pruebas de paridad de comprobantes (deben fallar): boleta con descuento y exonerada, factura y nota de crédito, en beta; QR igual a `DocumentPdf::qrPayload`, resumen, monto en letras y marca de pruebas; en 58 y 80 mm — Verificación: en rojo
- [x] **T013** `DocumentPrint` — Verificación: T012 en verde
- [x] **T014** Prueba de RF-004: imprimir no cambia `updated_at`, el stock ni la auditoría (conteos antes y después) — Verificación: en verde
- [x] **T015** `php artisan openapi:generate`, suites SQLite y PostgreSQL y Pint — Verificación: códigos de salida 0 y cifras anotadas Hecho: API 825 pruebas en SQLite (13 omitidas) y 838 en PostgreSQL, Pint limpio. El generador documenta `width` como entero y `qr` como texto (sin la lista de valores).
## Fase 4 · App Android: plugin `BluetoothPrinter`

- [x] **T016** Permisos y `uses-feature` en `AndroidManifest.xml` (`BLUETOOTH_CONNECT`; `BLUETOOTH` y `BLUETOOTH_ADMIN` con `maxSdkVersion="30"`) — Verificación: `aapt dump permissions` sobre el APK *debug*, sin `BLUETOOTH_SCAN` ni ubicación
- [x] **T017** `BluetoothPrinterPlugin.java`: `checkPermission`, `requestPermission`, `status`, `listPaired`, `openSettings` y `requestEnable`; registro en `MainActivity` — Verificación: `pnpm android:release` compila (JDK 21) Hecho: también se sobrescriben `checkPermissions` y `requestPermissions` de Capacitor: en Android 11 o menos responden «concedido», porque ahí `BLUETOOTH_CONNECT` no existe.
- [x] **T018** `print({ address, data })` en un hilo aparte: RFCOMM SPP seguro y luego *insecure*, bloques de 512 bytes, pausa final, vigilantes de 8 s (conexión) y 15 s (escritura) y los códigos de error del plan — Verificación: compila; se prueba en T030 y T031 Hecho: código de error extra `invalid` (faltan la dirección o los datos).
## Fase 5 · Frontend

- [x] **T019** `pnpm api:types`; separar `fetchFile(path): Promise<Blob>` en `core/api/download.ts` sin cambiar `downloadFile` — Verificación: pruebas actuales en verde y `pnpm type-check`
- [x] **T020** Pruebas de `device().printer` (deben fallar): en `device-native.spec.ts` el plugin simulado recibe base64, un rechazo se traduce a `PrinterError` con su código y `listPaired` pide el permiso si falta; en `device-web.spec.ts`, `available` es `false` y los métodos lanzan `unsupported` — Verificación: en rojo
- [x] **T021** `printer` en `types.ts`, `web.ts` y `native.ts`, y `PrinterError` — Verificación: T020 en verde Hecho: `PrinterError` lleva también `unknown` para rechazos sin código conocido.
- [x] **T022** [P] Pruebas y store `printing/stores/printer-settings.ts` (`sunat.printer`, sobrevive a `logout()` y al cambio de usuario, sin configurar si `localStorage` falla) — Verificación: en verde
- [x] **T023** `printing/api.ts` y pruebas de `usePrinting` (deben fallar):
  - con impresora, pide los bytes con el ancho y el modo guardados y llama a `print`;
  - sin impresora, `window.print` (ticket) o PDF (comprobante) en la web, y PDF en la app;
  - `busy` impide el doble envío; un fallo deja `failure` con el código.

  Verificación: en rojo
- [x] **T024** `composables/usePrinting.ts` — Verificación: T023 en verde
- [x] **T025** Pruebas y `PrintFailedDialog.vue`: motivo por código, «Reintentar», «Compartir PDF», «Elegir otra impresora» (`not_paired`) y «Encender Bluetooth» (`bluetooth_off`) — Verificación: en verde Hecho: es un aviso dentro de la pantalla (`PrintFailedAlert.vue`), no un diálogo: en «Venta registrada» habría quedado un diálogo encima de otro, y así sigue el estilo de los avisos de error que ya había.
- [x] **T026** Pruebas y `PrinterView.vue` con su ruta `/impresora` (`meta.nativeOnly`):
  - lista las emparejadas y guarda la elección;
  - sin emparejadas, acceso a los ajustes;
  - permiso negado, aviso y nada guardado;
  - Bluetooth no disponible o apagado;
  - ancho, «Mi impresora no imprime QR», «Imprimir al cobrar» (apagado) y hoja de prueba con lo elegido.

  Verificación: en verde
- [x] **T027** Guard de `meta.nativeOnly` en el router y «Impresora» en `MainNav` solo con `isNative`, con pruebas — Verificación: en verde
- [x] **T028** `SaleDoneDialog`, `TicketView` y `SalesDocumentView` pasan por `usePrinting`; impresión automática una sola vez con `autoPrint` (pruebas de RF-007); «Imprimir» nuevo en el detalle del comprobante — Verificación: pruebas nuevas y existentes en verde Hecho: `usePrinting` no importa nada de `sales`: cada pantalla le pasa su PDF (`pdf`) y, para el ticket en la web, la impresión del navegador (`web`).
- [x] **T029** Comprobaciones del frontend: `pnpm type-check && pnpm lint && pnpm test:unit --run && pnpm build-only`; subir `version` en `package.json` y `pnpm android:release` — Verificación: códigos de salida 0 y cifras anotadas Hecho: 347 pruebas, tipos, lint y build; versión 0.2.0. El APK para empresas exige una URL HTTPS: se compiló con `VITE_API_BASE_URL=https://api.invalid` solo para comprobarlo. `aapt2` confirma los permisos, sin búsqueda ni ubicación.
## Cierre

- [x] **T030** Emulador Android a 360 dp: permiso (conceder y negar), «Bluetooth no disponible», lista vacía con acceso a ajustes, diálogo de fallo y «Compartir PDF»; en la web no aparece «Impresora» y todo se imprime como antes — Verificación: capturas revisadas y anotadas en `verificacion.md` Hecho: ver [verificación](verificacion.md). Corrigió que con el Bluetooth apagado se ofreciera buscar equipos.
- [x] **T031** Teléfono real (Poco X3) con un equipo Bluetooth cualquiera emparejado: lista, elección, CE-004 (`connect_failed` en menos de 10 s, cronometrado) y la venta intacta; con el Bluetooth apagado, `bluetooth_off`. Lo hace el usuario o con depuración inalámbrica — Verificación: tiempos anotados en `verificacion.md` Hecho (2026-10-29, lo probó el usuario): aviso en unos 8 s y la venta intacta. Corrigió que el aviso de fallo pasara a la venta siguiente en «Venta registrada».
- [x] **T032** Revisión a ojo del ticket, una boleta, una factura y una nota en 58 y 80 mm con `EscposText` (texto decodificado guardado en `verificacion.md`) — Verificación: revisado Hecho: ticket, boleta, factura y nota en [verificación](verificacion.md).
- [ ] **T033** 🖨️ Con la impresora de referencia: hoja de prueba (tabla PC850 y QR nativo/imagen), CE-001 (menos de 5 s), CE-002 (QR leído con el celular igual al del PDF) y CE-003 (tildes y anchos en 58 y 80 mm) — Verificación: resultados anotados. Hasta entonces la spec queda «Implementada, falta verificar con impresora»
- [x] **T034** Comprobaciones del principio VI en verde (API SQLite y PostgreSQL, Pint; frontend completo) — Verificación: códigos de salida 0 y cifras anotadas Hecho: API 825 (SQLite) y 838 (PostgreSQL), Pint; frontend 347 pruebas, tipos, lint y build.
- [x] **T035** Documentación:
  - README de la API (rutas de impresión) y del frontend (módulo `printing`, permisos Android);
  - trampas nuevas en `AGENTS.md`, si las hay;
  - bitácora;
  - estado en `docs/specs/README.md`.

  Verificación: revisada
- [ ] **T036** 🔒 Merge local de `015-impresion-bluetooth` a `main` en ambos repositorios y commit de la raíz — Verificación: suites en verde sobre `main`
