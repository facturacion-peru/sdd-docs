# Spec 015: Impresión Bluetooth en la app Android

**Estado:** Aprobada (2026-10-23) · **Versión:** 1.0 · **Fecha:** 2026-10-23
**Depende de:** 003, 005, 007, 013 · **Aclaraciones:** A-75 a A-78 (retoma A-63, opción B)
**Evaluación previa:** [`investigacion/003-impresion-bluetooth.md`](../../investigacion/003-impresion-bluetooth.md)

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que quien vende desde la app Android imprima el ticket o el comprobante **directo en su impresora térmica Bluetooth** con un solo toque. Hoy tiene que pasar por «Compartir» y por la app del fabricante, que a veces no acepta PDF (A-63).

## Historias de usuario

### HU-1 · Elegir mi impresora (P1)

Como usuario de empresa en la app, quiero elegir una vez mi impresora Bluetooth y su ancho de papel para no configurarla en cada venta.

**Por qué esta prioridad:** sin una impresora elegida no hay impresión directa.
**Prueba independiente:** con una impresora ya emparejada en Android, la elijo en la app, imprimo la hoja de prueba y la elección sigue ahí al cerrar y abrir la app.

**Escenarios de aceptación**

1. **Dado** impresoras emparejadas en los ajustes de Android, **cuando** abro «Impresora» en la app, **entonces** veo la lista de equipos emparejados y elijo uno, con su ancho de papel: 58 u 80 mm (A-76).
2. **Dado** que la app aún no tiene permiso de Bluetooth, **cuando** abro «Impresora», **entonces** se pide el permiso explicando para qué es. Si lo niego, veo cómo darlo después y la app sigue funcionando con «Compartir».
3. **Dado** una impresora elegida, **cuando** pulso «Hoja de prueba», **entonces** sale una hoja con el nombre de la empresa, una línea de ancho completo, tildes y «ñ», y un QR de prueba.
4. **Dado** que no hay impresoras emparejadas, **cuando** abro «Impresora», **entonces** veo cómo emparejarla desde los ajustes de Android, con un acceso directo a esos ajustes.
5. **Dado** una impresora elegida, **cuando** cierro sesión o entra otro usuario en el mismo teléfono, **entonces** la elección se conserva, porque es del teléfono y no de la cuenta (A-78).

### HU-2 · Imprimir el ticket directo (P1)

Como vendedor, quiero que «Imprimir» saque el ticket en mi térmica sin pasar por otras apps.

**Por qué esta prioridad:** el ticket es la venta más frecuente y es lo que más se imprime.
**Prueba independiente:** con una impresora de referencia, cobrar un ticket y pulsar «Imprimir» lo imprime completo y legible.

**Escenarios de aceptación**

1. **Dado** una impresora elegida, **cuando** pulso «Imprimir» tras cobrar o en el detalle del ticket, **entonces** se imprime el ticket con el mismo contenido que su versión de 80 mm, ajustado al ancho elegido.
2. **Dado** que la impresión está en curso, **cuando** espero, **entonces** veo que está imprimiendo y no puedo mandarla dos veces por accidente.
3. **Dado** que activé «Imprimir al cobrar» en este teléfono, **cuando** cobro un ticket, **entonces** se imprime sin pulsar «Imprimir». La opción viene apagada (A-78).

### HU-3 · Imprimir comprobantes (P1)

Como vendedor, quiero imprimir boletas, facturas y notas de crédito en la misma térmica, con su QR, para entregarlas al cliente.

**Por qué esta prioridad:** el comprobante impreso debe llevar los datos de la representación impresa (QR y resumen), igual que el PDF.
**Prueba independiente:** una boleta aceptada en beta se imprime con todos los datos del PDF de 80 mm, incluido un QR que se lee con el celular y coincide con el del PDF.

**Escenarios de aceptación**

1. **Dado** una boleta, factura o nota de crédito, **cuando** pulso «Imprimir», **entonces** sale con el mismo contenido que su PDF de 80 mm, incluidos la marca de pruebas en beta, el monto en letras, el QR y el resumen (A-77).
2. **Dado** una impresora que no dibuja QR por sí misma, **cuando** imprimo un comprobante, **entonces** el QR sale igual, como imagen.

### HU-4 · Cuando la impresión falla (P2)

Como vendedor, quiero saber por qué no imprimió y tener otra salida para no dejar al cliente esperando.

**Por qué esta prioridad:** la venta ya está registrada. Lo importante es poder entregar el documento.
**Prueba independiente:** con la impresora apagada, «Imprimir» muestra el aviso con «Reintentar» y «Compartir PDF», y al encenderla «Reintentar» imprime.

**Escenarios de aceptación**

1. **Dado** la impresora apagada, fuera de alcance o con el Bluetooth del teléfono apagado, **cuando** pulso «Imprimir», **entonces** en pocos segundos veo un aviso claro con «Reintentar» y «Compartir PDF».
2. **Dado** que no hay impresora elegida, **cuando** pulso «Imprimir», **entonces** se abre «Compartir» con el PDF, como hasta ahora (spec 013).
3. **Dado** que falla la impresión, **cuando** reviso la venta, **entonces** sigue registrada igual: imprimir nunca cambia la venta, el stock ni el comprobante.

## Casos límite

- **Bluetooth apagado:** se avisa y se ofrece encenderlo, o ir a los ajustes.
- **Sin papel o tapa abierta:** la mayoría de estas impresoras no lo informa por Bluetooth, así que la app no puede saberlo. La impresión no sale y se repite con «Imprimir» tras poner papel.
- **Impresora desemparejada:** si se desempareja en Android después de elegirla, la app avisa al imprimir y pide elegir otra.
- **Textos largos:** nombres de producto, razón social o dirección más largos que la línea pasan a la línea siguiente sin cortar palabras ni importes. Los importes quedan alineados a la derecha.
- **Caracteres sin equivalente** en la impresora (por ejemplo, emojis en un nombre): se reemplazan por uno parecido o se omiten, sin cortar la impresión.
- **Sin internet:** como en la 013 (A-66), se avisa. Imprimir un documento necesita sus datos del servidor.
- **Muchas líneas:** un ticket de 50 líneas se imprime completo.
- **Web:** en el navegador no aparece nada de Bluetooth. Se imprime como hoy.

## Requisitos funcionales

- **RF-001:** La app Android DEBE permitir elegir una impresora térmica Bluetooth emparejada y su ancho de papel (A-75, A-76), y recordarlos en ese teléfono.
- **RF-002:** «Imprimir» DEBE enviar el ticket, la boleta, la factura o la nota de crédito directo a la impresora elegida, desde los mismos lugares donde hoy se imprime.
- **RF-003:** El contenido impreso DEBE ser el mismo que el del PDF de 80 mm del documento, adaptado al ancho, incluidos el QR y el resumen en los comprobantes. Lo genera el servidor a partir de los datos guardados (principio II).
- **RF-004:** Imprimir NO DEBE modificar la venta, el stock ni el comprobante, y DEBE poder repetirse.
- **RF-005:** Ante un fallo, la app DEBE indicar el motivo y ofrecer «Reintentar» y «Compartir PDF».
- **RF-006:** La app DEBE pedir solo los permisos de Bluetooth necesarios para usar equipos emparejados, en el momento en que se configura la impresora, y seguir funcionando si se niegan.
- **RF-007:** La impresión automática al cobrar DEBE ser opcional, por teléfono y apagada por defecto (A-78).
- **RF-008:** Sin impresora elegida, o en la web, el comportamiento DEBE ser el de las specs 003 y 013.
- **RF-009:** El acceso al Bluetooth DEBE estar aislado en el módulo del dispositivo (principio VII), de modo que las pantallas no dependan de él.

## Entidades clave

- **Impresora del teléfono:** el equipo Bluetooth elegido (nombre e identificador), su ancho de papel y la opción de imprimir al cobrar. Vive solo en el teléfono, no en el servidor.
- **Documento para la térmica:** la versión del ticket o del comprobante preparada para la impresora y el ancho elegidos, con el mismo contenido que su PDF de 80 mm.

## Criterios de éxito

- **CE-001:** Con la impresora de referencia (A-75) ya elegida, de «Imprimir» al papel pasan **menos de 5 segundos** para un ticket de 10 líneas.
- **CE-002:** El QR impreso de una boleta aceptada en beta se lee con un celular y su contenido es igual al del PDF.
- **CE-003:** Las tildes, la «ñ», «S/» y los importes salen bien en 58 y 80 mm, sin líneas desbordadas.
- **CE-004:** Con la impresora apagada, el aviso con alternativas aparece en **menos de 10 segundos**, y la venta sigue intacta.

## Fuera de alcance

- Impresión Bluetooth desde la web.
- Impresoras Wi-Fi, de red o USB.
- Bluetooth de baja energía (A-75).
- Buscar y emparejar impresoras desde la app (se hace en los ajustes de Android).
- Logo impreso y varias copias (A-77).
- Abrir el cajón de dinero, imprimir sin conexión, y etiquetas o códigos de barras de productos.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-23 | Spec creada a partir de A-63, opción B | — |
| 2026-10-23 | A-75 | Bluetooth clásico (SPP), ESC/POS; impresora de referencia de 58 mm (modelo por elegir) |
| 2026-10-23 | A-76 | 58 y 80 mm, elegidos al configurar la impresora |
| 2026-10-23 | A-77 | Ticket, boleta, factura y nota de crédito como el PDF de 80 mm, con QR; sin logo; una copia |
| 2026-10-23 | A-78 | Emparejar en Android y elegir en la app; por teléfono; «Imprimir al cobrar» opcional y apagado; «Reintentar» y «Compartir PDF» si falla |
| 2026-10-23 | Aprobada (v1.0) | Al redactar el plan: «sin papel» pasa de HU-4 a los casos límite, porque estas impresoras no lo informan por Bluetooth |
