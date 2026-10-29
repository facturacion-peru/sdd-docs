# Aclaraciones

Registro único de preguntas abiertas y decisiones. Las specs las citan por su identificador (`A-01`) en lugar de repetirlas.

**Cómo se usa**

- Cada pregunta indica qué spec **bloquea**: esa spec no pasa a `Aprobada` hasta que la pregunta esté `Resuelta`.
- La **recomendación** es una propuesta, no una decisión. Decide el usuario.
- Al resolverla: se cambia el estado, se anota la respuesta con fecha y se actualiza la spec afectada (sección *Clarificaciones*).
- Las preguntas legales o tributarias marcadas con ⚖️ deben confirmarse con un contador o con una fuente oficial de SUNAT antes de pasar a producción.

**Estados:** `Abierta` · `Resuelta` · `Descartada`

## Resumen

| ID | Tema | Bloquea | Estado |
|---|---|---|---|
| [A-01](#a-01) | Adaptar o reiniciar la API | 001 | Resuelta |
| [A-02](#a-02) | Datos reales en la API actual | 001 | Resuelta |
| [A-03](#a-03) | Cómo se da de alta una empresa | 001 | Resuelta |
| [A-04](#a-04) | Usuarios en varias empresas | 001 | Resuelta |
| [A-05](#a-05) | Roles del MVP | 001 | Resuelta |
| [A-06](#a-06) | Modelo de aislamiento y volumen esperado | 001 | Resuelta |
| [A-07](#a-07) | Establecimientos en el MVP | 001, 004 | Resuelta |
| [A-08](#a-08) | Dónde administra su empresa el administrador | 001 | Resuelta |
| [A-09](#a-09) | Convivencia de tickets y comprobantes | 001, 003 | Resuelta |
| [A-10](#a-10) | Enmienda de la constitución 1.1.0 | 001 | Resuelta |
| [A-11](#a-11) ⚖️ | Tipos de contribuyente y régimen | 001, 004 | Resuelta |
| [A-12](#a-12) | Almacenes | 002 | Resuelta |
| [A-13](#a-13) | Lotes y vencimientos | 002 | Resuelta |
| [A-14](#a-14) | Política de salida de stock | 002 | Resuelta |
| [A-15](#a-15) | Stock negativo | 002 | Resuelta |
| [A-16](#a-16) | Servicios, unidades, costo e impuestos del producto | 002 | Resuelta |
| [A-17](#a-17) ⚖️ | Propósito legal del ticket interno | 003 | Resuelta |
| [A-18](#a-18) | Contenido, impresión y anulación del ticket | 003 | Resuelta |
| [A-19](#a-19) | Caja y medios de pago | 003 | Resuelta |
| [A-20](#a-20) | Ambientes SUNAT del MVP | 004, 005 | Resuelta |
| [A-21](#a-21) | Series y certificados | 004 | Resuelta |
| [A-22](#a-22) ⚖️ | Comprobantes y casos tributarios del MVP | 005 | Resuelta |
| [A-23](#a-23) | Envío a SUNAT y fallos | 005 | Resuelta |
| [A-24](#a-24) ⚖️ | Retención de documentos y protección de datos | Producción | Abierta |
| [A-25](#a-25) | Recuperación de contraseña | 001 | Resuelta |
| [A-26](#a-26) | Importación masiva de productos | 002 | Resuelta |
| [A-27](#a-27) | Visibilidad del costo | 002 | Resuelta |
| [A-28](#a-28) | Quién registra entradas de mercadería | 002 | Resuelta |
| [A-29](#a-29) | Cambiar el precio al vender | 003 | Resuelta |
| [A-30](#a-30) | Quién registra ventas con ticket | 003 | Resuelta |
| [A-31](#a-31) | Credenciales en el ambiente beta | 004, 005 | Resuelta |
| [A-32](#a-32) | Qué comprobantes ve el vendedor | 005 | Resuelta |
| [A-33](#a-33) | Dónde se emite el comprobante | 005 | Resuelta |
| [A-34](#a-34) | Quién registra clientes | 005 | Resuelta |
| [A-35](#a-35) | Tecnología del panel de la plataforma | 006 | Resuelta |
| [A-36](#a-36) | Alcance del panel | 006 | Resuelta |
| [A-37](#a-37) | Qué ve la plataforma de cada empresa | 006 | Resuelta |
| [A-38](#a-38) | Protección del acceso de la plataforma | 006 | Resuelta |
| [A-39](#a-39) ⚖️ | Instrumentos de corrección | 007 | Resuelta |
| [A-40](#a-40) | Cuándo vuelve el stock | 007 | Resuelta |
| [A-41](#a-41) | Quién emite notas de crédito | 007 | Resuelta |
| [A-42](#a-42) | Comprobantes rechazados | 007 | Resuelta |
| [A-43](#a-43) | Alcance del primer despliegue | 009 | Resuelta |
| [A-44](#a-44) | Alojamiento | 009 | Resuelta |
| [A-45](#a-45) | Copias de seguridad | 009 | Resuelta |
| [A-46](#a-46) | Despliegue y repositorios | 009 | Resuelta |
| [A-47](#a-47) | Correo | 009 | Resuelta |
| [A-48](#a-48) | Monitoreo y avisos | 009 | Resuelta |
| [A-49](#a-49) | Tema visual | 010 | Resuelta |
| [A-50](#a-50) | Tipografía | 010 | Resuelta |
| [A-51](#a-51) | Logo de la aplicación | 010 | Resuelta |
| [A-55](#a-55) | Modo oscuro y fondo | 010 | Resuelta |
| [A-52](#a-52) | Indicadores del inicio | 011 | Resuelta |
| [A-53](#a-53) | Qué ve el vendedor en el inicio | 011 | Resuelta |
| [A-54](#a-54) | Qué suma como venta en el inicio | 011 | Resuelta |
| [A-56](#a-56) | Diseño de «Vender» | 012 | Resuelta |
| [A-57](#a-57) | Catálogo en «Vender» | 012 | Resuelta |
| [A-58](#a-58) | Persistencia de la venta en curso | 012 | Resuelta |
| [A-59](#a-59) | Vaciar el carrito | 012 | Resuelta |
| [A-60](#a-60) | Recordar comprobante y serie | 012 | Resuelta |
| [A-61](#a-61) | Distribución e identidad de la app Android | 013 | Resuelta |
| [A-62](#a-62) | Servidor al que se conecta la app | 013 | Resuelta |
| [A-63](#a-63) | Imprimir y descargar en la app | 013 | Resuelta |
| [A-64](#a-64) | Dónde guarda la sesión la app | 013 | Resuelta |
| [A-65](#a-65) | Panel de la plataforma en la app | 013 | Resuelta |
| [A-66](#a-66) | Uso sin conexión | 013 | Resuelta |
| [A-67](#a-67) | Actualizaciones de la app | 013 | Resuelta |
| [A-68](#a-68) | Qué datos se exportan e importan | 014 | Resuelta |
| [A-69](#a-69) | Formato de los archivos | 014 | Resuelta |
| [A-70](#a-70) | Registros que ya existen al importar | 014 | Resuelta |
| [A-71](#a-71) | Filas con errores al importar | 014 | Resuelta |
| [A-72](#a-72) | Stock inicial al importar productos | 014 | Resuelta |
| [A-73](#a-73) | Quién exporta e importa, y límites | 014 | Resuelta |
| [A-74](#a-74) | Contenido de la exportación de ventas | 014 | Resuelta |
| [A-75](#a-75) | Qué impresoras y qué conexión | 015 | Resuelta |
| [A-76](#a-76) | Ancho de papel | 015 | Resuelta |
| [A-77](#a-77) | Qué se imprime por Bluetooth | 015 | Resuelta |
| [A-78](#a-78) | Cómo se elige la impresora y cuándo imprime | 015 | Resuelta |
| [A-79](#a-79) | Dónde se guardan las fotos y quién puede verlas | 016 | Abierta |
| [A-80](#a-80) | Formatos y tamaño de la foto | 016 | Abierta |
| [A-81](#a-81) | Tamaños guardados, miniatura y datos ocultos | 016 | Abierta |
| [A-82](#a-82) | Qué pasa con la foto al cambiarla, quitarla o desactivar el producto | 016 | Abierta |
| [A-83](#a-83) | Fotos en exportar e importar (014) | 016 | Abierta |
| [A-84](#a-84) | Almacenamiento en desarrollo y en pruebas | 016 | Abierta |

---

## Base técnica

### A-01

**Adaptar la API existente o reiniciar su dominio.** · Bloquea: 001 · **Resuelta**

La [evaluación](investigacion/001-evaluacion-api-existente.md) concluye que la API no cumple ninguno de los seis criterios de viabilidad. No aísla empresas, no aplica permisos, guarda los secretos en texto plano y su suite de pruebas falla.

- **A.** Adaptar el código actual.
- **B.** Mismo repositorio, reiniciar el dominio (empresas, usuarios, roles) y reincorporar la emisión SUNAT con pruebas en 005.
- **C.** Proyecto Laravel nuevo copiando las piezas útiles.

**Recomendación:** B.

> **Decisión (2026-09-26):** Opción **B**: mismo repositorio, se reinicia el dominio y la emisión SUNAT se reincorpora con pruebas en la 005.

### A-02

**¿La API actual tiene empresas, comprobantes o usuarios reales que haya que conservar?** · Bloquea: 001 · **Resuelta**

Si hay datos reales, 001 necesita un plan de migración. Si solo hay datos de prueba, se puede empezar con migraciones limpias.

**Recomendación:** si no hay datos reales, empezar con un esquema nuevo.

> **Decisión (2026-09-26):** No hay datos reales, solo de prueba. Se empieza con un esquema limpio, sin plan de migración.

---

## 001 · Empresa, usuarios y aislamiento

### A-03

**¿Cómo se da de alta una empresa?** · Bloquea: 001 · **Resuelta**

- **A. Autoservicio:** cualquiera entra a una página pública, se registra y crea su empresa.
- **B. Por el administrador de la plataforma:** este crea la empresa y su administrador recibe una invitación para definir su contraseña.

A exige un endpoint público que crea datos. Eso requiere enmendar el principio IV y añadir verificación de correo, protección contra abuso y validación del RUC. Con 5 empresas iniciales, B es más simple y seguro.

**Recomendación:** B en el MVP; el autoservicio como spec propia cuando haya cobros.

> **Decisión (2026-09-26):** Opción **B**: la empresa la crea el administrador de la plataforma y su administrador recibe una invitación. No hay registro público en el MVP.

### A-04

**¿Un mismo usuario puede pertenecer a varias empresas?** · Bloquea: 001 · **Resuelta**

El borrador de 001 se contradecía: decía que "un usuario pertenece siempre a una empresa" y a la vez que "el mismo usuario puede atender varias empresas". Caso típico: un contador que lleva varias empresas.

**Recomendación:** en el MVP, cada usuario de empresa pertenece a **una sola** empresa, y el administrador de la plataforma a ninguna. La pertenencia se modela como relación aparte para poder abrirla después sin migrar datos.

> **Decisión (2026-09-26):** Aceptada la recomendación: cada usuario de empresa pertenece a una sola empresa y el administrador de la plataforma a ninguna. La pertenencia se modela aparte para poder abrirla después.

### A-05

**¿Qué roles hay en el MVP?** · Bloquea: 001 · **Resuelta**

La visión define tres personas; el borrador de 001 añadía un rol "consultor/lectura".

**Recomendación:** tres roles **fijos** (sin editor de roles ni permisos personalizables):

| Rol | Ámbito |
|---|---|
| Administrador de plataforma | Todas las empresas; no opera ventas |
| Administrador de empresa | Todo dentro de su empresa |
| Vendedor | Vender, emitir tickets y comprobantes, y consultar productos, stock y sus ventas |

El rol de solo lectura se añade cuando haya un caso real.

> **Decisión (2026-09-26):** Aceptada la recomendación: tres roles fijos (administrador de plataforma, administrador de empresa y vendedor), sin roles ni permisos personalizables.

### A-06

**Modelo de aislamiento y volumen esperado.** · Bloquea: 001 · **Resuelta**

¿Cuántas empresas se esperan a los 12 y a los 24 meses? ¿Cuántos usuarios y comprobantes al mes por empresa, aproximadamente?

**Recomendación:** una base de datos compartida con `company_id` en cada tabla de negocio, aplicado de forma automática (*global scope*) y verificado con pruebas de acceso cruzado en cada endpoint. Es suficiente con holgura para decenas de empresas y miles de comprobantes al mes. Las bases separadas por empresa multiplican el costo de migraciones y copias de seguridad sin beneficio a esta escala.

> **Decisión (2026-09-26):** Aceptada la recomendación: una base de datos compartida con `company_id`, aplicado automáticamente y verificado con pruebas de acceso cruzado. No se informó el volumen esperado; se asume del orden de decenas de empresas y se revisa si cambia.

### A-07

**¿Una empresa puede tener varios establecimientos en el MVP?** · Bloquea: 001, 004 · **Resuelta**

SUNAT asocia las series a establecimientos. La API actual ya modela sucursales (`branches`).

**Recomendación:** en el MVP, **un solo establecimiento** por empresa (el domicilio fiscal, anexo `0000`), creado automáticamente. El modelo de datos lo trata como entidad aparte para admitir varios más adelante.

> **Decisión (2026-09-26):** Aceptada la recomendación: un solo establecimiento por empresa (anexo `0000`), creado automáticamente y modelado como entidad aparte.

### A-08

**¿Dónde administra su empresa el administrador de empresa?** · Bloquea: 001 · **Resuelta**

El README del frontend dice que "la administración vive en Blade"; la visión dice que eso aplica a la plataforma, no necesariamente a cada empresa.

**Recomendación:** todo lo que hace un usuario de empresa, incluido su administrador (usuarios, series, productos, configuración SUNAT), va en el **frontend Vue**. Blade o Livewire queda solo para el panel de la plataforma (spec 006). Así la empresa usa una sola aplicación, también en el APK.

> **Decisión (2026-09-26):** Todo lo que hace un usuario de empresa, incluido su administrador, va en el **frontend Vue**. Blade o Livewire queda solo para el panel de la plataforma.
>
> **Actualización (2026-09-29, A-35):** el panel de la plataforma también se construye en el frontend Vue, en un área `/plataforma`.

### A-09

**¿Una empresa puede usar tickets internos y comprobantes SUNAT a la vez?** · Bloquea: 001, 003 · **Resuelta**

- **A.** La empresa elige un modo global: solo tickets o solo SUNAT.
- **B.** Son capacidades independientes: los tickets siempre están disponibles y SUNAT se activa cuando la configuración está validada.

**Recomendación:** B. El vendedor elige qué emitir en cada venta y la interfaz lo distingue con claridad. Depende de lo que se decida en [A-17](#a-17).

> **Decisión (2026-09-26):** Aceptada la recomendación: tickets y SUNAT son capacidades independientes. El alcance legal del ticket sigue pendiente en A-17.

### A-10

**Enmienda de la constitución a 1.1.0.** · Bloquea: 001 · **Resuelta**

La visión introduce obligaciones que la constitución 1.0.0 no recoge. Propuesta:

- **Nuevo principio VIII — Aislamiento por empresa y auditoría:** todo dato de negocio pertenece a una empresa; toda consulta se limita a la empresa del usuario autenticado; cada endpoint tiene una prueba de acceso cruzado; las acciones sensibles se auditan.
- ~~**Principio IV:** añadir el registro público a las excepciones~~ — ya no hace falta: en A-03 se eligió el alta por el administrador.
- **Principio VII:** aclarar que el frontend lo usan todos los usuarios de empresa, incluido su administrador (según [A-08](#a-08)).

**Recomendación:** aprobar la enmienda.

> **Decisión (2026-09-26):** Aprobada. La constitución pasa a 1.1.0 con el principio VIII y la aclaración del principio VII.

### A-11 ⚖️

**Tipos de contribuyente y régimen tributario.** · Bloquea: 001, 004 · **Resuelta**

Lo que se sabe, **a confirmar con un contador**:

- El RUC `10…` corresponde a una persona natural con negocio; el `20…`, a una persona jurídica. Ambas pueden emitir facturas y boletas según su régimen.
- En el **Nuevo RUS** solo se pueden emitir boletas (y tickets de máquina registradora), **no facturas**.
- RER, Régimen MYPE Tributario y Régimen General pueden emitir facturas y boletas.

**Recomendación:** en el alta, pedir RUC, razón social, nombre comercial (opcional), régimen tributario y domicilio fiscal con ubigeo. El tipo de persona se deduce del RUC y no se pregunta. Si el régimen es NRUS, no se permite emitir facturas.

> **Decisión (2026-09-26):** Aceptada la recomendación **como decisión de trabajo**: se piden RUC, razón social, nombre comercial opcional, régimen y domicilio fiscal; el tipo de persona se deduce del RUC; el Nuevo RUS no emite facturas. ⚖️ Hay que confirmarlo con un contador antes de producción.

---

## 002 · Inventario

### A-12

**¿Varios almacenes o un stock único por producto?** · Bloquea: 002 · **Resuelta**

**Recomendación:** stock único por empresa en el MVP (la visión ya excluye varios almacenes).

> **Decisión (2026-09-28):** Aceptada la recomendación: un stock único por empresa. El modelo no impide añadir almacenes después.

### A-13

**¿Todos los productos llevan lote y vencimiento?** · Bloquea: 002 · **Resuelta**

La visión dice que cada entrada registra un lote, pero muchos productos (ferretería, ropa) no tienen número de lote.

**Recomendación:**

- Toda entrada crea un lote. Si el usuario no indica número, el sistema genera uno (p. ej. `L-20260926-001`).
- El vencimiento es opcional y se activa **por producto** con la opción "controla vencimiento". Si está activa, la fecha es obligatoria.

> **Decisión (2026-09-28):** Aceptada la recomendación: toda entrada crea un lote (número propio o generado, p. ej. `L-20260928-001`). El vencimiento se activa por producto («controla vencimiento») y entonces es obligatorio.

### A-14

**¿Qué lote se descuenta al vender?** · Bloquea: 002 · **Resuelta**

**Recomendación:** automático. **FEFO** (vence primero, sale primero) si el producto controla vencimiento; **FIFO** (entra primero, sale primero) si no. Una venta puede consumir varios lotes. No se venden lotes vencidos. La elección manual de lote queda fuera del MVP.

> **Decisión (2026-09-28):** Aceptada la recomendación: asignación automática, FEFO si el producto controla vencimiento y FIFO si no. Una venta puede consumir varios lotes y nunca se venden lotes vencidos.

### A-15

**¿Se permite vender sin stock (stock negativo)?** · Bloquea: 002, 003 · **Resuelta**

**Recomendación:** no. La venta se bloquea si no hay stock disponible. Los servicios no controlan stock (ver A-16).

> **Decisión (2026-09-28):** Aceptada la recomendación: no se permite stock negativo; la venta se bloquea e indica el disponible. Los servicios no controlan stock.

### A-16

**Servicios, unidades de medida, costo e impuestos del producto.** · Bloquea: 002 · **Resuelta**

**Recomendación:**

- **Tipo de producto:** bien (controla stock) o servicio (no controla stock).
- **Unidad de medida:** una por producto, del catálogo SUNAT (`NIU` unidad, `KGM` kilo, `ZZ` servicio, etc.), sin conversiones entre unidades.
- **Costo:** opcional en la entrada de stock.
- **Precio de venta:** con IGV incluido, como lo ve el cliente.
- **Afectación al IGV:** obligatoria por producto (gravado, exonerado o inafecto), porque la emisión de comprobantes (005) la necesita.

> **Decisión (2026-09-28):** Aceptada la recomendación tal cual: bien o servicio; una unidad del catálogo SUNAT sin conversiones; costo opcional; precio de venta con IGV incluido; afectación al IGV obligatoria.

---

## 003 · Tickets

### A-17 ⚖️

**¿Para qué sirve legalmente el ticket interno?** · Bloquea: 003 · **Resuelta**

En el Perú toda venta debe sustentarse con un comprobante de pago y casi todos los contribuyentes están obligados a emitirlo electrónicamente. Un documento interno puede usarse como nota de venta, pedido o control, pero **no sustituye** al comprobante. El producto no debe facilitar ventas sin comprobante.

¿Cuál es el uso previsto?

- **A.** Control interno de negocios que emiten sus comprobantes con otro sistema.
- **B.** Nota de venta que después se convierte en boleta o factura desde este mismo sistema.
- **C.** Negocios todavía no formalizados.

**Recomendación:** A y B. La conversión a comprobante en una spec posterior. El ticket incluye siempre la leyenda "Documento interno — no es comprobante de pago". Confirmar con un contador antes de producción.

> **Decisión (2026-09-28):** Aceptada la recomendación: control interno para negocios que emiten sus comprobantes en otro sistema y nota de venta que se convertirá en boleta o factura en una spec posterior. Siempre con la leyenda «Documento interno — no es comprobante de pago». ⚖️ Confirmar con un contador antes de producción.

### A-18

**Contenido, impresión y anulación del ticket.** · Bloquea: 003 · **Resuelta**

**Recomendación:**

- **Cliente:** opcional; por defecto "Cliente varios".
- **Impresión:** vista imprimible de 80 mm y PDF desde el navegador. Bluetooth queda fuera del MVP.
- **Anulación:** solo el administrador de empresa. El ticket conserva su número con estado "anulado" y el stock se repone con un movimiento de reversión.
- **Numeración:** una secuencia por empresa con prefijo propio (p. ej. `T-000001`) que no pueda confundirse con una serie SUNAT.

> **Decisión (2026-09-28):** Aceptada la recomendación tal cual: cliente opcional («Cliente varios»), vista de 80 mm imprimible y PDF desde el navegador (sin Bluetooth), anulación solo por el administrador conservando el número y devolviendo el stock, y numeración propia por empresa `T-000001`.

### A-19

**¿Hace falta caja (apertura y cierre) o medios de pago?** · Bloquea: 003 · **Resuelta**

**Recomendación:** sin caja en el MVP. Solo se registra el medio de pago de cada venta (efectivo, tarjeta, Yape/Plin, transferencia) para poder sacar totales simples.

> **Decisión (2026-09-28):** Aceptada la recomendación: sin caja en el MVP; cada venta registra su medio de pago (efectivo, tarjeta, Yape/Plin, transferencia) para totales por día y por medio.

---

## 004 / 005 · SUNAT

### A-20

**¿El MVP incluye producción o solo beta?** · Bloquea: 004, 005 · **Resuelta**

**Recomendación:** solo beta. Pasar a producción es una spec propia, con controles adicionales y revisión legal ([A-24](#a-24)).

> **Decisión (2026-09-29):** Aceptada la recomendación: solo beta en el MVP. Producción será una spec propia con controles adicionales y revisión legal (A-24).

### A-21

**Series y certificados.** · Bloquea: 004 · **Resuelta**

**Recomendación:**

- **Series:** varias por tipo de comprobante, cada una con su correlativo independiente, asociadas al establecimiento. Al crear una serie se puede indicar el último número usado (para empresas que vienen de otro sistema). Después, el correlativo solo lo avanza la emisión y no puede editarse hacia atrás.
- **Certificado:** uno vigente por empresa. Se valida al subirlo (contraseña correcta, no vencido, RUC coincidente) y se avisa 30 días antes de su vencimiento. Reemplazarlo conserva el historial.

> **Decisión (2026-09-29):** Aceptada la recomendación: varias series por tipo, cada una con su correlativo, con último número usado al crearla y sin retroceso; con comprobantes solo se desactiva. Un certificado vigente por empresa, validado al subirlo (contraseña, vigencia, RUC), cifrado, con aviso 30 días antes de vencer; reemplazarlo conserva el historial.

### A-22 ⚖️

**Comprobantes y casos tributarios del MVP.** · Bloquea: 005 · **Resuelta**

**Recomendación:**

- **Comprobantes:** factura y boleta.
- **Operaciones:** gravadas (IGV 18 %), exoneradas e inafectas; solo en soles y al contado.
- **Fuera del MVP:** notas de crédito y débito, comunicación de baja, resumen diario, ISC, ICBPER, detracciones, percepciones, anticipos, moneda extranjera y ventas a crédito.

Consecuencia: en beta no se podrán anular comprobantes, lo cual es aceptable porque no tienen valor legal.

> **Decisión (2026-09-29):** Aceptada la recomendación: factura y boleta; gravadas (IGV 18 %), exoneradas e inafectas; solo soles y al contado (con los medios de pago de A-19: efectivo, tarjeta, Yape/Plin y transferencia). La boleta va a «Cliente varios» hasta S/ 700 y por encima exige el documento del comprador. Todo lo demás queda fuera del MVP y en beta no se anulan comprobantes. ⚖️ Confirmar con un contador antes de producción, incluido el tope de S/ 700.

### A-23

**Envío a SUNAT y qué pasa si SUNAT no responde.** · Bloquea: 005 · **Resuelta**

**Recomendación:**

- **Envío:** automático al emitir. El usuario no confirma aparte.
- **Si SUNAT no responde:** el comprobante queda "pendiente de envío" y se reintenta en segundo plano con espera creciente. Además hay un botón de reintento manual. El número asignado se conserva.
- **Si SUNAT rechaza:** estado "rechazado" con el motivo que devuelve SUNAT, sin reintentos.

> **Decisión (2026-09-29):** Aceptada la recomendación: envío automático al emitir; si SUNAT no responde, «pendiente de envío» con reintentos en segundo plano con espera creciente y reintento manual, conservando el número; si SUNAT rechaza, «rechazado» con el motivo y sin reintentos.

### A-32

**¿Qué comprobantes ve el vendedor?** · Bloquea: 005 · **Resuelta**

Surgió al releer la spec 005 (HU-5.2 lo dejaba para el plan). En la spec 003 el vendedor solo ve sus tickets.

**Recomendación:** solo los que emitió, igual que los tickets; el administrador ve todos.

> **Decisión (2026-09-29):** El vendedor ve **todos** los comprobantes de su empresa (útil cuando varios atienden la misma caja) y puede reintentar el envío de cualquiera. Los tickets siguen como en la 003: cada vendedor ve los suyos.

### A-33

**¿Dónde se emite el comprobante?** · Bloquea: 005 · **Resuelta**

Surgió al releer la spec 005: el objetivo dice «desde la misma venta», pero no qué pantalla.

**Recomendación:** en la pantalla «Vender» se elige Ticket, Boleta o Factura; el carrito, los precios y los descuentos son los mismos.

> **Decisión (2026-09-29):** Aceptada la recomendación: en «Vender» se elige Ticket, Boleta o Factura.

### A-34

**¿Quién registra clientes?** · Bloquea: 005 · **Resuelta**

RF-030 pide registrar clientes pero no decía quién.

**Recomendación:** el vendedor puede buscar y registrar al cliente desde «Vender» (DNI, RUC o carné de extranjería, validando el formato); editarlos es del administrador.

> **Decisión (2026-09-29):** Aceptada la recomendación: el vendedor busca y registra clientes al vender; editarlos queda para el administrador. No se borran clientes con comprobantes (RF-006 conserva los datos del momento de emitir).

### A-24 ⚖️

**Retención de documentos, protección de datos y seguridad legal.** · Bloquea: paso a producción · Abierta

Revisar con un especialista, antes de producción, la Ley 29733 de protección de datos personales, los plazos de conservación de comprobantes y las obligaciones del proveedor del SaaS. No bloquea el MVP en beta.

También previo a producción (A-38): segundo factor (TOTP) para los administradores de la plataforma.

## 006 · Panel de la plataforma

### A-35

**¿Con qué tecnología se construye el panel de la plataforma?** · Bloquea: 006 · **Resuelta**

A-08 dejó el panel de la plataforma en «Blade o Livewire», dentro del proyecto Laravel, cuando aún no existía nada. Hoy hay dos datos nuevos: la API de plataforma de la 001 (alta, edición, activar y desactivar empresas) ya existe y está probada, y el frontend Vue tiene su sistema de componentes, autenticación y pruebas. Livewire no está instalado.

- **A. Vue, en el mismo frontend:** un área `/plataforma` con su propio layout, que consume la API existente. Sin tecnología nueva; un solo sistema de diseño. El código del panel viaja en el mismo proyecto (se puede excluir del APK).
- **B. Blade + Livewire en Laravel:** separado de la app de las empresas, como decía A-08. Obliga a añadir Livewire, otra forma de probar y otro sistema de estilos, y duplica lógica que ya expone la API.

**Recomendación:** A. Reutiliza la API y las pruebas existentes y no suma tecnología (principio V). Requiere actualizar A-08.

> **Decisión (2026-09-29):** Aceptada la recomendación: el panel se construye en **Vue, en el mismo frontend**, como un área `/plataforma` con su propio layout que consume la API de plataforma. Actualiza A-08.

### A-36

**¿Qué entra en el panel del MVP?** · Bloquea: 006 · **Resuelta**

**Recomendación:**

- **Sí:** lista y búsqueda de empresas con su estado; ficha; alta; corrección de datos legales; desactivar (con motivo) y reactivar; reenviar la invitación al administrador de la empresa; vista de soporte de la emisión; auditoría de la plataforma.
- **No:** crear administradores de la plataforma desde el panel (sigue por consola, son pocos y muy sensibles), suplantar usuarios, planes y cobros, métricas de negocio.

> **Decisión (2026-09-29):** Aceptada la recomendación: lista y búsqueda con estado, ficha, alta, corrección de datos legales, suspender con motivo y reactivar, reenviar la invitación, soporte de la emisión y auditoría. Fuera: administradores de la plataforma (siguen por consola), suplantación, cobros y métricas de negocio.

### A-37

**¿Qué ve el administrador de la plataforma de cada empresa?** · Bloquea: 006 · **Resuelta**

Da soporte, pero los datos de negocio son de cada empresa (y personales en el caso de los clientes; ver A-24 ⚖️).

**Recomendación:** solo metadatos y contadores (usuarios, estado SUNAT, número de comprobantes pendientes o rechazados, fecha de la última actividad) y, para soporte, los datos de envío de los comprobantes pendientes o rechazados (número, estado, código y mensaje de SUNAT, intentos) con la opción de reintentar. Nunca secretos, productos, ventas, clientes ni líneas de comprobantes, y sin «entrar como» un usuario.

> **Decisión (2026-09-29):** Aceptada la recomendación: solo metadatos y contadores, más los datos de envío de los comprobantes pendientes o rechazados con opción de reintentar. Nunca secretos, productos, ventas, clientes ni líneas; sin «entrar como».

### A-38

**¿Cómo se protege el acceso del administrador de la plataforma?** · Bloquea: 006 · **Resuelta**

Es la cuenta con más poder del sistema. Hoy entra con correo y contraseña, igual que cualquier usuario.

- **A. Segundo factor (TOTP, app de autenticación) obligatorio ya en la 006.**
- **B. En la 006, sesión más corta (p. ej. 8 h) y auditoría de cada acceso; el segundo factor queda como requisito previo a producción**, junto con A-24.

**Recomendación:** B. En beta, con pocas empresas y sin valor legal, el riesgo es acotado y el segundo factor merece su propia spec; queda registrado como bloqueante de producción.

> **Decisión (2026-09-29):** Aceptada la recomendación: en la 006, sesión de 8 h para la plataforma y auditoría de cada acceso. El segundo factor (TOTP) queda como **requisito previo a producción**, junto con A-24.

## 007 · Anulación y corrección

### A-39 ⚖️

**¿Con qué instrumentos se corrige o anula un comprobante?** · Bloquea: 007 · **Resuelta**

SUNAT ofrece varios, con reglas distintas:

- **Nota de crédito (07):** modifica una factura o boleta aceptada. Motivos del catálogo 09, entre ellos: 01 anulación de la operación, 02 anulación por error en el RUC, 03 corrección por error en la descripción, 04/05 descuentos, 06 devolución total, 07 devolución por ítem, 09 disminución en el valor.
- **Nota de débito (08):** aumenta el valor (intereses, penalidades). Poco común en una tienda.
- **Comunicación de baja:** anula facturas que **no se entregaron** al cliente, dentro de un plazo corto.
- **Resumen diario:** la vía para anular boletas «por estado»; es un envío asíncrono con ticket de consulta.

**Recomendación:** solo **nota de crédito**, con los motivos **01** (anulación de la operación), **06** (devolución total) y **07** (devolución por ítem). Cubren anular, devolver todo y devolver parte, para facturas y boletas, con el mismo flujo de envío que la 005. Los errores de RUC o descripción se corrigen anulando (01) y emitiendo un comprobante nuevo. Quedan fuera las notas de débito, la baja, el resumen diario y los descuentos posteriores. ⚖️ Confirmar con el contador.

> **Decisión (2026-09-30):** Aceptada la recomendación: solo **nota de crédito**, con los motivos 01 (anulación de la operación), 06 (devolución total) y 07 (devolución por ítem), para facturas y boletas. Fuera: notas de débito, comunicación de baja, resumen diario y descuentos posteriores. ⚖️ Confirmar con el contador antes de producción.

### A-40

**¿Cuándo vuelve el stock?** · Bloquea: 007 · **Resuelta**

**Recomendación:** en una **devolución** (06, 07) el stock de lo devuelto se repone siempre. En una **anulación** (01) el administrador indica si la mercadería volvió, marcado por defecto que sí (por ejemplo, una venta anulada porque el cliente se arrepintió en caja). La reposición es una entrada nueva con el costo de los lotes que salieron, y referencia a la nota.

> **Decisión (2026-09-30):** Aceptada la recomendación: la devolución repone siempre lo devuelto; en la anulación se indica si la mercadería volvió (por defecto, sí). La reposición es una entrada con el costo de los lotes que salieron.

### A-41

**¿Quién emite notas de crédito?** · Bloquea: 007 · **Resuelta**

**Recomendación:** solo el **administrador de la empresa**, igual que anular tickets en la 003 (A-18): mueve dinero y stock. El vendedor ve las notas en el comprobante.

> **Decisión (2026-09-30):** **Cualquier usuario de la empresa**, también el vendedor, puede emitir notas de crédito (devoluciones y anulaciones), sobre cualquier comprobante de su empresa (A-32). Queda auditado quién la emitió. Descartar comprobantes rechazados sigue siendo solo del administrador (A-42).

### A-42

**¿Qué pasa con un comprobante rechazado por SUNAT?** · Bloquea: 007 · **Resuelta**

Surgió al revisar la 005: al emitir se descuenta el stock y se consume el número; si SUNAT rechaza (casi siempre por un dato mal ingresado), el comprobante no existe para SUNAT pero la venta y el stock quedaron registrados. No se le puede emitir una nota de crédito porque SUNAT no lo aceptó.

**Recomendación:** el administrador lo **descarta** con un motivo: queda «descartado», se repone su stock y su número queda como usado (no se reutiliza, para no romper la correlatividad). Luego se emite un comprobante nuevo con los datos corregidos.

> **Decisión (2026-09-30):** Aceptada la recomendación: el **administrador** descarta el rechazado con un motivo; queda «descartado», se repone su stock y su número queda como usado. Luego se emite uno nuevo.

## 009 · Despliegue y operación

### A-43

**¿Para qué es el primer despliegue?** · Bloquea: 009 · **Resuelta**

La revisión legal (A-24 ⚖️) de datos personales sigue pendiente, y todo es beta (sin valor legal). Los clientes que registran las empresas (DNI, nombres) son datos personales de terceros.

- **A. Piloto con empresas reales, sin datos de clientes reales:** las empresas prueban con sus productos, pero emiten a «Cliente varios» o con datos ficticios hasta cerrar A-24.
- **B. Piloto completo:** datos reales de clientes desde el primer día, con un aviso de privacidad provisional.
- **C. Solo demostración interna:** nadie de fuera la usa todavía.

**Recomendación:** A. Permite validar la operación (servidor, copias, avisos, uso en el celular) sin exponer datos personales de terceros antes de la revisión legal.

> **Decisión (2026-09-30):** Aceptada la recomendación: **piloto con empresas reales sin datos reales de sus clientes**. Emiten a «Cliente varios» o con datos ficticios hasta cerrar A-24 ⚖️; así se valida la operación sin exponer datos personales de terceros.

### A-44

**¿Dónde se aloja?** · Bloquea: 009 · **Resuelta**

**Recomendación:** **un solo servidor virtual (VPS)** en un proveedor con región cercana al Perú (p. ej. São Paulo o Miami): Ubuntu LTS, Nginx, PHP 8.5-FPM, PostgreSQL y la aplicación web como archivos estáticos en el mismo Nginx, con HTTPS de Let's Encrypt. Dos subdominios (aplicación y API). Para el volumen de A-06 un servidor basta y es lo más simple y barato; la base de datos local se protege con las copias de A-45. Una base de datos gestionada (más cara) queda para cuando haya más empresas.

> **Decisión (2026-09-30):** Aceptada la recomendación: **un VPS propio** en región cercana al Perú, con Ubuntu LTS, Nginx, PHP 8.5-FPM, PostgreSQL local y la aplicación web estática en el mismo Nginx, HTTPS de Let's Encrypt y dos subdominios.

### A-45

**¿Cómo son las copias de seguridad?** · Bloquea: 009 · **Resuelta**

**Recomendación:** copia **diaria** de PostgreSQL y de `storage/app/public` (logos), **cifrada** y enviada a un almacenamiento de objetos externo (compatible con S3), con retención de **30 copias diarias y 12 semanales**. `APP_KEY` se guarda **aparte** (gestor de contraseñas del responsable), porque sin ella los secretos de SUNAT de las copias no se descifran. Restauración de prueba **mensual**. La retención legal de comprobantes (A-24 ⚖️) se ajusta cuando se resuelva.

> **Decisión (2026-09-30):** Aceptada la recomendación: copia **diaria y cifrada** de PostgreSQL y de los logos en un almacenamiento externo compatible con S3, con 30 diarias y 12 semanales; `APP_KEY` aparte en un gestor de contraseñas; restauración de prueba mensual.

### A-46

**¿Cómo se despliega y dónde viven los repositorios?** · Bloquea: 009 · **Resuelta**

Hoy el `origin` de la API apunta al repositorio del autor original (`yorchavez9`) y el frontend no tiene remoto.

**Recomendación:** crear **repositorios privados propios** (API y frontend; la raíz de documentación también) en tu cuenta de GitHub. **GitHub Actions** corre las suites (API en SQLite y PostgreSQL, frontend completo) en cada cambio y, al integrar en `main`, despliega al servidor por SSH con un script sin tiempo de caída (carpetas de versión y enlace simbólico). Un ambiente de **pruebas (staging)** en el mismo servidor, con otra base de datos, para probar migraciones antes de producción.

> **Decisión (2026-09-30):** Aceptada la recomendación: **repositorios privados propios** en GitHub, **GitHub Actions** con todas las suites en cada cambio y despliegue por SSH sin tiempo de caída al integrar en `main`, y un ambiente de **staging** en el mismo servidor.
>
> **Actualización (2026-09-30, del usuario):** los repositorios se publican **públicos** en GitHub; después se harán unos ajustes que quedarán **solo en privado**. El despliegue al servidor se **pospone**: por ahora no se contrata el VPS. Consecuencia: antes de publicar se audita el historial de git en busca de secretos, y ningún secreto (ni siquiera de prueba real) puede entrar en los repositorios.

### A-47

**¿Con qué se envían los correos?** · Bloquea: 009 · **Resuelta**

**Recomendación:** un **proveedor de correo transaccional** (p. ej. Amazon SES, Resend o Brevo) con el **dominio propio** autenticado (SPF, DKIM y DMARC). El SMTP de un correo personal termina en spam y tiene límites bajos.

> **Decisión (2026-09-30):** Aceptada la recomendación: **proveedor de correo transaccional** con el dominio propio autenticado (SPF, DKIM, DMARC).

### A-48

**¿Qué se vigila y a quién se avisa?** · Bloquea: 009 · **Resuelta**

**Recomendación:**

- Un monitor externo de disponibilidad (servicio gratuito de *uptime*) sobre la web y un endpoint de salud de la API.
- Un «latido» del *scheduler*: si deja de correr, el monitor avisa.
- Una revisión cada 15 minutos que avisa si hay comprobantes pendientes hace más de 1 hora, si falló la última copia o si el disco pasa del 80 %.
- Errores 500 registrados con su detalle en el log (sin secretos) y aviso si se repiten.
- **Canal:** correo al responsable del SaaS.

> **Decisión (2026-09-30):** Aceptada la recomendación, con avisos **por correo** al responsable del SaaS.

---

## 010 · Identidad visual

### A-49

**¿Qué tema visual usa la aplicación?** · Bloquea: 010 · **Resuelta**

- **A.** «Petróleo»: verde azulado sobrio con neutros pizarra.
- **B.** Índigo, más «tecnológico».
- **C.** Azul marino con acento ámbar, más corporativo.

**Recomendación:** A. Transmite seriedad sin parecerse al azul por defecto de muchos SaaS. Cualquiera se cambia después en un solo lugar.

> **Decisión (2026-10-03):** A, «Petróleo».

### A-50

**¿Qué tipografía usa la aplicación?** · Bloquea: 010 · **Resuelta**

**Recomendación:** Inter, servida por la propia aplicación (sin pedirla a terceros), con dígitos de ancho fijo en montos.

> **Decisión (2026-10-03):** Aceptada la recomendación.

### A-51

**¿Qué logo usa la aplicación?** · Bloquea: 010 · **Resuelta**

El proyecto es público y reutilizable: no debe llevar una marca ajena.

> **Decisión (2026-10-03):** Un logo provisional y genérico (un comprobante con un visto), reemplazable en un solo lugar y pintado con los colores del tema. El definitivo se decide más adelante.

### A-55

**¿La aplicación tiene modo oscuro? ¿Qué fondo usa el modo claro?** · Bloquea: 010 · **Resuelta**

Surgió al revisar el Inicio (2026-10-03): el fondo casi blanco se ve muy claro.

- **A.** Claro y oscuro: por defecto sigue al sistema, con un selector Claro / Oscuro / Sistema que se recuerda en el dispositivo.
- **B.** Solo oscuro.
- **C.** Solo según el sistema, sin selector.

**Recomendación:** A. Quien vende de día al sol lee mejor en claro; de noche, en oscuro. Fondo claro gris pizarra suave (`#eef2f6`) para que las tarjetas blancas resalten.

> **Decisión (2026-10-03):** A, con el fondo gris pizarra suave.

## 011 · Panel de inicio

### A-52

**¿Qué indicadores muestra el inicio?** · Bloquea: 011 · **Resuelta**

**Recomendación:** pocos y accionables: ventas de hoy (monto y cantidad), evolución de los últimos 7 días, comprobantes pendientes de envío y rechazados, alertas de inventario, últimas 5 ventas y la acción «Nueva venta». Nada de reportes por rango (fuera del MVP).

> **Decisión (2026-10-03):** Aceptada la recomendación: ventas de hoy, últimos 7 días, por atender, últimas 5 ventas y «Nueva venta».

### A-53

**¿Qué ve el vendedor en el inicio?** · Bloquea: 011 · **Resuelta**

El vendedor ve solo sus tickets (A-30) pero todos los comprobantes (A-32).

- **A.** Solo sus propias ventas en los totales y en las últimas ventas; los comprobantes por atender de toda la empresa (puede reintentarlos); sin alertas de inventario (hoy solo el administrador puede consultarlas).
- **B.** Los totales de toda la empresa.

**Recomendación:** A. Coherente con A-30 y no expone la facturación total de la empresa a cada vendedor.

> **Decisión (2026-10-03):** A, solo lo suyo.

### A-54

**¿Qué suma como venta en los totales del inicio?** · Bloquea: 011 · **Resuelta**

**Recomendación:** tickets no anulados más comprobantes no rechazados ni descartados (pendientes, enviados y aceptados), menos las notas de crédito emitidas en el período; montos con IGV, en soles. Se presenta como un indicador operativo, no como un reporte contable.

> **Decisión (2026-10-03):** Aceptada la recomendación (neto operativo).

## 012 · Vender

### A-56

**¿Qué diseño usa «Vender»?** · Bloquea: 012 · **Resuelta**

- **A.** Dos columnas en pantallas anchas (productos y carrito fijo); en el celular, una columna con barra fija (cantidad · total · Cobrar) que abre el carrito.
- **B.** Dos columnas siempre.
- **C.** Una columna mejorada.

**Recomendación:** A. Dos columnas es el estándar de los puntos de venta, pero a 360 px no cabe.

> **Decisión (2026-10-03):** A.

### A-57

**¿Qué muestra la columna de productos sin búsqueda?** · Bloquea: 012 · **Resuelta**

**Recomendación:** el catálogo de productos activos, paginado, para agregar con un toque; la búsqueda lo filtra.

> **Decisión (2026-10-03):** Aceptada la recomendación.

### A-58

**¿Cómo persiste la venta en curso?** · Bloquea: 012 · **Resuelta**

- **A.** En el dispositivo, por usuario y empresa: sobrevive a cambiar de pantalla, recargar o cerrar la aplicación; se borra al cobrar, vaciar o cerrar sesión. Al volver se actualizan precios y disponible.
- **B.** Solo mientras la aplicación está abierta.

**Recomendación:** A. Incluye el cliente elegido: queda en el dispositivo de la empresa y se borra al cerrar sesión (tener en cuenta en A-24 ⚖️).

> **Decisión (2026-10-03):** A.

### A-59

**¿«Vaciar carrito» pide confirmación?** · Bloquea: 012 · **Resuelta**

> **Decisión (2026-10-03):** Sí, con confirmación que indica cuántos productos se quitarán.

### A-60

**¿«Vender» recuerda el último comprobante y serie elegidos?** · Bloquea: 012 v1.5 · **Resuelta**

Pedido del usuario: quien emite casi siempre boletas debe cambiar comprobante y serie en cada venta, porque al cobrar todo vuelve a ticket. Puntos a decidir:

1. **Qué se recuerda:** solo comprobante y serie, o también el medio de pago.
2. **Cuánto dura:** solo mientras dure la sesión, o también tras cerrar sesión.
3. **Aviso en el botón:** si el botón de cobro muestra comprobante y serie.

**Recomendación:** (1) solo comprobante y serie: el cliente es de cada venta y un medio de pago distinto de efectivo pasa inadvertido con facilidad; (2) también tras cerrar sesión, guardado por usuario y empresa (no es dato sensible); (3) sí, porque una boleta o factura emitida por error va a SUNAT y solo se corrige con una nota de crédito.

> **Decisión (2026-10-06):** Aceptada la recomendación en los tres puntos.

## 013 · Aplicación Android

### A-61

**¿Cómo se distribuye la app y con qué identidad?** · Bloquea: 013 · **Resuelta**

- **A.** APK firmado que se instala directamente (enlace de descarga o archivo enviado a cada empresa del piloto). Sin cuenta de Google Play; el usuario debe permitir «instalar apps de origen desconocido».
- **B.** Google Play, en prueba cerrada para el piloto y luego pública. Exige cuenta de desarrollador (pago único de 25 USD), política de privacidad publicada, ficha de la tienda y la revisión de Google; la política de privacidad depende de A-24 ⚖️.
- **C.** A para el piloto y B cuando se pase a producción.

También hay que fijar el **nombre** visible de la app y su **identificador** (por ejemplo `pe.tudominio.facturacion`), que en Google Play no se puede cambiar nunca; conviene que sea un dominio propio.

**Recomendación:** C. El APK directo permite empezar ya con 5 empresas sin trámites; Play se prepara junto con la revisión legal de A-24. Identificador basado en el dominio del SaaS, decidido antes de la primera entrega aunque se use solo el APK.

> **Decisión (2026-10-06):** C. APK firmado de instalación directa para el piloto; Google Play al pasar a producción, junto con A-24 ⚖️. El nombre visible y el identificador (basado en el dominio del SaaS) se fijan antes de la primera entrega a una empresa.

### A-62

**¿A qué servidor se conecta la app?** · Bloquea: 013 · **Resuelta**

Android bloquea por defecto las conexiones sin HTTPS, y el servidor de la 009 aún no existe (despliegue en espera).

- **A.** Esperar al servidor de la 009 (API con HTTPS en su dominio) para cualquier prueba en el celular.
- **B.** Desarrollar ya con una versión «de desarrollo» que se conecta a la API de la computadora por la red local (sin HTTPS, permitido solo en esa versión), y entregar a las empresas solo versiones contra la API con HTTPS de la 009.
- **C.** Un túnel temporal con HTTPS hacia la computadora de desarrollo para probar en celulares fuera de la red local.

**Recomendación:** B. Permite construir y probar toda la app ahora; la entrega a las empresas (CE-001 con usuarios reales) espera al servidor de la 009. C solo si hace falta probar fuera de casa.

> **Decisión (2026-10-06):** B. Versión de desarrollo contra la API de la computadora por la red local; a las empresas solo se entregan versiones contra la API con HTTPS de la 009.

### A-63

**¿Cómo se imprime y descarga en la app?** · Bloquea: 013 · **Resuelta**

Dentro de una app, la impresión del navegador y las descargas de archivos no funcionan como en la web.

- **A.** Al imprimir o descargar, la app genera el PDF (o XML, CDR) y abre el menú «Compartir» / «Abrir con» de Android: se elige el servicio de impresión del sistema, la app de la impresora térmica (muchas aceptan PDF), WhatsApp o guardar en archivos.
- **B.** Impresión Bluetooth directa a la térmica (ESC/POS), sin pasar por otras apps.

**Recomendación:** A en esta spec: funciona con casi cualquier impresora y app y no requiere permisos de Bluetooth. B queda para una spec propia (está fuera del MVP en la visión) cuando se sepa qué impresoras usan las empresas.

> **Decisión (2026-10-06):** A. Menú «Compartir» / «Abrir con» de Android con el PDF o archivo. La impresión Bluetooth directa queda para una spec propia.

### A-64

**¿Dónde guarda la app la sesión?** · Bloquea: 013 · **Resuelta**

En la web el token va en el almacenamiento del navegador. La constitución (principio VII) prevé el almacenamiento seguro del dispositivo para el APK.

- **A.** Almacenamiento protegido de Android (cifrado con el almacén de claves del sistema), excluido de las copias de seguridad.
- **B.** El mismo almacenamiento web que hoy, dentro de la app.

**Recomendación:** A. Es lo que prevé la constitución y el módulo de la sesión ya está aislado para cambiarlo sin tocar el resto. La venta en curso y las preferencias (no sensibles) siguen en el almacenamiento de la app.

> **Decisión (2026-10-06):** A. Almacenamiento protegido de Android, excluido de las copias de seguridad.

### A-65

**¿El panel de la plataforma va en la app?** · Bloquea: 013 · **Resuelta**

**Recomendación:** no. La app es para usuarios de empresa; si entra el administrador de la plataforma, se le indica que use la web (A-35 ya preveía excluirlo del APK). Menos código en la app y menos superficie de riesgo en un teléfono.

> **Decisión (2026-10-06):** No. El administrador de la plataforma usa la web.

### A-66

**¿La app permite vender sin conexión?** · Bloquea: 013 · **Resuelta**

**Recomendación:** no en esta spec. Los correlativos, el stock y el envío a SUNAT se deciden en el servidor (principio II); vender sin conexión exige reglas de numeración y conflictos propias de otra spec. Sin conexión se avisa claramente y la venta en curso se conserva (spec 012).

> **Decisión (2026-10-06):** No en esta spec; aviso claro y la venta en curso se conserva.

### A-67

**¿Cómo se mantiene la app compatible con la API?** · Bloquea: 013 · **Resuelta**

La app lleva la interfaz dentro: un cambio en la API puede romper versiones viejas instaladas.

- **A.** Cada versión de la app informa su número; la API publica la versión mínima aceptada y la app pide actualizar si está por debajo. Cada cambio de interfaz es un APK nuevo.
- **B.** La app carga la interfaz desde el servidor web en cada apertura (siempre al día), y el APK es solo un contenedor.
- **C.** Sin control: se avisa a las empresas a mano cuando haya versión nueva.

**Recomendación:** A. Es el modelo habitual y el que acepta Google Play (B depende siempre de la red para abrir y Play puede rechazarla como «sitio web empaquetado»). Requiere un cambio pequeño en la API (la versión mínima), que irá en el plan.

> **Decisión (2026-10-06):** A. Versión mínima publicada por la API; la app pide actualizar si está por debajo.

## 014 · Exportar e importar datos

Evaluación previa: [`investigacion/002-exportar-importar-datos.md`](investigacion/002-exportar-importar-datos.md). Retoma A-26 (importación masiva con spec propia).

### A-68

**¿Qué datos se exportan y cuáles se importan?** · Bloquea: 014 · **Resuelta**

- **A.** Exportar productos (con stock), clientes y ventas (tickets y comprobantes). Importar solo productos y clientes.
- **B.** Lo mismo que A, y además importar ventas de otro sistema como historial: ventas «históricas» que no descuentan stock ni se envían a SUNAT. Es un tipo de dato nuevo y aparecería en reportes y en el panel de inicio.
- **C.** Solo productos, exportar e importar (lo mínimo de A-26). Clientes y ventas, más adelante.

**Recomendación:** A. Para empezar a vender, quien migra necesita el catálogo, el stock y los clientes. Importar ventas choca con el principio II: los comprobantes deben firmarse y enviarse desde aquí, con correlativos de las series de la 004. Además, los tickets mueven el stock.

> **Decisión (2026-10-07):** A. Exportar productos con stock, clientes y ventas (tickets y comprobantes); importar solo productos y clientes.

### A-69

**¿En qué formato?** · Bloquea: 014 · **Resuelta**

- **A.** Solo CSV. Es lo más simple, pero Excel lo abre mal si no se cuida la codificación y el separador.
- **B.** Solo Excel (XLSX). Es lo que usan las tiendas y no tiene problemas de tildes ni de separadores.
- **C.** Exportar en XLSX y CSV; importar desde XLSX o CSV con la plantilla descargable.

**Recomendación:** C. Se exporta en XLSX por defecto y en CSV para quien lo lleve a otro sistema. La plantilla de importación trae columnas, ejemplos y valores admitidos.

> **Decisión (2026-10-07):** C. Exportar en XLSX (por defecto) y CSV; importar desde XLSX o CSV con la plantilla descargable.

### A-70

**Al importar, ¿qué pasa con un producto o cliente que ya existe?** · Bloquea: 014 · **Resuelta**

El producto se reconoce por su código y el cliente por su tipo y número de documento (los dos son únicos por empresa).

- **A.** Solo se crean registros nuevos; los que ya existen se marcan como error.
- **B.** Siempre se actualizan con los datos del archivo.
- **C.** Al importar, el usuario elige entre «solo crear» y «crear y actualizar». La vista previa muestra cuáles se crean y cuáles se actualizan, campo por campo.

**Recomendación:** C, con «solo crear» por defecto. Sirve tanto para la carga inicial como para actualizar precios en bloque, y nada se pisa sin verlo antes. Cada cambio queda auditado como si fuera manual. El stock nunca se modifica al actualizar un producto: se cambia con entradas y ajustes, como hoy.

> **Decisión (2026-10-07):** C. El usuario elige «solo crear» (por defecto) o «crear y actualizar», con vista previa de los cambios; cada cambio se audita y el stock nunca se modifica al actualizar.

### A-71

**¿Qué pasa si algunas filas tienen errores?** · Bloquea: 014 · **Resuelta**

- **A.** Todo o nada: si una fila tiene un error, no se importa ninguna. La vista previa indica cada error por fila y columna para corregir el archivo.
- **B.** Se importan las filas válidas y se informan las que fallaron.

**Recomendación:** A. Así no quedan importaciones a medias: con B, al corregir y volver a subir el archivo se duplicarían o pisarían filas. La vista previa evita sorpresas porque nada se aplica hasta confirmar.

> **Decisión (2026-10-07):** A. Todo o nada, con vista previa de los errores por fila y columna; nada se aplica hasta confirmar.

### A-72

**¿La importación de productos puede cargar el stock inicial?** · Bloquea: 014 · **Resuelta**

- **A.** No. Los productos entran sin stock y se registran entradas por cada uno, como hoy.
- **B.** Sí, con columnas opcionales (cantidad, costo, lote y vencimiento) que generan una entrada de mercadería normal (A-28), solo para productos **nuevos**.

**Recomendación:** B. Con cientos de productos, cargar el stock a mano anula la ventaja de importar. Al ser una entrada normal, el kárdex, los lotes FEFO/FIFO y la auditoría siguen funcionando igual. Para un producto que ya existe, la columna de stock se ignora y se avisa de ello.

> **Decisión (2026-10-07):** B. Columnas opcionales de stock inicial (cantidad, costo, lote y vencimiento) que generan una entrada normal, solo para productos nuevos; en productos existentes se ignoran con aviso.

### A-73

**¿Quién puede exportar e importar, y con qué límites?** · Bloquea: 014 · **Resuelta**

- **A.** Solo el administrador de empresa, para todo.
- **B.** El administrador, para todo; el vendedor solo exporta sus propias ventas (lo que ya ve, A-30).

Límites propuestos:
- importaciones de hasta 2 000 filas y 5 MB por archivo;
- exportaciones de ventas por rango de fechas de hasta 12 meses;
- cada exportación e importación queda en la auditoría.

**Recomendación:** A. La exportación saca el costo (A-27) y datos personales de clientes (A-24 ⚖️). Mientras no esté la revisión legal, conviene que solo lo haga el responsable de la empresa. Los límites pueden subirse si alguna empresa lo necesita.

> **Decisión (2026-10-07):** A. Solo el administrador de empresa. Límites: 2 000 filas y 5 MB por importación, exportación de ventas de hasta 12 meses; todo queda en la auditoría.

### A-74

**¿Qué trae la exportación de ventas?** · Bloquea: 014 · **Resuelta**

- **A.** Una fila por ticket o comprobante (fecha, tipo, número, cliente, vendedor, medio de pago, bases, IGV, total y estado).
- **B.** Una fila por línea vendida (producto, cantidad, precio, descuento e importe), con los datos del documento repetidos.
- **C.** Las dos, en dos hojas del mismo XLSX (en CSV, dos archivos), con filtros de fechas, tipo (ticket, boleta, factura o nota) y estado.

En todos los casos se incluyen los anulados, los rechazados y las notas de crédito, con su estado, y los importes de las notas van en negativo.

**Recomendación:** C. La hoja de documentos sirve para cuadrar caja y la de líneas, para ver qué se vende. Los formatos oficiales de SUNAT (PLE / registro de ventas) quedan fuera ⚖️: requieren al contador y van en otra spec.

> **Decisión (2026-10-07):** C. Dos hojas (documentos y líneas; en CSV, dos archivos) con filtros de fechas, tipo y estado; incluye anulados, rechazados y notas de crédito con importes en negativo. El PLE de SUNAT queda fuera ⚖️.

## 015 · Impresión Bluetooth

Evaluación previa: [`investigacion/003-impresion-bluetooth.md`](investigacion/003-impresion-bluetooth.md). Retoma A-63, opción B (impresión Bluetooth directa con spec propia).

### A-75

**¿Qué impresoras y qué tipo de Bluetooth se admiten?** · Bloquea: 015 · **Resuelta**

- **A.** Impresoras térmicas ESC/POS con **Bluetooth clásico** («puerto serie», SPP). Es el que usa la gran mayoría de los modelos baratos de 58 y 80 mm.
- **B.** Solo **Bluetooth de baja energía** (BLE). Lo traen algunos modelos nuevos, y cada fabricante usa sus propios canales.
- **C.** Las dos.

También hace falta saber **qué modelos usan o usarían las empresas del piloto**, para comprar al menos uno de referencia y verificar con él.

**Recomendación:** A, verificada con una impresora de referencia de 58 mm (y una de 80 mm si es posible). BLE puede sumarse después, si aparece una impresora que solo lo tenga.

> **Decisión (2026-10-23):** A. Impresoras térmicas ESC/POS con Bluetooth clásico (SPP), verificadas con una impresora de referencia de 58 mm (y una de 80 mm si es posible). BLE puede sumarse después. **Pendiente:** el modelo de referencia, que se elige con lo que usen las empresas del piloto antes de la verificación.

### A-76

**¿Qué anchos de papel se admiten?** · Bloquea: 015 · **Resuelta**

- **A.** Solo 80 mm, como el PDF actual.
- **B.** 58 mm y 80 mm, elegidos al configurar la impresora.

**Recomendación:** B. Las de 58 mm son las más comunes en bodegas, y la constitución ya prevé un formato angosto (principio II).

> **Decisión (2026-10-23):** B. 58 mm y 80 mm, elegidos al configurar la impresora.

### A-77

**¿Qué se imprime por Bluetooth y con qué contenido?** · Bloquea: 015 · **Resuelta**

1. **Documentos:**
   - solo el ticket interno;
   - o también boletas, facturas y notas de crédito, con su QR y su resumen, como en el PDF de 80 mm.
2. **Logo de la empresa:** impreso como imagen, o solo el nombre en letras grandes.
3. **Copias:** una sola, o un número configurable (por ejemplo, una para el cliente y otra para el negocio).

**Recomendación:**
1. Todos: ticket, boleta, factura y nota de crédito, con el mismo contenido que el PDF de 80 mm. El QR va con el comando de la impresora, y como imagen si no lo admite.
2. Sin logo en esta spec: alarga la impresión y su calidad depende del modelo. El encabezado lleva nombre comercial, RUC y dirección.
3. Una copia, y se pulsa «Imprimir» otra vez si hace falta.

> **Decisión (2026-10-23):** Aceptada la recomendación: ticket, boleta, factura y nota de crédito con el mismo contenido que el PDF de 80 mm (QR con el comando de la impresora, o como imagen si no lo admite); sin logo en esta spec; una copia.

### A-78

**¿Cómo se elige la impresora y cuándo imprime?** · Bloquea: 015 · **Resuelta**

1. **Elegir la impresora:**
   - la impresora se empareja desde los ajustes de Android y en la app se elige entre las emparejadas;
   - o la app busca impresoras cercanas y las empareja ella misma, lo que pide más permisos, como la búsqueda de equipos y la ubicación en Android antiguos.
2. **Dónde se guarda:** solo en ese teléfono, o para toda la empresa.
3. **Cuándo imprime:**
   - solo al pulsar «Imprimir»;
   - o también automáticamente al cobrar, como opción que se activa en ese teléfono.
4. **Si falla:** reintentar, o «Compartir PDF» como hoy (A-63).

**Recomendación:**
1. Emparejar desde Android y elegir en la app: menos permisos y es lo habitual en las apps de punto de venta. Se añade una «Hoja de prueba» para comprobar ancho y tildes.
2. Solo en ese teléfono: cada caja tiene su impresora, y no es un dato de la empresa.
3. «Imprimir» va directo a la impresora elegida. La impresión automática al cobrar existe como opción, apagada por defecto.
4. Si falla, el aviso dice el motivo (apagada, fuera de alcance, sin permiso) y ofrece «Reintentar» y «Compartir PDF». Sin impresora configurada, «Imprimir» sigue abriendo «Compartir» como en la 013.

> **Decisión (2026-10-23):** Aceptada la recomendación: se empareja en Android y se elige en la app (con «Hoja de prueba»); la elección vive solo en ese teléfono; «Imprimir» va directo a la impresora y «Imprimir al cobrar» es opcional y viene apagado; si falla, aviso con el motivo, «Reintentar» y «Compartir PDF»; sin impresora elegida, «Compartir» como en la 013.

## 016 · Imagen del producto

Pedido del usuario: foto opcional en cada producto, guardada en un almacenamiento compatible con S3. Lo que ya hay en la API:

- **Dependencia:** `league/flysystem-aws-s3-v3` 3.35 y `aws/aws-sdk-php` 3.398 ya están instaladas (entraron con la spec 009, para las copias), sin avisos de seguridad.
- **Disco `s3`:** existe en `config/filesystems.php`, pero hoy es el destino de las **copias de seguridad cifradas** (`BACKUP_DISK=s3`).
- **`.env.example`:** no tiene variables `AWS_*`.
- **Logo de la empresa:** se guarda en el disco local `public` (y entra en las copias).

### A-79

**¿Dónde se guardan las fotos y quién puede verlas?** · Bloquea: 016 · **Abierta**

1. **Lugar:**
   - **A.** El mismo bucket que las copias de seguridad (spec 009).
   - **B.** Un bucket **aparte**, en el mismo proveedor que las copias (Backblaze B2, Cloudflare R2 o AWS S3, A-45).
2. **Acceso:**
   - **A.** Bucket **privado**: la API entrega direcciones firmadas que vencen (por ejemplo, en una hora). Nadie ve una foto sin pasar por la API.
   - **B.** Bucket **público** con direcciones imposibles de adivinar. Es más simple y el navegador las guarda en caché, pero quien tenga una dirección la puede compartir.
3. **Logo de la empresa:** ¿pasa también a S3 en esta spec («todas las imágenes»), o se queda como está?

**Recomendación:**
1. B, bucket aparte. Mezclar fotos con copias cifradas complica los permisos y la retención (las copias se borran a los 30 días; las fotos, nunca).
2. A, privado con direcciones firmadas: sigue el principio VIII sin excepciones. Las direcciones se firman con vencimientos redondeados para que el celular pueda reusar las fotos ya descargadas.
3. Solo los productos en esta spec. El logo funciona y entra en las copias; moverlo puede ir en otra spec.

### A-80

**¿Qué formatos y tamaño se aceptan?** · Bloquea: 016 · **Abierta**

- **Formatos:** JPEG, PNG y WebP; ¿también HEIC (fotos de iPhone)?
- **Tamaño máximo del archivo subido:** por ejemplo 5 MB o 10 MB.

**Recomendación:** JPEG, PNG y WebP, hasta **5 MB**. HEIC queda fuera: Android entrega JPEG, y HEIC necesitaría una librería más en el servidor.

### A-81

**¿Se guarda la foto tal cual o se procesa?** · Bloquea: 016 · **Abierta**

- **A.** Tal cual, como se subió.
- **B.** Procesada en el servidor:
  - una versión **grande** (por ejemplo, 800 px de lado) y una **miniatura** (por ejemplo, 200 px) para las listas;
  - girada según su orientación y **sin datos ocultos** (ubicación GPS, modelo del teléfono);
  - en un formato liviano (WebP).

**Recomendación:** B. Una foto de celular pesa 3 a 5 MB: sin miniaturas, «Vender» descargaría decenas de MB por página (CE-002). Quitar la ubicación protege a quien toma la foto en su casa o su tienda. Se puede hacer con la extensión GD, que la API ya exige, sin dependencias nuevas.

### A-82

**¿Qué pasa con la foto al cambiarla, quitarla o desactivar el producto?** · Bloquea: 016 · **Abierta**

- **Cambiar o quitar:** ¿se borra la foto anterior del almacenamiento enseguida, o se conserva un tiempo?
- **Desactivar el producto:** ¿se conserva la foto?
- **Copias:** ¿las fotos entran en las copias de seguridad de la 009, o basta con el almacenamiento (que ya guarda varias réplicas)?

**Recomendación:**
- Al cambiar o quitar, se **borra enseguida**, con una limpieza diaria que quita las fotos huérfanas si un borrado falla.
- Al desactivar, la foto **se conserva** (el producto no se borra, spec 002).
- Las fotos **no entran** en las copias cifradas. Se activa el **versionado** del bucket con 30 días de retención, para recuperar una foto borrada por error.

### A-83

**¿Las fotos entran en exportar e importar (spec 014)?** · Bloquea: 016 · **Abierta**

- **A.** No en esta spec.
- **B.** La exportación agrega una columna con la dirección de la foto.
- **C.** También importar fotos (por ejemplo, un ZIP con las imágenes).

**Recomendación:** A. Una dirección firmada vence (A-79) y no sirve en un Excel. Importar fotos en lote es un trabajo aparte, que puede ir en otra spec si hace falta.

### A-84

**¿Qué almacenamiento se usa en desarrollo y en pruebas?** · Bloquea: 016 · **Abierta**

- **A.** Un S3 de verdad también en desarrollo (un bucket de pruebas).
- **B.** MinIO (un S3 local en Docker) en desarrollo.
- **C.** En desarrollo, el disco local de siempre, elegido con una variable; en producción, S3.

En las pruebas automáticas se usa un almacenamiento simulado (`Storage::fake`), sin red.

**Recomendación:** C para el día a día, sin servicios extra. Antes de cerrar la spec se verifica una vez contra un S3 real, o contra MinIO, para comprobar las direcciones firmadas y los permisos. Las pruebas automáticas usan el almacenamiento simulado.

## Pendientes menores

### A-25

**¿El MVP incluye recuperación de contraseña por correo?** · Bloquea: 001 · **Resuelta**

Requiere configurar el envío de correo, que también usan las invitaciones de A-03.

**Recomendación:** sí. Si ya se envían invitaciones por correo, el costo adicional es bajo y evita depender del soporte.

> **Decisión (2026-09-26):** Aceptada la recomendación: la recuperación de contraseña por correo entra en el MVP.

### A-26

**¿El MVP incluye importar productos desde Excel/CSV?** · Bloquea: 002 · **Resuelta**

Las empresas que vienen de otro sistema suelen tener cientos de productos.

**Recomendación:** no en el MVP; spec propia después de la 002. Con 5 empresas iniciales, la carga manual o asistida es viable.

> **Decisión (2026-09-28):** Aceptada la recomendación: la importación masiva queda fuera del MVP y tendrá spec propia.

### A-27

**¿El vendedor ve el costo de productos y lotes?** · Bloquea: 002 · **Resuelta**

Surgió al releer la spec 002 con las decisiones anteriores: el costo revela el margen del negocio.

> **Decisión (2026-09-28):** No. El costo solo lo ve el administrador de empresa; el vendedor ve precio de venta y stock disponible.

### A-28

**¿Quién registra las entradas de mercadería?** · Bloquea: 002 · **Resuelta**

En tiendas pequeñas a menudo recibe la mercadería el vendedor.

> **Decisión (2026-09-28):** Solo el administrador de empresa, como ya decía RF-020. Ajustes y reversiones también.

### A-29

**¿El vendedor puede cambiar el precio al vender?** · Bloquea: 003 · **Resuelta**

Surgió al releer la spec 003: solo hablaba de descuento por línea.

> **Decisión (2026-09-28):** No. El precio es el del catálogo; se admite un descuento por línea (nunca mayor que el importe), registrado en el ticket y auditado.

### A-30

**¿Quién registra ventas con ticket?** · Bloquea: 003 · **Resuelta**

> **Decisión (2026-09-28):** El administrador de empresa y el vendedor. El vendedor ve solo sus tickets; el administrador, todos.

### A-31

**¿Qué credenciales se usan en el ambiente beta?** · Bloquea: 004, 005 · **Resuelta**

Surgió al releer la spec 004: en beta SUNAT acepta credenciales genéricas de prueba (usuario `MODDATOS`) y cualquier certificado, así que una conexión exitosa en beta no demuestra que la clave SOL real de la empresa sea correcta.

> **Decisión (2026-09-29):** En beta se envía con las credenciales genéricas de prueba de SUNAT. La empresa registra igual su clave SOL y su certificado reales (cifrados y validados localmente) para cuando pase a producción, y la interfaz indica claramente que la clave SOL real aún no se verificó contra SUNAT.
