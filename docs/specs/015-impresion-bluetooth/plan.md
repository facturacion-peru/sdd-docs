# Plan 015: Impresión Bluetooth en la app Android

**Estado:** Aprobado (2026-10-23) · **Fecha:** 2026-10-23 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

**API:** arma los **bytes ESC/POS** del ticket, del comprobante y de una hoja de prueba, para 58 u 80 mm. Usa los mismos datos guardados que los PDF de 80 mm, con el mismo QR (`DocumentPdf::qrPayload`) y el mismo monto en letras (principio II).
- **Dónde:** una pequeña librería propia en `app/Printing/Escpos`, sin dependencias nuevas.
- **Cómo se pide:** tres rutas `GET` nuevas que responden `application/octet-stream`. Imprimir no escribe nada en la base.

**App Android:**
- **Plugin propio** `BluetoothPrinter`, en Java dentro de `android/`. Lista las impresoras emparejadas, abre un canal SPP, envía los bytes y lo cierra. Solo pide `BLUETOOTH_CONNECT`.
- **`core/device`:** queda detrás de una capacidad nueva, `device().printer`. En la web no está disponible.

**Frontend:**
- **Feature nueva `printing`:** la pantalla «Impresora» (elegir equipo, ancho y opciones, más la hoja de prueba) y el *composable* `usePrinting`, que decide entre Bluetooth y el camino de siempre (`window.print()` o «Compartir PDF»).
- **Uso:** «Imprimir» del diálogo tras cobrar, del detalle del ticket y del detalle del comprobante pasa por `usePrinting`.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | API y frontend (incluido `android/`) |
| Ramas | `015-impresion-bluetooth` en los dos, desde `main` |
| Dependencias nuevas | **Ninguna.** El ESC/POS se escribe a mano (es poco: texto, estilos, QR y corte). El QR como imagen sale de la matriz de `endroid/qr-code`, que ya está, sin GD. El plugin Bluetooth es código propio (ver la evaluación: los plugins SPP publicados están abandonados o no soportan Capacitor 8) |
| Migraciones | Ninguna |
| Permisos Android | `BLUETOOTH_CONNECT` (Android 12+, en tiempo de ejecución). `BLUETOOTH` y `BLUETOOTH_ADMIN` con `maxSdkVersion="30"` (Android 7–11, sin diálogo). **No** se piden `BLUETOOTH_SCAN` ni la ubicación |
| Lenguaje del plugin | **Java**, como `MainActivity`: Kotlin obligaría a añadir su plugin de Gradle |
| Versión de la app | Sube `version` en `package.json` (de ahí sale `versionCode`). Las versiones viejas siguen funcionando, porque las rutas nuevas son solo aditivas. `APP_ANDROID_MIN_VERSION` no cambia |
| Impresora física | **No hay todavía** (A-75). Ver «Verificación sin impresora» |

## Chequeo de constitución

| Principio | Cumple | Cómo |
|---|---|---|
| I. Spec primero | Sí | Spec 015 v1.0 aprobada, con A-75 a A-78 |
| II. Corrección fiscal | Sí | La API arma la impresión desde los datos guardados, con el mismo QR, resumen, monto en letras y marca «PRUEBAS — SIN VALOR LEGAL» que el PDF de 80 mm. Una prueba de paridad compara los textos de los dos formatos. Imprimir es de solo lectura |
| III. Contrato de API | Sí | Rutas bajo `/api/v1` con un FormRequest (`width`, `qr`), `php artisan openapi:generate` y `pnpm api:types`. La respuesta es binaria y no hay JSON que tipar |
| IV. Seguridad | Sí | Mismas reglas de acceso que los PDF: el vendedor solo pide sus tickets (404 para los demás). Rutas en `TenantRoutes.php`. En el teléfono solo se guarda el nombre y la dirección Bluetooth de la impresora, que no es un dato sensible (A-64). Permiso mínimo: sin búsqueda ni ubicación |
| V. Simplicidad | Sí | Sin librerías nuevas. Una conexión por impresión (abrir, escribir, cerrar), sin conexiones persistentes ni colas. Solo Bluetooth clásico (A-75) |
| VI. Verificable | Parcial | Pest (bytes exactos, anchos, codificación, paridad con el PDF) y Vitest (plugin simulado). El plugin Java y los criterios CE-001 a CE-004 necesitan una impresora real: quedan **pendientes** hasta tenerla, y la spec no se da por completada sin ellos |
| VII. Personas reales | Sí | Todo el Bluetooth va en `core/device` (el principio lo nombra). Avisos en español con el motivo y salidas («Reintentar», «Compartir PDF»). La pantalla funciona a 360 px. Sin impresora configurada, nada cambia |
| VIII. Aislamiento y auditoría | Sí | Los documentos se resuelven en el contexto de empresa, como los PDF. Imprimir no se audita, igual que descargar el PDF hoy: no cambia nada |

## Formato ESC/POS

### Anchos (A-76)

| Papel | Puntos | Caracteres por línea (fuente A) | Módulo del QR nativo |
|---|---|---|---|
| 58 mm | 384 | 32 | 5 |
| 80 mm | 576 | 48 | 6 |

### Comandos que se usan

Son los del juego básico de Epson, que entienden las compatibles:

| Uso | Bytes |
|---|---|
| Iniciar | `ESC @` |
| Tabla de caracteres **PC850** | `ESC t 2` |
| Alinear izquierda, centro o derecha | `ESC a 0/1/2` |
| Negrita | `ESC E 1/0` |
| Doble alto y ancho (título y total) | `GS ! 0x11` / `GS ! 0x00` |
| QR nativo, modelo 2, corrección M | `GS ( k`: funciones 165, 167, 169, 180 y 181 |
| QR como imagen | `GS v 0`, en bandas de 24 filas para no saturar el búfer de las impresoras baratas |
| Avance y corte parcial | `ESC d 4` + `GS V 66 0`. Las impresoras sin cuchilla lo ignoran |

### Texto

- **Codificación:** se convierte a **CP850** con `mb_convert_encoding`, que ya se probó con á, é, í, ó, ú, ñ, Ñ, ¿, ¡, « y ». PC850 es la tabla más extendida en estas impresoras. Si la referencia no la trae, cambiar la tabla es una constante.
- **Caracteres sin equivalente:** antes de convertir, `—`, `−` y `×` pasan a `-`, `-` y `x`, y las comillas tipográficas a rectas. Lo que siga sin equivalente (emojis) se omite. Se cumple el caso límite de la spec.
- **Ajuste de línea:**
  - El ajuste de línea se calcula **antes** de codificar, contando caracteres y no bytes.
  - Las palabras más largas que la línea se parten.
  - Las filas `texto … importe` ponen el importe a la derecha. Si no cabe, el texto ocupa sus líneas y el importe va en la última.
  - **Ninguna línea supera el ancho**, y una prueba lo comprueba en los dos anchos.

### Contenido

Es el mismo que el de los PDF de 80 mm, en el mismo orden:

- **Ticket** (`pdf/ticket.blade.php`):
  - «ANULADO» si corresponde;
  - nombre, RUC, número y fecha;
  - leyenda de documento interno arriba y abajo;
  - cliente, documento y quién atendió;
  - líneas (nombre, cantidad × precio, importe y descuento);
  - descuentos, TOTAL y medio de pago.
- **Comprobante** (`pdf/sales-document.blade.php`, rama `80mm`):
  - marca de pruebas;
  - emisor (nombre comercial, razón social y dirección);
  - RUC, título y número;
  - fecha, cliente, documento, dirección y moneda con medio de pago;
  - en las notas de crédito, documento que modifica y motivo;
  - líneas con cantidad, descripción, afectación, descuento, precio unitario e importe;
  - operaciones gravadas, exoneradas e inafectas, descuentos, IGV e importe total;
  - monto en letras, **QR** y resumen;
  - leyenda de representación impresa y marca de pruebas.
- **Hoja de prueba:**
  - nombre de la empresa;
  - ancho elegido;
  - una regla de ancho completo (`1234567890…` hasta 32 o 48);
  - la línea «Tildes: áéíóú ÁÉÍÓÚ ñÑ ¿¡ «»»;
  - «S/ 1,234.50» alineado a la derecha;
  - un QR de prueba («Prueba de impresora») con el modo elegido.

Sin logo (A-77).

### Organización en la API

| Clase | Papel |
|---|---|
| `App\Printing\Escpos\EscposBuilder` | Constructor fluido de bytes: `text`, `line`, `row(left, right)`, `bold`, `big`, `align`, `rule`, `qr(data, mode)`, `feedAndCut`, `bytes()`. Conoce el ancho y la codificación |
| `App\Printing\Escpos\QrRaster` | Matriz de `endroid/qr-code` → imagen de 1 bit escalada para `GS v 0` |
| `App\Printing\TicketPrint`, `DocumentPrint`, `TestPagePrint` | Arman cada documento con el *builder*. Reutilizan `DocumentPdf::qrPayload`, `AmountInWords` y las etiquetas de los *enums* |
| `App\Printing\PrintOptions` | `width` (58 o 80) y `qr` (`native` o `image`), validados por `PrintRequest` |

## Contrato de API

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/api/v1/tickets/{ticket}/print?width=58\|80` | Usuario de empresa (el vendedor, solo sus tickets) | Bytes ESC/POS del ticket. Mismo control de acceso que `/pdf` |
| GET | `/api/v1/sales-documents/{salesDocument}/print?width=58\|80&qr=native\|image` | Usuario de empresa | Bytes ESC/POS del comprobante o la nota |
| GET | `/api/v1/printing/test-page?width=58\|80&qr=native\|image` | Usuario de empresa | Bytes de la hoja de prueba, con el nombre de la empresa |

- **Respuesta:** `200` con `Content-Type: application/octet-stream`.
- **Errores:** `422` si `width` o `qr` no son válidos, y el `404` de siempre para documentos ajenos o de otra empresa.
- **Valores por defecto:** `width=80` y `qr=native`.
- **Límite de frecuencia:** el general de la API.

## Plugin Android `BluetoothPrinter`

`android/app/src/main/java/io/github/facturacion_peru/app/printer/BluetoothPrinterPlugin.java`. Se registra en `MainActivity` con `registerPlugin(...)` antes de `super.onCreate`.

| Método | Qué hace |
|---|---|
| `checkPermission()` / `requestPermission()` | Estado de `BLUETOOTH_CONNECT`, con el alias `bluetooth` de Capacitor. En Android ≤ 11 siempre `granted` |
| `status()` | `{ supported, enabled }`: si el teléfono tiene Bluetooth y si está encendido |
| `listPaired()` | `[{ name, address }]` de los equipos emparejados. Muestra todos, porque muchas impresoras se anuncian con clases genéricas, y ordena primero los de clase «imagen/impresora» |
| `print({ address, data })` | `data` llega en base64. Corre en un hilo aparte: abre un `BluetoothSocket` RFCOMM con el UUID SPP `00001101-0000-1000-8000-00805F9B34FB` (si falla el seguro, prueba el *insecure*), escribe en bloques de 512 bytes con `flush`, espera unos 300 ms y cierra |
| `openSettings()` | Abre los ajustes de Bluetooth de Android (`Settings.ACTION_BLUETOOTH_SETTINGS`) |
| `requestEnable()` | Pide encender el Bluetooth (`ACTION_REQUEST_ENABLE`) |

- **Tiempo límite:**
  - La conexión tiene un vigilante que cierra el socket a los **8 s** (CE-004 pide un aviso antes de 10 s).
  - La escritura tiene otro vigilante de 15 s.
- **Errores** (`call.reject(message, code)`):

  | Código | Cuándo |
  |---|---|
  | `permission_denied` | Sin permiso de Bluetooth |
  | `bluetooth_off` | El Bluetooth del teléfono está apagado |
  | `not_paired` | La dirección ya no está emparejada |
  | `connect_failed` | Impresora apagada o fuera de alcance |
  | `write_failed` | Se cortó a mitad de la impresión |
  | `unsupported` | El teléfono no tiene Bluetooth |

- **Manifiesto:**
  - `BLUETOOTH_CONNECT`;
  - `BLUETOOTH` y `BLUETOOTH_ADMIN` con `maxSdkVersion="30"`;
  - `<uses-feature android:name="android.hardware.bluetooth" android:required="false"/>`, para que la app se instale también en teléfonos sin Bluetooth.

## Frontend

### `core/device`

Se agrega a `Device` (`types.ts`):

```ts
readonly printer: {
  /** false en la web: ahí se imprime como siempre. */
  readonly available: boolean
  status(): Promise<{ supported: boolean; enabled: boolean }>
  /** Pide el permiso si hace falta; lanza PrinterError('permission_denied') si se niega. */
  listPaired(): Promise<PairedPrinter[]>
  print(address: string, bytes: Uint8Array): Promise<void>
  openSettings(): Promise<void>
  requestEnable(): Promise<void>
}
```

- **`PrinterError`:** lleva un `code` con los códigos del plugin.
- **`web.ts`:** `available: false`, y los métodos lanzan `unsupported`.
- **`native.ts`:** usa `registerPlugin('BluetoothPrinter')` de `@capacitor/core` y convierte los bytes a base64 con el `toBase64` que ya existe.
- **Ninguna feature** importa el plugin.

### `core/api/download.ts`

La parte del `fetch` (cabeceras, errores y `X-App-Version`) se separa en `fetchFile(path): Promise<Blob>`. `downloadFile` la usa, e imprimir también, para pedir los bytes.

### Feature `printing`

| Archivo | Papel |
|---|---|
| `stores/printer-settings.ts` | Pinia con persistencia en `localStorage` bajo `sunat.printer`. Es **por teléfono, no por empresa ni usuario** (HU-1, esc. 5) y no se borra al cerrar sesión. Guarda `{ address, name, width: 58\|80, qr: 'native'\|'image', autoPrint: false }`. Si la lectura falla, queda sin configurar |
| `api.ts` | `ticketBytes`, `documentBytes` y `testPageBytes` (`fetchFile` → `Uint8Array`) |
| `composables/usePrinting.ts` | `printTicket(ticket)` y `printDocument(document)`. **Con impresora configurada** (`device().printer.available` y la impresora guardada), pide los bytes y los envía. **Sin ella**, hace lo de las specs 003 y 013: `window.print()` del ticket en la web, o el PDF de 80 mm («Compartir» en la app, descarga en la web). Expone `busy` (evita doble envío) y `failure` (código y mensaje) |
| `components/PrintFailedDialog.vue` | Motivo en español según el código, más «Reintentar» y «Compartir PDF». Con `not_paired`, «Elegir otra impresora». Con `bluetooth_off`, «Encender Bluetooth» |
| `views/PrinterView.vue` | Ruta `/impresora`, para todos los roles y con `meta.nativeOnly`: en la web, el guard redirige a «/». Muestra el estado y el permiso, con un aviso y cómo darlo en los ajustes si se niega. Lista las emparejadas con «Abrir ajustes de Bluetooth» si no hay ninguna. Luego ancho 58 u 80 mm (`BaseSelect`), «Mi impresora no imprime QR» (→ `qr: 'image'`), «Imprimir al cobrar» (apagado) y «Hoja de prueba» |
| `routes.ts` | La ruta, colgada de `AppLayout` |

### Cambios en pantallas existentes

| Dónde | Cambio |
|---|---|
| `SaleDoneDialog.vue` | «Imprimir» → `usePrinting`. Con `autoPrint` y una impresora configurada, imprime solo al abrirse (ticket o comprobante), una vez. «Imprimiendo…» deshabilita el botón |
| `TicketView.vue` | «Imprimir» → `printTicket` |
| `SalesDocumentView.vue` | Botón «Imprimir» nuevo (`printDocument`), visible si hay impresora configurada. «PDF 80 mm» sigue igual |
| `MainNav.vue` | «Impresora» en el grupo principal, **solo** si `device().isNative` |
| `router` | Guard de `meta.nativeOnly` |

Los colores salen solo del tema y los campos usan `BaseSelect` y casillas como las demás pantallas: lo comprueba `theme.spec.ts`.

## Estrategia de pruebas

| Requisito / escenario | Prueba | Tipo |
|---|---|---|
| RF-003 · bytes | `EscposBuilderTest`: secuencias exactas de iniciar, PC850, negrita, alineación, `GS !`, QR nativo (longitudes `pL pH` correctas para cargas de 100 y 300 bytes), cabecera `GS v 0` y corte | Unitaria (Pest) |
| RF-003 · texto | Tildes, ñ, ¿, ¡, « y » en CP850. Emojis omitidos y `—`/`×` reemplazados. `row()` alinea y parte. **Ninguna línea** supera 32 o 48 caracteres con nombres largos (100 caracteres) en los dos anchos | Unitaria |
| RF-003 · QR imagen | `QrRaster`: el ancho en bytes es múltiplo de 8 puntos, cabe en 384 y 576, y la matriz decodificada coincide con la de endroid | Unitaria |
| RF-003, HU-3 · paridad | Para un ticket, una boleta con descuento y exonerada, una factura y una nota de crédito: todo texto del PDF de 80 mm (Blade sin etiquetas, por normalización) aparece en la impresión decodificada, salvo el logo. Incluye QR igual a `qrPayload`, resumen, monto en letras y marca de pruebas | Feature |
| HU-2 · 50 líneas | Ticket de 50 líneas completo y con el TOTAL al final | Feature |
| Acceso | El vendedor recibe 404 por el ticket de otro vendedor, y el administrador recibe 200. Otra empresa recibe 404 (vía `TenantRoutes.php`). `width=70` y `qr=x` dan 422. `Content-Type` correcto | Feature |
| RF-004 | Imprimir no cambia `updated_at`, el stock ni la auditoría (conteo antes y después) | Feature |
| RF-009, RF-006 | `device-native.spec.ts`: el plugin simulado recibe base64 de los bytes; un rechazo se traduce a `PrinterError` con su código; `listPaired` pide el permiso si falta. `device-web.spec.ts`: `available` es `false` | Unitaria (Vitest) |
| HU-1 | `PrinterView.spec.ts`: lista emparejadas y guarda la elección; sin emparejadas, muestra el acceso a ajustes; permiso negado, aviso y nada guardado; hoja de prueba con el ancho y el modo de QR elegidos | Unitaria |
| HU-1 · esc. 5 | `printer-settings.spec.ts`: persiste en `sunat.printer`, sobrevive a `logout()` y a un cambio de usuario, y queda sin configurar si `localStorage` falla | Unitaria |
| HU-2, HU-3, RF-002, RF-008 | `usePrinting.spec.ts`: con impresora, bytes del ancho y modo guardados → `print`; sin impresora, en la web `window.print` (ticket) o PDF (comprobante); sin impresora en la app, PDF; `busy` impide el doble envío | Unitaria |
| HU-4, RF-005 | `PrintFailedDialog`: cada código muestra su motivo. «Reintentar» vuelve a imprimir. «Compartir PDF» llama a la descarga de 80 mm | Unitaria |
| RF-007 | `SaleDoneDialog`: con `autoPrint` imprime una sola vez al abrir. Sin `autoPrint`, o sin impresora, no imprime | Unitaria |
| Navegación | «Impresora» solo con `isNative`. `/impresora` en la web redirige a «/» | Unitaria |
| Suites | API en SQLite y PostgreSQL, Pint, frontend completo y `pnpm android:release` compila | Suites |

### Verificación sin impresora (hasta tener la de referencia)

1. **Decodificador de pruebas:** un ayudante de pruebas, `tests/Support/EscposText`, convierte los bytes en el texto que saldría impreso, con marcas de estilo. Se usa en la prueba de paridad y para revisar a ojo el ticket de 58 y 80 mm en la verificación.
2. **Emulador:** no tiene Bluetooth real. Sirve para revisar el permiso, «Bluetooth no disponible», la lista vacía, el diálogo de fallo y «Compartir PDF», además de la pantalla a 360 px.
3. **Teléfono real (Poco X3):**
   - **Con un equipo Bluetooth cualquiera emparejado** (audífonos o un parlante): se comprueba `connect_failed` en menos de 10 s (CE-004) y que la venta queda intacta.
   - **Si hay un segundo teléfono:** una app que emule una impresora ESC/POS por SPP permite ver la impresión completa. Es opcional y no reemplaza la impresora real.
4. **Quedan pendientes** hasta tener la impresora: CE-001 (menos de 5 s), CE-002 (QR legible), CE-003 (tildes y anchos en papel) y la prueba real del QR nativo frente al QR como imagen. La spec queda **«Implementada, falta verificar con impresora»**, sin pasar a *Completada*.

## Riesgos

| Riesgo | Mitigación |
|---|---|
| La impresora de referencia no usa PC850, o numera distinto la tabla | La tabla es una constante y la hoja de prueba lo muestra al primer intento. Si hace falta, se agrega una opción «Tabla de caracteres», que sería una nueva versión de la spec |
| No dibuja el QR nativo | Opción «Mi impresora no imprime QR» → imagen (A-77) |
| Implementar el plugin Java sin poder probarlo con una impresora | Código corto y conocido (RFCOMM SPP), revisado con el teléfono real y un equipo cualquiera emparejado (caminos de error). La primera impresión real puede pedir ajustes (pausa al final, tamaño de bloque) |
| `connect()` de Android bloquea hasta unos 12 s | Vigilante que cierra el socket a los 8 s |
| Datos que no caben en una línea angosta (razón social, dirección, motivo) | Ajuste de línea por palabras y prueba de que ninguna línea supera el ancho |
| Paridad PDF ↔ ESC/POS con el tiempo (se cambia uno y no el otro) | Prueba de paridad automática sobre los textos |
| Teléfonos sin Bluetooth | `uses-feature required=false`, más `status().supported` en la pantalla |
| Varias impresiones rápidas, o auto-impresión más «Imprimir» | `busy` en `usePrinting`, compartido por la pantalla |
