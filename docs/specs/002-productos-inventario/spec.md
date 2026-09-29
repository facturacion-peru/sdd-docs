# Spec 002: Productos, lotes y movimientos de inventario

**Estado:** Aprobada · **Versión:** 1.0 · **Fecha:** 2026-09-28
**Depende de:** 001 · **Aclaraciones:** A-12 a A-16, A-26 a A-28

> Las marcas `[A-xx]` indican la aclaración de la que sale cada requisito; todas están resueltas (ver *Clarificaciones*).

## Objetivo

Que cada empresa sepa qué vende, cuánto tiene y de dónde salió cada unidad. Las entradas se registran por lote y cada cambio de stock deja un movimiento auditable. El stock es un dato operativo crítico, no una aproximación.

## Historias de usuario

### HU-1 · Catálogo de productos (P1)

Como administrador de empresa, quiero registrar mis productos y servicios con su precio y su tratamiento de IGV, para venderlos y facturarlos correctamente.

**Prueba independiente:** crear un bien y un servicio, editarlos, desactivar uno y comprobar que no aparece para vender.

**Escenarios de aceptación**

1. **Dado** que soy administrador de empresa, **cuando** registro un producto con código, nombre, tipo (bien o servicio), unidad de medida, precio de venta y afectación al IGV, **entonces** queda disponible en mi catálogo `[A-16]`.
2. **Dado** un código que ya existe en mi empresa, **cuando** registro otro producto con él, **entonces** el sistema lo rechaza. El mismo código sí puede existir en otra empresa.
3. **Dado** un producto desactivado, **cuando** un vendedor busca productos para vender, **entonces** no aparece; su historial se conserva.
4. **Dado** que soy vendedor, **cuando** consulto el catálogo, **entonces** veo productos, precios y stock, pero **no** el costo, y no puedo crear ni editar productos `[A-27]`.

### HU-2 · Entrada de stock por lote (P1)

Como administrador de empresa, quiero registrar la mercadería que recibo, indicando el lote, para saber qué tengo y cuándo vence.

**Prueba independiente:** registrar dos entradas del mismo producto y comprobar que el stock es la suma y que hay dos lotes.

**Escenarios de aceptación**

1. **Dado** un bien, **cuando** registro una entrada con cantidad, fecha de ingreso, número de lote (opcional), costo (opcional) y referencia (opcional), **entonces** se crea un lote, el stock aumenta y queda un movimiento de entrada con mi usuario `[A-13]`.
2. **Dado** una entrada sin número de lote, **cuando** la registro, **entonces** el sistema le asigna uno automáticamente `[A-13]`.
3. **Dado** un producto que controla vencimiento, **cuando** registro una entrada sin fecha de vencimiento o con una fecha pasada, **entonces** el sistema lo rechaza `[A-13]`.
4. **Dado** un servicio, **cuando** intento registrar una entrada de stock, **entonces** el sistema lo impide.
5. **Dado** una cantidad cero o negativa, **cuando** intento registrarla, **entonces** el sistema lo rechaza.

### HU-3 · Salida de stock al vender (P1)

Como vendedor, quiero que al vender se descuente el stock del lote correcto, sin tener que elegirlo.

Esta historia define la **regla**; la venta que la dispara está en las specs 003 y 005.

**Prueba independiente:** con dos lotes de distinta fecha, vender una cantidad que los abarque a ambos y comprobar qué se descontó de cada uno.

**Escenarios de aceptación**

1. **Dado** un producto que controla vencimiento y tiene varios lotes, **cuando** se vende, **entonces** se descuenta primero del lote que vence antes (FEFO) `[A-14]`.
2. **Dado** un producto sin control de vencimiento, **cuando** se vende, **entonces** se descuenta primero del lote que entró antes (FIFO) `[A-14]`.
3. **Dado** una venta mayor que el stock de un lote, **cuando** se procesa, **entonces** se consumen varios lotes y queda un movimiento por cada uno.
4. **Dado** una cantidad mayor que el stock disponible, **cuando** se intenta vender, **entonces** la venta se bloquea e indica el disponible `[A-15]`.
5. **Dado** lotes vencidos, **cuando** se calcula el disponible para vender, **entonces** no cuentan.

### HU-4 · Ajustes y correcciones (P2)

Como administrador de empresa, quiero corregir el stock por conteo, merma o error, dejando constancia del motivo.

**Escenarios de aceptación**

1. **Dado** una diferencia de inventario, **cuando** registro un ajuste positivo o negativo sobre un lote con su motivo (conteo, merma, daño, vencimiento, otro), **entonces** el stock cambia y queda un movimiento de ajuste con el motivo y mi usuario.
2. **Dado** un movimiento registrado por error, **cuando** lo revierto indicando el motivo, **entonces** se crea un movimiento inverso enlazado al original; el original **nunca** se borra ni se edita.
3. **Dado** un ajuste negativo que dejaría el lote por debajo de cero, **cuando** lo intento, **entonces** el sistema lo rechaza.
4. **Dado** que soy vendedor, **cuando** intento ajustar o revertir, **entonces** el sistema lo impide.

### HU-5 · Historial y alertas (P2)

Como administrador de empresa, quiero ver el historial de cada producto y enterarme de lo que se acaba o vence.

**Escenarios de aceptación**

1. **Dado** un producto, **cuando** consulto su historial, **entonces** veo cada movimiento con fecha, tipo, lote, cantidad, stock resultante, responsable y documento de origen.
2. **Dado** un producto con stock mínimo definido, **cuando** su stock disponible queda en el mínimo o por debajo, **entonces** aparece en la lista de alertas de stock bajo.
3. **Dado** un lote que vence dentro del plazo de aviso de la empresa, **cuando** consulto las alertas, **entonces** aparece como próximo a vencer.

## Casos límite

- Dos ventas simultáneas del último stock: solo una prospera; la otra recibe el error de stock insuficiente.
- Cambiar un producto de "bien" a "servicio" con stock existente: no se permite mientras tenga stock.
- Activar "controla vencimiento" en un producto con lotes sin fecha: los lotes existentes siguen sin fecha y se tratan como los de salida más tardía.
- Desactivar un producto con stock: se permite y se avisa del stock que queda.
- Las cantidades admiten decimales solo si la unidad lo permite (kilos sí, unidades no).

## Requisitos funcionales

**Productos**

- **RF-001:** El sistema DEBE gestionar productos con código único por empresa, nombre, tipo (bien o servicio), unidad de medida del catálogo SUNAT, precio de venta con IGV incluido, afectación al IGV, stock mínimo opcional, opción "controla vencimiento" y estado `[A-16]`.
- **RF-002:** Los productos NO DEBEN borrarse si tienen movimientos o ventas; solo desactivarse.

**Lotes y movimientos**

- **RF-010:** Toda entrada de stock DEBE crear un lote con cantidad, fecha de ingreso, número (propio o generado), vencimiento si aplica, costo opcional y referencia opcional `[A-13]`.
- **RF-011:** Todo cambio de stock DEBE registrarse como movimiento inmutable con tipo, lote, cantidad, stock resultante, responsable, fecha, motivo y documento de origen si existe.
- **RF-012:** Los tipos de movimiento DEBEN ser: entrada, salida por venta, ajuste, devolución y reversión.
- **RF-013:** El stock de un producto DEBE ser igual a la suma de sus movimientos; no se guarda un valor editable a mano.
- **RF-014:** Las salidas DEBEN asignar los lotes automáticamente: FEFO si el producto controla vencimiento, FIFO si no `[A-14]`.
- **RF-015:** El sistema NO DEBE permitir stock negativo en ningún lote `[A-15]`.
- **RF-016:** Las correcciones DEBEN hacerse con movimientos de reversión o ajuste; los movimientos no se editan ni se borran.
- **RF-017:** La operación de descontar stock DEBE ofrecerse a otras capacidades (003, 005) de forma que la venta y el descuento se confirmen juntos o fallen juntos.

**Permisos y alertas**

- **RF-020:** Solo el administrador de empresa DEBE poder crear y editar productos, registrar entradas, ajustes y reversiones. El vendedor solo consulta `[A-28]`.
- **RF-022:** El costo de productos y lotes solo DEBE mostrarse al administrador de empresa; las respuestas para el vendedor no lo incluyen `[A-27]`.
- **RF-021:** El sistema DEBE listar productos con stock bajo y lotes próximos a vencer, con un plazo de aviso configurable por empresa (30 días por defecto).

## Entidades clave

- **Producto:** qué se vende, cómo se mide, cuánto cuesta y cómo tributa.
- **Lote:** unidades de un producto que entraron juntas, con su vencimiento y su costo.
- **Movimiento de inventario:** cambio de stock inmutable, enlazado a su lote, su responsable y su documento de origen.

## Criterios de éxito

- **CE-001:** Para cualquier producto, el stock mostrado coincide con la suma de sus movimientos (verificado por prueba).
- **CE-002:** Ninguna combinación de operaciones concurrentes deja un lote en negativo (verificado por prueba de concurrencia).
- **CE-003:** Registrar una entrada de stock toma menos de 30 segundos desde el celular.

## Fuera de alcance

- Varios almacenes y transferencias entre ellos `[A-12]`.
- Conversión entre unidades de medida y presentaciones (caja, blister, unidad).
- Elección manual del lote al vender.
- Compras, proveedores y órdenes de compra; costeo promedio o valorización contable.
- Variantes (talla, color), combos y recetas de producción.
- Importación masiva de productos `[A-26]`.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-09-28 | A-27 · Costo | Solo lo ve el administrador (HU-1.4, RF-022) |
| 2026-09-28 | A-28 · Entradas | Solo el administrador registra entradas, ajustes y reversiones (RF-020 confirmado) |
| 2026-09-28 | **Aprobación** | Spec aprobada por el usuario (v1.0) |
| 2026-09-28 | A-12 a A-16 y A-26 | Recomendaciones aceptadas: stock único; lote siempre y vencimiento por producto; salida automática FEFO/FIFO; sin stock negativo; modelo de producto tal cual (precio con IGV, afectación obligatoria); sin importación masiva en el MVP |
| 2026-09-26 | Se elimina la alerta "una operación inmoviliza más stock del normal" del borrador | No era verificable; se reintroducirá con una definición medible si hace falta |
