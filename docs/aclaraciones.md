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
