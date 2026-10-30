# Spec 017: Varios establecimientos

**Estado:** Aprobada (2026-10-29) · **Versión:** 1.0 · **Fecha:** 2026-10-29
**Depende de:** 001, 002, 003, 004, 005, 007, 011, 012, 014 · **Aclaraciones:** A-86 a A-91 (resueltas)

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que una empresa con **más de un local** (por ejemplo, una bodega y su sucursal) los registre y trabaje con cada uno por separado: cada venta sale del local donde se hace, con su dirección y su serie en el comprobante, su stock y sus propias cifras. Hasta ahora cada empresa tiene un solo establecimiento, el principal (A-07).

## Historias de usuario

### HU-1 · Registrar los locales de la empresa (P1)

Como administrador de empresa, quiero registrar mis otros locales con el código de anexo que tienen en SUNAT, para emitir desde cada uno.

**Por qué esta prioridad:** sin locales registrados, nada de lo demás aplica.
**Prueba independiente:** creo un local con su código de anexo, le creo una serie de boleta y emito una boleta con esa serie; el comprobante lleva la dirección y el código del local.

**Escenarios de aceptación**

1. **Dado** una empresa con solo el principal, **cuando** creo un local con código de anexo, nombre, dirección y ubigeo (A-89), **entonces** aparece en la lista de locales como activo.
2. **Dado** un código de anexo que ya usa otro local de la empresa, o que no tiene 4 dígitos, **cuando** lo guardo, **entonces** veo un mensaje claro y no se crea.
3. **Dado** un local, **cuando** edito su nombre o dirección, **entonces** los comprobantes nuevos llevan los datos nuevos y los ya emitidos no cambian.
4. **Dado** un local con ventas, series o stock, **cuando** lo desactivo, **entonces** deja de ofrecerse para vender y para asignar usuarios, y su historial sigue visible. El principal no se puede desactivar (A-89).
5. **Dado** un local, **cuando** creo una serie, **entonces** elijo a qué local pertenece (hoy siempre es el principal).
6. **Dado** que soy vendedor, **cuando** abro «Mi empresa», **entonces** no puedo crear, editar ni desactivar locales.

### HU-2 · Vender desde el local donde estoy (P1)

Como vendedor, quiero que mis ventas salgan del local donde trabajo, con su serie y su stock, sin tener que pensarlo en cada venta.

**Por qué esta prioridad:** es el uso diario; un error aquí emite con la dirección o la serie de otro local.
**Prueba independiente:** con dos locales y un vendedor asignado solo al segundo, vendo una boleta; sale con la serie y la dirección del segundo local y descuenta su stock.

**Escenarios de aceptación**

1. **Dado** un vendedor asignado a un solo local (A-87), **cuando** abre «Vender», **entonces** ve el nombre de su local y solo puede usar las series de ese local.
2. **Dado** un usuario que puede trabajar en varios locales, **cuando** entra, **entonces** elige el local y la app lo recuerda en ese dispositivo; puede cambiarlo, y el cambio no mezcla la venta en curso de un local con otro (spec 012).
3. **Dado** una venta con ticket, boleta o factura, **cuando** se cobra, **entonces** queda registrada en el local actual y el ticket o comprobante lo muestra.
4. **Dado** stock por local (A-86), **cuando** vendo, **entonces** el catálogo muestra y descuenta el stock de mi local, y una nota de crédito con devolución lo repone en el local de la venta original.
5. **Dado** un local desactivado, **cuando** un usuario tenía ese local recordado, **entonces** la app le pide elegir otro y no le deja vender en el desactivado.

### HU-3 · Mover mercadería entre locales (P1)

Como administrador de empresa, quiero registrar entradas en un local y trasladar mercadería de un local a otro, para que el stock de cada uno sea real.

**Por qué esta prioridad:** con stock por local (A-86), sin traslados no hay forma de abastecer la sucursal.
**Prueba independiente:** registro una entrada de 10 unidades en el principal y traslado 4 al segundo local; el principal queda con 6 y el segundo con 4, con los dos movimientos en el historial.

**Escenarios de aceptación**

1. **Dado** una entrada de mercadería, **cuando** la registro, **entonces** elijo el local donde entra.
2. **Dado** stock suficiente en el local de origen, **cuando** traslado una cantidad a otro local, **entonces** baja en el origen y sube en el destino a la vez, conservando lote y vencimiento.
3. **Dado** stock insuficiente en el origen, **cuando** traslado, **entonces** veo un mensaje y nada cambia.
4. **Dado** un traslado, **cuando** veo los movimientos, **entonces** aparece como salida en el origen y entrada en el destino, con quién y cuándo lo hizo.

### HU-4 · Ver cifras y listas por local (P2)

Como administrador de empresa, quiero ver las ventas, el stock y las alertas de cada local o de todos juntos, para saber cómo va cada uno.

**Por qué esta prioridad:** útil desde el primer día con dos locales, pero vender bien (HU-2) va primero.
**Prueba independiente:** con ventas en dos locales, el inicio filtrado por cada local suma sus ventas, y «Todos» suma ambas.

**Escenarios de aceptación**

1. **Dado** varios locales, **cuando** abro el inicio, las listas de tickets y comprobantes, el stock o los movimientos, **entonces** puedo filtrar por un local o ver «Todos» (A-90).
2. **Dado** un vendedor con un solo local, **cuando** abre esas pantallas, **entonces** ve solo las de su local, sin filtro.
3. **Dado** que exporto ventas o productos (spec 014), **cuando** abro el archivo, **entonces** cada fila indica su local.
4. **Dado** una empresa con un solo local activo, **cuando** usa la app, **entonces** no ve selectores ni filtros de local: todo se ve como hoy (A-91).

## Casos límite

- **Empresas existentes:** todo lo anterior a esta spec queda en el principal (A-91), sin que la empresa haga nada.
- **Código de anexo inexistente en SUNAT:** la app no lo puede comprobar; SUNAT observará o rechazará el comprobante y se verá como hoy (spec 005). El aviso al crear el local lo advierte (A-89 ⚖️).
- **Ventas concurrentes en dos locales** del mismo producto: cada una descuenta de su local sin bloquear a la otra; en el mismo local, ninguna deja el stock en negativo (A-15), como hoy.
- **Traslado concurrente** con una venta en el local de origen: nunca queda más stock del que había ni se pierde ninguna de las dos operaciones.
- **Usuario sin local activo asignado:** no puede vender y ve un mensaje para pedir que lo asignen.
- **Nota de crédito** de un comprobante de otro local: usa la serie de nota del local del comprobante original y repone stock allí.
- **Venta en curso guardada** (spec 012) de un local que se cambió o desactivó: no se cobra en otro local sin que el usuario lo vea.
- **Aislamiento:** una empresa nunca ve ni usa locales de otra (principio VIII); un vendedor no ve ventas ni stock de locales que no tiene asignados (A-87).
- **Panel de la plataforma** (spec 006): muestra cuántos locales tiene cada empresa, sin datos de venta por local.
- **Impresión** (specs 013 y 015): el ticket y el PDF de 80 mm llevan el nombre y la dirección del local.
- **Pantallas angostas:** a 360 px, el nombre del local y su selector caben sin desbordar.

## Requisitos funcionales

- **RF-001:** El administrador de empresa DEBE poder crear, editar y desactivar locales con código de anexo único en la empresa, nombre, dirección y ubigeo (A-89). El principal no se desactiva.
- **RF-002:** Cada serie DEBE pertenecer a un local, y el comprobante DEBE llevar el código de anexo y la dirección de ese local.
- **RF-003:** Cada ticket, comprobante y nota DEBE quedar registrado en un local.
- **RF-004:** El sistema DEBE decidir en qué locales puede trabajar cada usuario según A-87, y la API DEBE rechazar una venta en un local no permitido o inactivo.
- **RF-005:** El stock, los lotes, las alertas y los movimientos DEBEN ser de cada local (A-86), y las entradas y traslados DEBEN indicarlo. Un traslado DEBE ser atómico.
- **RF-006:** La numeración de tickets DEBE seguir A-88; la de comprobantes sigue por serie, sin cambios.
- **RF-007:** El inicio, las listas y la exportación DEBEN poder filtrarse por local según A-90.
- **RF-008:** Crear, editar o desactivar un local, asignar usuarios y trasladar stock DEBEN quedar en la auditoría (principio VIII).
- **RF-009:** Los datos existentes DEBEN quedar en el principal al instalar la versión, sin pérdida ni cambio de cifras (A-91).
- **RF-010:** Con un solo local activo, la app NO DEBE mostrar selectores ni filtros de local.

## Entidades clave

- **Establecimiento (local):** un local de la empresa registrado en SUNAT, con código de anexo, nombre, dirección, ubigeo y si está activo. Ya existe; hoy siempre es el principal.
- **Asignación de usuario a local:** en qué locales puede trabajar cada usuario (A-87).
- **Stock por local** (A-86): la cantidad de cada producto y lote en cada local.
- **Traslado:** salida de un local y entrada en otro, en una sola operación.

## Criterios de éxito

- **CE-001:** Con dos locales, una boleta emitida desde el segundo es aceptada por SUNAT beta con el código de anexo y la dirección de ese local.
- **CE-002:** Una prueba automática confirma que un vendedor no puede vender, ver ventas ni ver stock de un local que no tiene asignado (A-87).
- **CE-003:** Tras instalar la versión en una base con datos, las cifras del inicio, el stock y los listados de una empresa con un solo local son idénticos a los de antes.
- **CE-004:** Un traslado y una venta simultáneos en el local de origen dejan el stock correcto (prueba concurrente en PostgreSQL).

## Fuera de alcance

- Precios distintos por local.
- Clientes, productos o usuarios propios de cada local (son de la empresa).
- Consultar a SUNAT si el código de anexo existe.
- Cajas, arqueos o turnos por local.
- Pedidos de reposición o aprobación de traslados (el traslado es inmediato).
- Almacenes que no venden (solo locales que son establecimientos de SUNAT).
- Límite de locales por plan de cobro.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-29 | Spec creada a pedido del usuario (varios establecimientos, retoma A-07) | — |
| 2026-10-29 | A-86 a A-91 | Stock por local con traslados; usuarios asignados a locales; un correlativo de tickets por empresa; recomendaciones de A-89 a A-91 aceptadas |
| 2026-10-29 | Aprobada (v1.0) | El usuario aprueba la v0.2 sin cambios |
