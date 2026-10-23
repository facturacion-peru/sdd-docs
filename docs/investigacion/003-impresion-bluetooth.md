# Evaluación: impresión Bluetooth en la app Android

**Fecha:** 2026-10-23 · **Para:** [spec 015](../specs/015-impresion-bluetooth/spec.md) · **Preguntas:** A-75 a A-78 en [`aclaraciones.md`](../aclaraciones.md#015--impresión-bluetooth)

Pedido del usuario: que la app Android imprima por Bluetooth en impresoras térmicas. La spec 013 lo dejó para una spec propia (A-63, opción B). La visión lo tenía fuera del MVP.

## Estado actual

- **Web:** el ticket se imprime con el diálogo del navegador (vista de 80 mm, `TicketReceipt`). Los comprobantes, con su PDF de 80 mm.
- **App (spec 013):** «Imprimir» pide el PDF de 80 mm a la API y abre «Compartir». Desde ahí el usuario elige la app de su impresora, si la tiene y si esa app acepta PDF. Son varios toques por venta, y en algunos modelos baratos la app del fabricante no acepta PDF.
- **Dónde se imprime:** «Imprimir» del diálogo tras cobrar (`SaleDoneDialog`), el detalle del ticket y el detalle del comprobante (PDF A4 o 80 mm).
- **La API genera:**
  - el PDF de 80 mm del ticket y el del comprobante;
  - el QR de SUNAT (`DocumentPdf::qrPayload`) y el resumen (*hash*).
- **Constitución:**
  - **Principio II:** el PDF, el QR y los formatos de impresión (A4, 80 mm, 50 mm) pertenecen a la API.
  - **Principio VII:** la impresión Bluetooth pasa por un módulo aislado y sustituible (`src/core/device`).
- **Permisos:** la app solo declara `INTERNET`. No hay nada de Bluetooth.

## Cómo funcionan estas impresoras

- **Lenguaje:** casi todas las térmicas portátiles y de mostrador (Xprinter, GOOJPRT, MTP-II, Epson TM, Bixolon, 3nStar…) entienden **ESC/POS**. Son bytes con el texto y comandos de negrita, alineación, corte, QR e imagen.
- **Conexión:**
  - **Bluetooth clásico (SPP, «puerto serie»):** es el de la gran mayoría de las impresoras baratas de 58 y 80 mm. Se emparejan desde los ajustes de Android, a veces con el PIN 0000 o 1234.
  - **Bluetooth de baja energía (BLE):** algunos modelos más nuevos lo traen, a menudo junto con SPP. Cada fabricante usa sus propios «canales» (UUID), así que no hay una forma estándar.
- **Ancho del papel:** 58 mm (32 caracteres por línea, 384 puntos) y 80 mm (48 caracteres, 576 puntos). Las de 58 mm son las más comunes en bodegas.
- **Tildes y «ñ»:** hay que elegir una tabla de caracteres (por ejemplo PC850 o PC858) y convertir el texto. Si no, salen símbolos raros. No todas las impresoras traen las mismas tablas.
- **QR:**
  - La mayoría lo dibuja con un comando propio (`GS ( k`), que es rápido.
  - Las más baratas no lo tienen. Para ellas se manda el QR como imagen, algo más lento (unos 2 a 3 KB).
  - Los comprobantes necesitan el QR en la representación impresa. El ticket interno no.
- **Logo:** se manda como imagen en blanco y negro (*raster*). Sirve, pero alarga la impresión y depende de cómo se vea el logo a 1 bit.

## Opciones técnicas (para el plan, no para la spec)

### Quién arma el contenido

| Opción | A favor | En contra |
|---|---|---|
| **La API devuelve los bytes ESC/POS** del ticket o comprobante, para 58 u 80 mm | Cumple el principio II: el formato vive junto al PDF y el QR, con los mismos datos. Se prueba en PHP con pruebas de bytes | Hay que pasar a la API los ajustes de cada impresora (ancho, tabla de caracteres, QR nativo o imagen) |
| La app arma los bytes desde el JSON del documento | Funciona con los datos ya cargados | Va contra el principio II: duplica el formato y el QR en el frontend |

**Recomendación técnica:** la API arma los bytes. Sería una ruta nueva por documento con el ancho y las opciones de la impresora. La app solo los envía.

### Conexión desde Android

| Opción | Estado (2026-10-23) | Comentario |
|---|---|---|
| `@capacitor-community/bluetooth-le` | 8.3.0, MIT, Capacitor 8 | Mantenido, pero **solo BLE**: no sirve para la mayoría de las impresoras SPP |
| `capacitor-bluetooth-serial` | 0.0.4, de 2022 | Abandonado y sin Capacitor 8 |
| `@kduma-autoid/capacitor-bluetooth-printer` | 0.2.1, para Capacitor 6 | Desactualizado |
| `capacitor-thermal-printer` | 0.2.5, para Capacitor 7 | Desactualizado y con poco uso |
| **Plugin propio** en `android/` (Kotlin) | — | Son unas 150 líneas: listar las impresoras emparejadas, abrir un `BluetoothSocket` SPP (UUID `00001101-…`), escribir los bytes y cerrar. No agrega dependencias de terceros, lo que va en línea con la política de supply-chain |

**Recomendación técnica:** plugin propio para SPP, detrás de `core/device`. Si A-75 incluye BLE, evaluar entonces `@capacitor-community/bluetooth-le` en una versión que cumpla `minimumReleaseAge`.

### Permisos y emparejamiento

- **Android 12 o más:**
  - `BLUETOOTH_CONNECT` basta para usar las impresoras **ya emparejadas**.
  - Buscar equipos nuevos desde la app exigiría además `BLUETOOTH_SCAN`.
  - En versiones anteriores, también la ubicación.
- **Android 11 o menos:** `BLUETOOTH` y `BLUETOOTH_ADMIN` (con `maxSdkVersion`).
- **Emparejar desde los ajustes de Android** y elegir en la app entre las emparejadas evita pedir la búsqueda y la ubicación. Es lo habitual en las apps de punto de venta.

### Web

Chrome tiene Web Bluetooth, pero solo para BLE y no en todos los navegadores. En la web se mantiene el diálogo de impresión del navegador.

## Riesgos

- **Variedad de impresoras:** sin un modelo de referencia no se puede verificar. Hace falta al menos una impresora de 58 mm y, si es posible, una de 80 mm, de las que usarían las empresas del piloto.
- **Fallos a mitad de impresión** (papel, batería, fuera de alcance): la venta ya está registrada, así que imprimir solo **reproduce** el documento y nunca cambia datos. Debe poder reintentarse y quedar la opción de «Compartir PDF» (A-63).
- **Sin conexión a internet:** si los bytes vienen de la API, no se puede imprimir sin red. Es coherente con A-66, porque sin red tampoco se vende.
- **Emulador:** no tiene Bluetooth real. Las pruebas en él se limitan a la interfaz con un dispositivo simulado. La verificación real es con teléfono e impresora.

## Alcance sugerido para la spec 015

- Elegir en la app una impresora Bluetooth emparejada, con su ancho de papel, y guardarla en ese teléfono.
- Imprimir directo a ella el ticket y los comprobantes (boleta, factura y nota de crédito, con QR y resumen) desde los mismos botones «Imprimir».
- Hoja de prueba, avisos claros si falla y la opción de «Compartir PDF» como alternativa.
- **Fuera de alcance:**
  - impresión Bluetooth desde la web;
  - impresoras de red (Wi-Fi o USB);
  - abrir el cajón de dinero;
  - imprimir sin conexión;
  - etiquetas o códigos de barras de productos.
