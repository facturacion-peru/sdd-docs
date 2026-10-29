# Verificación 015: Impresión Bluetooth en la app Android

**Fecha:** 2026-10-23 · **Ramas:** `015-impresion-bluetooth` (API y frontend) · **Estado:** implementada, **falta verificar con impresora** (A-75). CE-004 verificado en el teléfono real

Sin impresora física todavía. Se verificó todo lo que no la necesita: los bytes en pruebas automáticas, la app en el emulador y el texto impreso a ojo. Quedan pendientes CE-001 a CE-003, que solo se pueden comprobar en papel. CE-004 se verificó en el teléfono real (T031).

## Pruebas automáticas

| Suite | Resultado |
|---|---|
| API · SQLite | 825 pruebas (13 omitidas: solo PostgreSQL o `pcntl`) |
| API · PostgreSQL | 838 pruebas |
| API · Pint | Limpio |
| Frontend | 348 pruebas, tipos, lint y build |
| APK para empresas | Compila firmado (0.2.0, `versionCode` 200) |

Lo que cubren las nuevas:

- **Bytes:** secuencias exactas de cada comando, QR nativo con su longitud en dos bytes, PC850 y caracteres de control quitados de los textos.
- **Ancho:** ninguna línea pasa de 32 o 48 caracteres, ni con textos de 100.
- **QR como imagen:** se compara módulo por módulo con la matriz de endroid.
- **Paridad:** toda palabra del PDF de 80 mm aparece en la impresión, en ticket, ticket anulado, boleta con descuento y exonerada, factura y nota de crédito, en los dos anchos. El QR es igual a `qrPayload`.
- **Acceso:** el vendedor recibe 404 por un ticket ajeno, otra empresa recibe 404, una opción inválida da 422 y sin sesión da 401.
- **Solo lectura (RF-004):** imprimir no cambia nada y dos impresiones dan los mismos bytes.
- **App:** plugin simulado, store por teléfono, `usePrinting`, aviso de fallo, pantalla «Impresora», guard y menú solo nativos, «Imprimir al cobrar» una vez por venta.

## Permisos del APK

`aapt2 dump permissions` del APK para empresas:

- **Permisos:** `INTERNET`, `ACCESS_NETWORK_STATE`, `BLUETOOTH` y `BLUETOOTH_ADMIN` (`maxSdkVersion=30`) y `BLUETOOTH_CONNECT`.
- **Sin** `BLUETOOTH_SCAN` ni ubicación.
- **Bluetooth no obligatorio:** `android.hardware.bluetooth` figura como `uses-feature-not-required`, así que la app se instala también en teléfonos sin Bluetooth.

## Emulador (T030)

Emulador Android 15 (`gama-baja`, 360 dp, Bluetooth virtual sin equipos), variante *debug* contra la API local, como vendedor.

| Comprobación | Resultado |
|---|---|
| Menú | «Impresora» aparece en la app, después de «Productos» |
| Permiso (HU-1 esc. 2) | La app explica el permiso antes de pedirlo. «Ver impresoras emparejadas» abre el diálogo «Dispositivos cercanos» de Android |
| Permiso negado | Aviso «Sin permiso de Bluetooth» con cómo darlo en Ajustes, y nada guardado (`sunat.printer` vacío). «Volver a intentar» pide el permiso otra vez |
| Permiso concedido, sin equipos (HU-1 esc. 4) | La lista vacía llega del plugin real, con «Abrir ajustes de Bluetooth», que abre «Dispositivos conectados» de Android; «Atrás» vuelve a la app |
| Bluetooth apagado | Aviso «El Bluetooth está apagado». «Encender Bluetooth» abre el diálogo de Android y, al permitir, el aviso desaparece |
| Impresora que ya no está emparejada (caso límite) | Con una impresora guardada a mano, «Imprimir» en un ticket pide `/tickets/11/print` a la API y el plugin responde `not_paired`. El aviso aparece en **1,4 s** con «Reintentar», «Compartir PDF» y «Elegir otra impresora» |
| «Compartir PDF» | Pide `/tickets/11/pdf` y abre el selector de Android |
| «Elegir otra impresora» | Lleva a «Impresora», con el equipo guardado y las opciones (360 dp sin desbordes) |
| Hoja de prueba con fallo | Mismo aviso, sin «Compartir PDF» (la hoja no tiene PDF) |
| Cerrar sesión (HU-1 esc. 5) | Vuelve al login y `sunat.printer` sigue guardado |

**Corrección en el camino:** con el Bluetooth apagado, la pantalla seguía ofreciendo «Ver impresoras emparejadas», que solo podía fallar. Ahora se oculta hasta encenderlo.

El emulador no puede probar `connect_failed`, `write_failed` ni una impresión real: no tiene equipos emparejados.

## Teléfono real (T031)

2026-10-29, Poco X3 NFC (Android 12, MIUI) con la variante de desarrollo 0.2.0 contra la API local. Lo probó el usuario. Como impresora se eligió un equipo Bluetooth cualquiera ya emparejado, apagado al imprimir.

| Comprobación | Resultado |
|---|---|
| Elegir el equipo en «Impresora», con el permiso «Dispositivos cercanos» | Correcto |
| **CE-004:** «Imprimir» con el equipo apagado | El aviso «No se imprimió» salió en **unos 8 s** (el plugin corta a los 8 s), menos que los 10 s pedidos |
| La venta tras el fallo | Quedó registrada sin cambios |

**Hallazgo corregido:** después de un fallo, el aviso seguía en «Venta registrada» en la venta siguiente. El diálogo no se desmonta entre ventas en «Vender». Ahora el aviso se limpia al cambiar de venta, y una prueba lo cubre (`print-bluetooth.spec.ts`; frontend: 348 pruebas).

## Texto impreso (T032)

Decodificado con `Tests\Support\EscposText` desde la base local, sin las marcas de estilo. El decodificador no muestra el centrado: los títulos, el QR y las leyendas salen centrados en el papel. `[QR:…]` es el QR nativo con su contenido exacto. Revisado a ojo:

- **Ancho:** cada línea cabe en el ancho elegido.
- **Importes:** van a la derecha.
- **Datos largos:** las etiquetas largas pasan el valor a la línea siguiente, como «Fecha de emisión» en 58 mm.
- **Tildes y signos:** se conservan, y la raya «—» sale como «-».

#### Ticket T-000015 · 58 mm
```
Bodega Demo
RUC 20600000013
TICKET T-000015
07/10/2026 12:21
--------------------------------
Documento interno - no es
comprobante de pago
--------------------------------
Cliente           Cliente varios
Atendió       Ana Administradora
--------------------------------
Atún en aceite 170 g
1 x S/ 6.50              S/ 6.50
Azúcar rubia
1 x S/ 4.20              S/ 4.20
Caramelos x 100
1 x S/ 9.00              S/ 9.00
--------------------------------
TOTAL   S/ 19.70
Pago                    Efectivo

Documento interno - no es
comprobante de pago
```

#### Boleta B001-00000013 · 58 mm
```
PRUEBAS - SIN VALOR LEGAL

Bodega Demo
Empresa Demo S.A.C.
Av. Demo 123
--------------------------------
RUC 20600000013
BOLETA DE VENTA ELECTRÓNICA
B001-00000013
--------------------------------
Fecha de emisión
  06/10/2026 17:36
Cliente          CLIENTES VARIOS
Moneda
  Soles · al contado (Efectivo)
--------------------------------
Cant. Descripción
P. unit.                 Importe
--------------------------------
2 Pollo entero
Dscto. 2.00
10.90                      19.80
--------------------------------
Op. gravadas            S/ 16.78
Descuentos (incluidos)   S/ 2.00
IGV (18 %)               S/ 3.02
Importe total           S/ 19.80

SON DIECINUEVE CON 80/100 SOLES

[QR:20600000013|03|B001|13|3.02|19.80|2026-10-06|0|-|4hkdyw1NCfBcOl4zLrvTTPtKIIc=|]
Resumen:
4hkdyw1NCfBcOl4zLrvTTPtKIIc=
--------------------------------
Representación impresa de la
boleta de venta electrónica
emitida en el ambiente de
pruebas de SUNAT.
PRUEBAS - SIN VALOR LEGAL
```

#### Factura F100-00000001 · 80 mm
```
PRUEBAS - SIN VALOR LEGAL

Bodega Demo
Empresa Demo S.A.C.
Av. Demo 123
------------------------------------------------
RUC 20600000013
FACTURA ELECTRÓNICA
F100-00000001
------------------------------------------------
Fecha de emisión                06/10/2026 13:29
Cliente                 FERRETERÍA EL SOL S.A.C.
RUC                                  20100070970
Dirección                Av. Los Pinos 456, Lima
Moneda             Soles · al contado (Efectivo)
------------------------------------------------
Cant. Descripción
P. unit.                                 Importe
------------------------------------------------
1 Detergente 900 g
11.90                                      11.90
1 Gaseosa 1.5 L
6.00                                        6.00
------------------------------------------------
Op. gravadas                            S/ 15.16
IGV (18 %)                               S/ 2.74
Importe total                           S/ 17.90

SON DIECISIETE CON 90/100 SOLES

[QR:20600000013|01|F100|1|2.74|17.90|2026-10-06|6|20100070970|t/z4MjKTn93RTxrEIEhpDWEW0xU=|]
Resumen: t/z4MjKTn93RTxrEIEhpDWEW0xU=
------------------------------------------------
Representación impresa de la factura electrónica
emitida en el ambiente de pruebas de SUNAT.
PRUEBAS - SIN VALOR LEGAL
```

#### Nota de crédito BC01-00000004 · 58 mm
```
PRUEBAS - SIN VALOR LEGAL

Bodega Demo
Empresa Demo S.A.C.
Av. Demo 123
--------------------------------
RUC 20600000013
NOTA DE CRÉDITO ELECTRÓNICA
BC01-00000004
--------------------------------
Fecha de emisión
  06/10/2026 13:45
Cliente          CLIENTES VARIOS
Moneda
  Soles · al contado (Efectivo)
Documento que modifica
  Boleta de venta B100-00000303
Motivo
  01 · Anulación de la
  operación: dañado
--------------------------------
Cant. Descripción
P. unit.                 Importe
--------------------------------
1 Delivery en el distrito
5.00                        5.00
1 Detergente 900 g
11.90                      11.90
1 Fósforos x 10
2.00                        2.00
--------------------------------
Op. gravadas            S/ 16.01
IGV (18 %)               S/ 2.89
Importe total           S/ 18.90

SON DIECIOCHO CON 90/100 SOLES

[QR:20600000013|07|BC01|4|2.89|18.90|2026-10-06|0|-|HnOdosnb4QzZdFWWu1JrX77pGc4=|]
Resumen:
HnOdosnb4QzZdFWWu1JrX77pGc4=
--------------------------------
Representación impresa de la
nota de crédito electrónica
emitida en el ambiente de
pruebas de SUNAT.
PRUEBAS - SIN VALOR LEGAL
```

## Pendiente con la impresora de referencia (T033)

- **Hoja de prueba:** que la regla llegue al borde, que las tildes salgan bien con PC850 y que el QR nativo se dibuje (si no, probar «Mi impresora no imprime QR»).
- **CE-001:** menos de 5 s de «Imprimir» al papel, con un ticket de 10 líneas.
- **CE-002:** el QR de una boleta aceptada en beta se lee con el celular y coincide con el del PDF.
- **CE-003:** tildes, «ñ», «S/» e importes bien en 58 y, si se puede, en 80 mm.
- **Ajustes posibles** en la primera impresión real: la pausa final (300 ms) o el tamaño de bloque (512 bytes) del plugin, y la tabla de caracteres.
