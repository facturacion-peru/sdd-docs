# Spec 005: Emisión y consulta de comprobantes en beta

**Estado:** Aprobada (2026-09-29) · **Versión:** 1.0 · **Fecha:** 2026-09-29
**Depende de:** 001, 002, 003, 004 · **Aclaraciones:** A-20, A-22 ⚖️, A-23, A-31, A-32, A-33, A-34

> ⚖️ Los casos tributarios de esta spec (tipos de operación, tope de S/ 700 para identificar al comprador de una boleta) son una decisión de trabajo y deben revisarse con un contador antes de producción ([A-22](../../aclaraciones.md#a-22)).

## Objetivo

Que un vendedor emita una **factura o boleta electrónica** en el ambiente beta de SUNAT desde la pantalla «Vender», conozca su resultado (aceptada, aceptada con observaciones o rechazada) y pueda descargar el PDF, el XML y el CDR. Si SUNAT no responde, el comprobante no se pierde: queda pendiente y se envía después.

## Historias de usuario

### HU-1 · Emitir una boleta (P1)

Como vendedor, quiero emitir una boleta a un consumidor desde «Vender», para entregarle un comprobante válido.

**Prueba independiente:** emitir una boleta en beta con productos gravados y exonerados y obtener el CDR aceptado.

**Escenarios de aceptación**

1. **Dado** una empresa con la configuración SUNAT validada y una serie de boleta activa, **cuando** armo la venta en «Vender», elijo «Boleta» y confirmo, **entonces** el sistema calcula los importes, asigna el siguiente correlativo, descuenta el stock, genera y firma el XML y lo envía a SUNAT beta automáticamente (A-23, A-33).
2. **Dado** una boleta sin cliente identificado, **cuando** el total es de S/ 700 o menos, **entonces** se emite a «Cliente varios»; si lo supera, el sistema exige el documento del comprador (DNI, carné de extranjería o RUC) ⚖️.
3. **Dado** una boleta aceptada, **cuando** la abro, **entonces** veo su estado, el mensaje de SUNAT y puedo descargar el PDF (A4 u 80 mm), el XML y el CDR.

### HU-2 · Emitir una factura (P1)

Como vendedor, quiero emitir una factura a una empresa cliente, para que pueda sustentar su gasto.

**Escenarios de aceptación**

1. **Dado** un cliente con RUC válido, **cuando** emito una factura, **entonces** se sigue el mismo flujo que la boleta, con la serie de factura.
2. **Dado** un cliente sin RUC, **cuando** intento emitirle una factura, **entonces** el sistema lo impide y sugiere una boleta.
3. **Dado** una empresa en Nuevo RUS, **cuando** intento emitir una factura, **entonces** el sistema lo impide (A-11).

### HU-3 · Cálculo correcto de importes (P1)

Como empresa, quiero que los importes y el IGV de mis comprobantes sean exactos, para no tener problemas con SUNAT.

**Escenarios de aceptación**

1. **Dado** productos gravados con precio con IGV incluido (A-16), **cuando** se emite, **entonces** el valor de venta, el IGV al 18 % y el total cumplen las reglas de redondeo de SUNAT y el comprobante es aceptado.
2. **Dado** una mezcla de productos gravados, exonerados e inafectos, **cuando** se emite, **entonces** cada total se informa por separado y el total general cuadra.
3. **Dado** descuentos por línea (A-29), **cuando** se emite, **entonces** se reflejan correctamente en la base imponible.
4. **Dado** el mismo conjunto de líneas, **cuando** se calcula varias veces, **entonces** el resultado es siempre idéntico, y la vista previa de «Vender» coincide con lo emitido.

### HU-4 · Recuperar comprobantes pendientes (P1)

Como vendedor, quiero que si SUNAT no responde el comprobante quede pendiente y se envíe después, para no perder la venta ni el número (A-23).

**Escenarios de aceptación**

1. **Dado** SUNAT sin responder, **cuando** emito, **entonces** la venta y el comprobante quedan guardados con su número en estado «pendiente de envío», y se me informa.
2. **Dado** un comprobante pendiente, **cuando** SUNAT vuelve a responder, **entonces** el sistema lo reenvía automáticamente con espera creciente entre intentos.
3. **Dado** un comprobante pendiente, **cuando** pulso «Reintentar», **entonces** se reenvía de inmediato.
4. **Dado** un comprobante que SUNAT rechaza, **cuando** llega la respuesta, **entonces** queda «rechazado» con el código y el motivo de SUNAT, y no se reintenta.
5. **Dado** un comprobante reenviado que SUNAT ya había recibido, **cuando** responde que está duplicado, **entonces** el sistema obtiene su estado real en lugar de marcarlo como error.

### HU-5 · Clientes (P1)

Como vendedor, quiero buscar o registrar al cliente mientras vendo, para no salir de «Vender» (A-34).

**Escenarios de aceptación**

1. **Dado** un cliente ya registrado, **cuando** escribo su documento o su nombre en «Vender», **entonces** lo encuentro y lo selecciono.
2. **Dado** un cliente nuevo, **cuando** ingreso tipo y número de documento, nombre o razón social y, opcionalmente, dirección, **entonces** queda registrado en mi empresa; un DNI debe tener 8 dígitos y un RUC 11 con dígito verificador válido.
3. **Dado** que soy vendedor, **cuando** intento editar un cliente, **entonces** no puedo; lo hace el administrador.
4. **Dado** un cliente que cambia sus datos después de emitir, **cuando** abro un comprobante anterior, **entonces** veo los datos que tenía al emitirse.

### HU-6 · Consultar comprobantes (P2)

Como usuario de empresa, quiero listar los comprobantes por fecha, tipo, estado y cliente, para hacer seguimiento.

**Escenarios de aceptación**

1. **Dado** comprobantes en varios estados, **cuando** filtro por «pendiente» o «rechazado», **entonces** veo solo esos, para actuar sobre ellos.
2. **Dado** que soy vendedor, **cuando** consulto comprobantes, **entonces** veo **todos** los de mi empresa y puedo reintentar el envío de cualquiera (A-32).

## Casos límite

- El stock se agota entre que armo la venta y la confirmo: no se emite nada ni se consume correlativo.
- Falla la firma (certificado corrupto o vencido): no se consume correlativo y el error se explica.
- La configuración SUNAT deja de estar validada, o la serie se desactiva, mientras armo la venta: no se emite y se dice por qué.
- SUNAT acepta con observaciones: el estado es «aceptado con observaciones» y se muestran.
- Dos vendedores emiten en la misma serie a la vez: números consecutivos, sin repetir.
- El usuario pulsa «Emitir» dos veces o la red se corta y reintenta: un solo comprobante (misma clave de idempotencia que los tickets).
- El reintento del usuario coincide con el reintento automático: se envía una sola vez.

## Requisitos funcionales

**Emisión**

- **RF-001:** El sistema DEBE emitir facturas y boletas electrónicas solo en el ambiente beta (A-20, A-22).
- **RF-002:** Solo DEBE permitirse emitir con la configuración SUNAT «validada» y una serie activa del tipo correspondiente (spec 004).
- **RF-003:** Todos los importes DEBEN calcularse en el servidor según las reglas de SUNAT, para operaciones gravadas (IGV 18 %), exoneradas e inafectas, en soles y al contado con los medios de pago de A-19 (A-22).
- **RF-004:** La venta, el correlativo, el comprobante y el descuento de stock DEBEN confirmarse juntos o fallar juntos. El envío a SUNAT ocurre **después** de confirmar y no lo deshace.
- **RF-005:** El correlativo solo DEBE consumirse cuando el comprobante queda guardado.
- **RF-006:** El comprobante DEBE conservar los datos del emisor, del cliente y de los productos vigentes al emitirse.
- **RF-007:** El sistema DEBE generar el PDF con el código QR exigido por SUNAT en formatos A4 y 80 mm.
- **RF-008:** La boleta DEBE exigir el documento del comprador si el total supera S/ 700; la factura DEBE exigir un cliente con RUC ⚖️.
- **RF-009:** La emisión DEBE hacerse desde «Vender», eligiendo Ticket, Boleta o Factura, con la misma clave de idempotencia que los tickets (A-33).

**Envío y estados**

- **RF-010:** Los estados del comprobante DEBEN ser: pendiente de envío, enviado, aceptado, aceptado con observaciones y rechazado.
- **RF-011:** El envío DEBE ser automático al emitir, con reintentos automáticos con espera creciente si SUNAT no responde, y un reintento manual disponible (A-23).
- **RF-012:** Un mismo comprobante NO DEBE enviarse dos veces a la vez.
- **RF-013:** El sistema DEBE guardar el XML firmado, el CDR y el código y mensaje de respuesta de SUNAT, y cada intento de envío.
- **RF-014:** Los comprobantes emitidos NO DEBEN editarse ni borrarse. En beta no se anulan (A-22).

**Visibilidad**

- **RF-020:** Los comprobantes de beta DEBEN mostrarse siempre marcados como «PRUEBAS — sin valor legal», también en el PDF.
- **RF-021:** El sistema DEBE permitir listar comprobantes por fecha, tipo, estado y cliente, y descargar PDF, XML y CDR. Todos los usuarios de la empresa ven todos sus comprobantes (A-32).
- **RF-022:** DEBEN auditarse la emisión, los reintentos y los cambios de estado.

**Clientes**

- **RF-030:** El sistema DEBE permitir registrar clientes de la empresa con tipo y número de documento (DNI, RUC, carné de extranjería), nombre o razón social y dirección opcional, validando el formato del documento. Documento único por empresa.
- **RF-031:** Cualquier usuario de la empresa DEBE poder buscar y registrar clientes; solo el administrador DEBE poder editarlos. Los clientes no se borran (A-34).

## Entidades clave

- **Comprobante:** factura o boleta con serie, correlativo, emisor, cliente, medio de pago, totales por tipo de afectación, estado, XML, CDR y respuesta de SUNAT.
- **Línea de comprobante:** producto, cantidad, valor unitario, precio, descuento, afectación al IGV e importes, tal como se emitió.
- **Cliente:** comprador identificado de la empresa.
- **Intento de envío:** cada envío a SUNAT, con fecha, resultado y error si lo hubo.

## Criterios de éxito

- **CE-001:** Un conjunto de casos de prueba del cálculo (gravado, exonerado, inafecto, mixto, con descuentos y redondeos límite) pasa y es **aceptado por SUNAT beta**.
- **CE-002:** Ante una caída simulada de SUNAT, el 100 % de los comprobantes pendientes termina en un estado definitivo al restablecerse, sin duplicados.
- **CE-003:** Un vendedor emite una boleta de 3 productos en menos de 45 segundos desde el celular.

## Fuera de alcance

- Ambiente de producción (A-20).
- Notas de crédito y débito, comunicación de baja, resumen diario y anulación de comprobantes (A-22).
- ISC, ICBPER, detracciones, retenciones, percepciones, anticipos, moneda extranjera y ventas a crédito (A-22).
- Guías de remisión.
- Envío del comprobante al cliente por correo o WhatsApp.
- Convertir tickets internos en comprobantes (A-17: spec posterior).
- Consulta de RUC o DNI en servicios externos para autocompletar datos del cliente.
- Borrar clientes.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-09-26 | Spec creada a partir de los puntos 6 y 7 del MVP de la visión | — |
| 2026-09-29 | A-22 ⚖️ | Factura y boleta; gravado, exonerado e inafecto; soles y al contado; «Cliente varios» hasta S/ 700; sin notas, baja ni resumen diario |
| 2026-09-29 | A-23 | Envío automático; pendiente con reintentos crecientes y manual; rechazado sin reintentos |
| 2026-09-29 | A-32 | El vendedor ve todos los comprobantes de su empresa |
| 2026-09-29 | A-33 | Se emite desde «Vender» eligiendo Ticket, Boleta o Factura |
| 2026-09-29 | A-34 | El vendedor busca y registra clientes; editarlos es del administrador; no se borran |
| 2026-09-29 | Aprobación | Spec aprobada por el usuario (v1.0) |
