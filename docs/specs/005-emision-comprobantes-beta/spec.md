# Spec 005: Emisión y consulta de comprobantes en beta

**Estado:** En aclaración · **Versión:** 0.1 · **Fecha:** 2026-09-26
**Depende de:** 001, 002, 004 · **Aclaraciones:** A-20, A-22, A-23

> Los requisitos marcados con `[A-xx]` siguen la recomendación de esa aclaración y se ajustarán si la decisión es otra.
> ⚖️ Los casos tributarios de esta spec deben revisarse con un contador ([A-22](../../aclaraciones.md#a-22)).

## Objetivo

Que un vendedor emita una **factura o boleta electrónica** en el ambiente beta de SUNAT desde la misma venta, conozca su resultado (aceptada, observada o rechazada) y pueda descargar el PDF, el XML y el CDR. Si SUNAT no responde, el comprobante no se pierde y se puede recuperar.

## Historias de usuario

### HU-1 · Emitir una boleta (P1)

Como vendedor, quiero emitir una boleta a un consumidor, para entregarle un comprobante válido.

**Prueba independiente:** emitir una boleta en beta con productos gravados y exonerados y obtener el CDR aceptado.

**Escenarios de aceptación**

1. **Dado** una empresa con la configuración SUNAT validada y una serie de boleta activa, **cuando** registro una venta con productos y elijo "boleta", **entonces** el sistema calcula los importes, asigna el siguiente correlativo, genera y firma el XML, lo envía a SUNAT beta y descuenta el stock `[A-23]`.
2. **Dado** una boleta sin cliente identificado, **cuando** el total no supera el tope que exige SUNAT para identificar al comprador, **entonces** se emite a "Cliente varios"; si lo supera, el sistema exige el documento del cliente ⚖️.
3. **Dado** una boleta aceptada, **cuando** la abro, **entonces** veo su estado, el mensaje de SUNAT y puedo descargar el PDF (A4 u 80 mm), el XML y el CDR.

### HU-2 · Emitir una factura (P1)

Como vendedor, quiero emitir una factura a una empresa cliente, para que pueda sustentar su gasto.

**Escenarios de aceptación**

1. **Dado** un cliente con RUC válido, **cuando** emito una factura, **entonces** se sigue el mismo flujo que la boleta, con la serie de factura.
2. **Dado** un cliente sin RUC, **cuando** intento emitirle una factura, **entonces** el sistema lo impide y sugiere una boleta.
3. **Dado** una empresa en Nuevo RUS, **cuando** intento emitir una factura, **entonces** el sistema lo impide `[A-11]`.

### HU-3 · Cálculo correcto de importes (P1)

Como empresa, quiero que los importes y el IGV de mis comprobantes sean exactos, para no tener problemas con SUNAT.

**Escenarios de aceptación**

1. **Dado** productos gravados con precio con IGV incluido, **cuando** se emite, **entonces** el valor de venta, el IGV al 18 % y el total cumplen las reglas de redondeo de SUNAT y el comprobante es aceptado `[A-22]`.
2. **Dado** una mezcla de productos gravados, exonerados e inafectos, **cuando** se emite, **entonces** cada total se informa por separado y el total general cuadra.
3. **Dado** descuentos por línea, **cuando** se emite, **entonces** se reflejan correctamente en la base imponible.
4. **Dado** el mismo conjunto de líneas, **cuando** se calcula varias veces, **entonces** el resultado es siempre idéntico.

### HU-4 · Recuperar comprobantes pendientes (P1)

Como vendedor, quiero que si SUNAT no responde el comprobante quede pendiente y se envíe después, para no perder la venta ni el número `[A-23]`.

**Escenarios de aceptación**

1. **Dado** SUNAT sin responder, **cuando** emito, **entonces** la venta y el comprobante quedan guardados con su número en estado "pendiente de envío", y se me informa.
2. **Dado** un comprobante pendiente, **cuando** SUNAT vuelve a responder, **entonces** el sistema lo reenvía automáticamente con espera creciente entre intentos.
3. **Dado** un comprobante pendiente, **cuando** pulso "reintentar", **entonces** se reenvía de inmediato.
4. **Dado** un comprobante que SUNAT rechaza, **cuando** llega la respuesta, **entonces** queda "rechazado" con el código y el motivo de SUNAT, y no se reintenta.
5. **Dado** un comprobante reenviado que SUNAT ya había recibido, **cuando** responde que está duplicado, **entonces** el sistema obtiene su estado real en lugar de marcarlo como error.

### HU-5 · Consultar comprobantes (P2)

Como usuario de empresa, quiero listar mis comprobantes por fecha, tipo, estado y cliente, para hacer seguimiento.

**Escenarios de aceptación**

1. **Dado** comprobantes en varios estados, **cuando** filtro por "pendiente" o "rechazado", **entonces** veo solo esos, para actuar sobre ellos.
2. **Dado** que soy vendedor, **cuando** consulto comprobantes, **entonces** veo los de mi empresa; el alcance exacto se fija en el plan según A-05.

## Casos límite

- El stock se agota entre que armo la venta y la confirmo: no se emite nada ni se consume correlativo.
- Falla la firma (certificado corrupto): no se consume correlativo y el error se explica.
- SUNAT acepta con observaciones: el estado es "aceptado con observaciones" y se muestran.
- Dos vendedores emiten en la misma serie a la vez: números consecutivos, sin repetir.
- El cliente cambia sus datos después: el comprobante conserva los datos del momento de emitir.
- El reintento del usuario coincide con el reintento automático: se envía una sola vez.

## Requisitos funcionales

**Emisión**

- **RF-001:** El sistema DEBE emitir facturas y boletas electrónicas en el ambiente beta `[A-20]` `[A-22]`.
- **RF-002:** Solo DEBE permitirse emitir con la configuración SUNAT "validada" y una serie activa del tipo correspondiente (spec 004).
- **RF-003:** Todos los importes DEBEN calcularse en el servidor según las reglas de SUNAT, para operaciones gravadas (IGV 18 %), exoneradas e inafectas, en soles y al contado `[A-22]`.
- **RF-004:** La venta, el correlativo, el comprobante y el descuento de stock DEBEN confirmarse juntos o fallar juntos. El envío a SUNAT ocurre **después** de confirmar y no lo deshace.
- **RF-005:** El correlativo solo DEBE consumirse cuando el comprobante queda guardado.
- **RF-006:** El comprobante DEBE conservar los datos del emisor, del cliente y de los productos vigentes al emitirse.
- **RF-007:** El sistema DEBE generar el PDF con el código QR exigido por SUNAT en formatos A4 y 80 mm.

**Envío y estados**

- **RF-010:** Los estados del comprobante DEBEN ser: pendiente de envío, enviado, aceptado, aceptado con observaciones y rechazado.
- **RF-011:** El envío DEBE ser automático al emitir, con reintentos automáticos con espera creciente si SUNAT no responde, y un reintento manual disponible `[A-23]`.
- **RF-012:** Un mismo comprobante NO DEBE enviarse dos veces a la vez.
- **RF-013:** El sistema DEBE guardar el XML firmado, el CDR y el código y mensaje de respuesta de SUNAT.
- **RF-014:** Los comprobantes emitidos NO DEBEN editarse ni borrarse.

**Visibilidad**

- **RF-020:** Los comprobantes de beta DEBEN mostrarse siempre marcados como "PRUEBAS — sin valor legal".
- **RF-021:** El sistema DEBE permitir listar comprobantes por fecha, tipo, estado y cliente, y descargar PDF, XML y CDR.
- **RF-022:** DEBEN auditarse la emisión, los reintentos y los cambios de estado.

**Clientes** (mínimo necesario para emitir)

- **RF-030:** El sistema DEBE permitir registrar clientes de la empresa con tipo y número de documento (DNI, RUC, carné de extranjería), nombre o razón social y dirección opcional, validando el formato del documento.

## Entidades clave

- **Comprobante:** factura o boleta con serie, correlativo, emisor, cliente, totales por tipo de afectación, estado, XML, CDR y respuesta de SUNAT.
- **Línea de comprobante:** producto, cantidad, valor unitario, precio, afectación al IGV e importes, tal como se emitió.
- **Cliente:** comprador identificado de la empresa.
- **Intento de envío:** cada envío a SUNAT, con fecha, resultado y error si lo hubo.

## Criterios de éxito

- **CE-001:** Un conjunto de casos de prueba del cálculo (gravado, exonerado, inafecto, mixto, con descuentos y redondeos límite) pasa y es **aceptado por SUNAT beta**.
- **CE-002:** Ante una caída simulada de SUNAT, el 100 % de los comprobantes pendientes termina en un estado definitivo al restablecerse, sin duplicados.
- **CE-003:** Un vendedor emite una boleta de 3 productos en menos de 45 segundos desde el celular.

## Fuera de alcance

- Ambiente de producción `[A-20]`.
- Notas de crédito y débito, comunicación de baja y resumen diario `[A-22]`.
- ISC, ICBPER, detracciones, retenciones, percepciones, anticipos, moneda extranjera y ventas a crédito `[A-22]`.
- Guías de remisión.
- Envío del comprobante al cliente por correo o WhatsApp.
- Convertir tickets internos en comprobantes (según A-17).
- Consulta de RUC o DNI en servicios externos para autocompletar datos del cliente.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-09-26 | Spec creada a partir de los puntos 6 y 7 del MVP de la visión | — |
