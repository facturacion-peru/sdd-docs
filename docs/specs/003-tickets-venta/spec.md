# Spec 003: Tickets de venta internos

**Estado:** En aclaración · **Versión:** 0.2 · **Fecha:** 2026-09-26
**Depende de:** 001, 002 · **Aclaraciones:** A-09, A-15, A-17, A-18, A-19

> Los requisitos marcados con `[A-xx]` siguen la recomendación de esa aclaración y se ajustarán si la decisión es otra.
> ⚖️ Esta spec **no puede aprobarse** sin resolver [A-17](../../aclaraciones.md#a-17): define para qué sirve legalmente el ticket.

## Objetivo

Que una empresa registre ventas rápidas y entregue un **ticket interno** con numeración propia, descontando stock, sin conectarse a SUNAT. El ticket **nunca** debe poder confundirse con un comprobante de pago.

## Historias de usuario

### HU-1 · Registrar una venta con ticket (P1)

Como vendedor, quiero registrar una venta en pocos toques y entregar un ticket, para atender rápido.

**Prueba independiente:** vender dos productos, ver el ticket y comprobar el stock y el total.

**Escenarios de aceptación**

1. **Dado** productos con stock, **cuando** agrego productos, cantidades, un descuento opcional por línea y el medio de pago, y confirmo, **entonces** se crea un ticket con el siguiente número de mi empresa, fecha, hora, vendedor, líneas y total, y el stock se descuenta según la 002 `[A-19]`.
2. **Dado** una venta sin cliente, **cuando** la confirmo, **entonces** el ticket sale a nombre de "Cliente varios" `[A-18]`.
3. **Dado** una línea con más cantidad que el stock disponible, **cuando** confirmo, **entonces** la venta completa se rechaza, se indica qué producto falta y no se descuenta nada `[A-15]`.
4. **Dado** un servicio, **cuando** lo vendo, **entonces** se incluye en el ticket sin tocar el stock.
5. **Dado** un descuento mayor que el importe de la línea, **cuando** confirmo, **entonces** el sistema lo rechaza.

### HU-2 · Ver e imprimir el ticket (P1)

Como vendedor, quiero mostrar o imprimir el ticket, para entregárselo al cliente.

**Escenarios de aceptación**

1. **Dado** un ticket emitido, **cuando** lo abro, **entonces** veo una vista de 80 mm que puedo imprimir o guardar como PDF desde el navegador `[A-18]`.
2. **Dado** cualquier ticket, **cuando** se muestra o imprime, **entonces** incluye de forma visible la leyenda **"Documento interno — no es comprobante de pago"**, y **no** usa las palabras "boleta" ni "factura", ni series SUNAT, ni código QR de SUNAT `[A-17]`.
3. **Dado** un ticket anulado, **cuando** se muestra, **entonces** aparece marcado como **ANULADO**.

### HU-3 · Anular un ticket (P2)

Como administrador de empresa, quiero anular un ticket emitido por error, para que el stock y los totales sean correctos `[A-18]`.

**Escenarios de aceptación**

1. **Dado** un ticket emitido, **cuando** lo anulo indicando el motivo, **entonces** queda en estado anulado, conserva su número y el stock vuelve con movimientos de reversión enlazados.
2. **Dado** que soy vendedor, **cuando** intento anular un ticket, **entonces** el sistema lo impide.
3. **Dado** un ticket ya anulado, **cuando** intento anularlo otra vez, **entonces** el sistema lo rechaza.

### HU-4 · Consultar ventas (P2)

Como administrador de empresa, quiero listar los tickets por fecha, vendedor, estado y medio de pago, con sus totales, para controlar el día.

**Escenarios de aceptación**

1. **Dado** tickets de varios días, **cuando** filtro por un rango de fechas, **entonces** veo los tickets y el total por medio de pago, sin contar los anulados.
2. **Dado** que soy vendedor, **cuando** consulto tickets, **entonces** solo veo los míos.

## Casos límite

- Dos vendedores confirman ventas a la vez: los números no se repiten ni se saltan.
- Se cae la conexión al confirmar: reintentar no debe crear un ticket duplicado.
- El precio del producto cambia después de emitir: el ticket conserva el precio con el que se vendió.
- Se desactiva un producto: los tickets emitidos lo siguen mostrando.

## Requisitos funcionales

- **RF-001:** El sistema DEBE crear tickets con líneas (producto, cantidad, precio unitario, descuento, importe), total, medio de pago, cliente opcional, vendedor, fecha y hora.
- **RF-002:** Los tickets DEBEN numerarse con una secuencia propia por empresa, sin huecos ni duplicados, con un formato que no pueda confundirse con una serie SUNAT (p. ej. `T-000001`) `[A-18]`.
- **RF-003:** El ticket y su descuento de stock DEBEN confirmarse juntos o fallar juntos (spec 002, RF-017).
- **RF-004:** Todo ticket DEBE mostrar la leyenda "Documento interno — no es comprobante de pago" y NO DEBE usar denominaciones, series ni QR de comprobantes SUNAT `[A-17]`.
- **RF-005:** Los tickets NO DEBEN enviarse a SUNAT ni generar XML.
- **RF-006:** Los tickets emitidos NO DEBEN editarse. Solo pueden anularse, con motivo, por un administrador de empresa, reponiendo el stock `[A-18]`.
- **RF-007:** El importe de cada línea y el total DEBEN calcularlos el servidor. El frontend puede mostrar una vista previa, pero el valor guardado es el del servidor (principio II).
- **RF-008:** Confirmar la misma venta dos veces (reintento) NO DEBE crear dos tickets.
- **RF-009:** El ticket guarda los precios y datos de los productos vigentes al momento de la venta.
- **RF-010:** Si se registra un cliente, el ticket guarda su nombre y, opcionalmente, su documento.

## Entidades clave

- **Ticket:** venta interna numerada, con su estado (emitido, anulado), su vendedor, su medio de pago y su total.
- **Línea de ticket:** producto vendido con cantidad, precio y descuento de ese momento.
- **Secuencia de tickets:** numeración propia de cada empresa.

## Criterios de éxito

- **CE-001:** Un vendedor registra una venta de 3 productos en menos de 30 segundos desde el celular.
- **CE-002:** Ningún ticket, en pantalla, impreso o en PDF, carece de la leyenda de documento no tributario (verificado por prueba).
- **CE-003:** Con ventas concurrentes, la numeración queda sin huecos ni duplicados (verificado por prueba).

## Fuera de alcance

- Caja: apertura, cierre y arqueo `[A-19]`.
- Convertir un ticket en boleta o factura: spec posterior, según [A-17](../../aclaraciones.md#a-17).
- Impresión Bluetooth y ESC/POS.
- Pagos mixtos, vuelto, propinas, ventas a crédito y devoluciones parciales.
- Impuestos desglosados en el ticket. Muestra el total con IGV incluido, porque no es comprobante.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-09-26 | La tarea del borrador "definir si el ticket es fiscal o no" | Ya decidido en la visión: es **no tributario** |
