# Spec 012: Vender — carrito persistente y diseño en dos columnas

**Estado:** Aprobada (2026-10-03) · **Versión:** 1.4 · **Fecha:** 2026-10-03
**Depende de:** 003, 005, 010 · **Aclaraciones:** A-56, A-57, A-58, A-59

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que vender sea más rápido y no se pierda trabajo: la venta en curso se conserva al cambiar de pantalla, recargar o cerrar la aplicación; los productos se agregan con un toque desde el catálogo; y en pantallas anchas el carrito y el total están siempre a la vista junto a los productos.

Hoy el carrito se pierde al salir de «Vender», los productos solo aparecen al escribir una búsqueda y todo va en una sola columna.

## Historias de usuario

### HU-1 · La venta en curso no se pierde (P1)

Como vendedor, quiero que los productos agregados sigan en el carrito si voy a otra pantalla, recargo o cierro la aplicación, para no tener que empezar de nuevo (A-58).

**Por qué esta prioridad:** perder un carrito a medio armar con el cliente delante es la peor experiencia de la pantalla.
**Prueba independiente:** agregar productos, ir al Inicio, recargar y volver a «Vender»: el carrito sigue igual.

**Escenarios de aceptación**

1. **Dado** un carrito con productos, tipo, cliente y medio de pago, **cuando** cambio de pantalla, recargo o cierro y vuelvo a abrir la aplicación, **entonces** «Vender» muestra la misma venta en curso.
2. **Dado** una venta en curso guardada, **cuando** vuelvo, **entonces** los precios y el disponible se actualizan con los del catálogo; si un producto ya no existe o se desactivó, se quita y se me avisa.
3. **Dado** una venta cobrada o emitida, **cuando** termina bien, **entonces** el carrito queda vacío.
4. **Dado** que cierro sesión, **cuando** otra persona entra en el mismo dispositivo, **entonces** no ve mi venta en curso; cada usuario y empresa tiene la suya.
5. **Dado** que mi sesión vence a mitad de una venta, **cuando** vuelvo a entrar, **entonces** la venta en curso sigue ahí (principio VII).
6. **Dado** un cobro que falló por la red, **cuando** reintento tras volver a la pantalla, **entonces** no se crea una venta duplicada.

### HU-2 · Vaciar el carrito (P1)

Como vendedor, quiero vaciar todo el carrito con un botón, para empezar otra venta sin quitar producto por producto.

**Escenarios de aceptación**

1. **Dado** un carrito con productos, **cuando** pulso «Vaciar carrito», **entonces** se me pide confirmar indicando cuántos productos se quitarán (A-59).
2. **Dado** la confirmación, **cuando** acepto, **entonces** el carrito queda vacío y la venta en curso guardada se borra; si cancelo, nada cambia.
3. **Dado** un carrito vacío, **cuando** lo veo, **entonces** el botón no se ofrece.

### HU-3 · Agregar desde el catálogo (P2)

Como vendedor de una bodega con pocos productos, quiero ver el catálogo y agregar con un toque sin escribir, para vender más rápido (A-57).

**Escenarios de aceptación**

1. **Dado** «Vender» sin búsqueda, **cuando** la abro, **entonces** veo los productos activos con precio y disponible, y puedo cargar más.
2. **Dado** una búsqueda, **cuando** escribo, **entonces** el catálogo se filtra; al limpiarla vuelve completo.
3. **Dado** un producto sin disponible, **cuando** lo veo, **entonces** no se puede agregar y se indica por qué.
4. **Dado** un producto ya en el carrito, **cuando** lo toco de nuevo, **entonces** su cantidad aumenta en uno y se ve cuántos llevo.

### HU-4 · Carrito siempre a la vista (P2)

Como vendedor, quiero ver el carrito y el total mientras agrego productos (A-56).

**Escenarios de aceptación**

1. **Dado** una pantalla ancha, **cuando** vendo, **entonces** los productos están a un lado y el carrito, el total y el botón de cobrar al otro, siempre visibles.
2. **Dado** el celular, **cuando** hay productos en el carrito, **entonces** una barra fija abajo muestra cuántos productos y el total, y abre el carrito completo para revisar y cobrar.
3. **Dado** una pantalla ancha, **cuando** vendo, **entonces** «Vender» ocupa todo el ancho y el alto de la ventana: catálogo y carrito se desplazan cada uno por su cuenta y la página no se desplaza (v1.1).
4. **Dado** el carrito, **cuando** elijo qué emitir (ticket, boleta o factura), **entonces** el selector está en el propio carrito, después de los productos y antes del cliente y del medio de pago (orden del cobro: qué llevo, qué emito y a quién, cómo me pagan, cobrar; v1.3), con el aviso de pruebas y el motivo si boleta o factura no están disponibles; en el celular, la barra fija muestra el tipo elegido (v1.2).

### HU-5 · Seguir vendiendo tras cobrar (P1, v1.4)

Como vendedor, quiero que al cobrar se confirme la venta sin salir de «Vender», para atender al siguiente cliente con un toque.

**Por qué esta prioridad:** «Vender» es la pantalla donde se pasa el día; salir a otra pantalla tras cada venta obliga a volver siempre.

**Escenarios de aceptación**

1. **Dado** un cobro o una emisión exitosa, **cuando** termina, **entonces** sigo en «Vender», el carrito queda vacío y un modal muestra «Venta registrada» con el número y el total.
2. **Dado** una boleta o factura, **cuando** se muestra la confirmación, **entonces** indica el resultado de SUNAT; si quedó pendiente de envío o fue rechazada, lo destaca y explica qué pasará.
3. **Dado** el modal, **cuando** pulso «Nueva venta», Escape o fuera, **entonces** se cierra y el foco queda en la búsqueda de productos.
4. **Dado** el modal, **cuando** pulso «Imprimir», **entonces** se imprime el ticket de 80 mm, o se abre el PDF de 80 mm del comprobante.
5. **Dado** el modal, **cuando** pulso «Ver detalle», **entonces** voy al detalle del ticket o comprobante (anular, nota de crédito, descargas).

## Casos límite

- **Almacenamiento no disponible** (navegación privada): la venta en curso dura mientras la aplicación esté abierta, sin errores.
- **Venta guardada de otra versión** o con datos dañados: se descarta en silencio y se empieza vacía.
- **Precio cambiado** desde que se agregó: se muestra el actual (el servidor siempre cobra el del catálogo, spec 003 RF-011).
- **Serie elegida ya inactiva** o SUNAT no disponible al volver: se elige la serie por defecto o se vuelve a ticket, como al abrir la pantalla.

## Requisitos funcionales

- **RF-001:** La venta en curso (líneas, tipo, cliente, medio de pago, serie y clave de idempotencia) DEBE conservarse en el dispositivo por usuario y empresa hasta cobrarla, vaciarla o cerrar sesión.
- **RF-002:** Al restaurarla, el sistema DEBE actualizar precio y disponible de cada producto y quitar los que ya no se pueden vender, avisando.
- **RF-003:** El vencimiento de la sesión NO DEBE borrar la venta en curso; cerrar sesión sí.
- **RF-004:** «Vaciar carrito» DEBE pedir confirmación.
- **RF-005:** «Vender» DEBE mostrar el catálogo de productos activos sin necesidad de buscar, paginado.
- **RF-006:** En pantallas anchas, productos y carrito DEBEN mostrarse en dos columnas, con el carrito fijo a la vista; en el celular, una barra fija con cantidad, total y acceso al carrito.
- **RF-007:** Los importes siguen siendo una vista previa; el servidor calcula y guarda los definitivos (principio II).
- **RF-008:** Tras cobrar o emitir con éxito, «Vender» DEBE quedarse en pantalla y confirmar la venta en un modal con número, total, resultado de SUNAT (si aplica) y las acciones Nueva venta, Imprimir y Ver detalle. Sustituye el paso «va al ticket / al comprobante» de las specs 003 y 005.

## Criterios de éxito

- **CE-001:** Agregar productos, cambiar de pantalla, recargar y volver conserva el carrito (prueba automatizada y verificación manual).
- **CE-002:** Una venta típica de 3 productos del catálogo se cobra sin escribir (solo toques).
- **CE-003:** «Vender» se ve correctamente a 360 px y en escritorio, en claro y en oscuro.

## Fuera de alcance

- Varias ventas en espera a la vez («ventas aparcadas»).
- Sincronizar la venta en curso entre dispositivos.
- Lector de código de barras.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-03 | A-56 Diseño | Dos columnas en pantallas anchas; en el celular, barra fija que abre el carrito |
| 2026-10-03 | A-57 Catálogo | Mostrar el catálogo para tocar; la búsqueda filtra |
| 2026-10-03 | A-58 Persistencia | Toda la venta en curso en el dispositivo, por usuario y empresa |
| 2026-10-03 | A-59 Vaciar | Con confirmación |
| 2026-10-03 | Pedido del usuario (v1.4) | Confirmación en un modal sin salir de «Vender» (HU-5) |
| 2026-10-03 | Pedido del usuario (v1.3) | Dentro del carrito: productos → qué emitir → serie y cliente → medio de pago → total |
| 2026-10-03 | Pedido del usuario (v1.2) | El selector Ticket / Boleta / Factura pasa al carrito |
| 2026-10-03 | Pedido del usuario (v1.1) | «Vender» a pantalla completa en escritorio; barra lateral contraíble (spec 010 v1.4) |
