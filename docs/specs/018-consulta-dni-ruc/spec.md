# Spec 018: Consulta de DNI y RUC

**Estado:** Borrador (aclaraciones resueltas, falta aprobación) · **Versión:** 0.2 · **Fecha:** 2026-10-30
**Depende de:** 001, 005, 012, 014, 017 · **Aclaraciones:** A-92 a A-96 (resueltas)

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que al vender, quien atiende escriba un DNI o un RUC y el cliente aparezca con su nombre (y, si es RUC, con su razón social y dirección) sin tipearlos, usando el servicio gratuito de apisperu.com. Como el plan gratuito da unas 1000 consultas al día para toda la plataforma, primero se busca en lo ya guardado y solo si no está se consulta. En el MVP no hay topes por empresa: si el uso crece, se pasará al plan ilimitado (A-96). El registro manual sigue disponible, pero un cliente traído del servicio queda **verificado** y prevalece sobre lo escrito a mano (A-92).

## Historias de usuario

### HU-1 · Buscar al cliente por DNI o RUC al vender (P1)

Como vendedor, quiero escribir el DNI o el RUC del cliente y que sus datos aparezcan solos, para emitir la boleta o la factura más rápido y sin errores de tipeo.

**Por qué esta prioridad:** es el uso diario y el motivo de la spec.
**Prueba independiente:** en «Vender», con un RUC que nunca se registró, escribo el número y aparece la razón social y la dirección; emito la factura con esos datos.

**Escenarios de aceptación**

1. **Dado** un cliente ya guardado en mi empresa, **cuando** escribo su DNI o RUC, **entonces** aparece al instante, sin consultar el servicio.
2. **Dado** un documento que mi empresa no tiene pero que la plataforma ya consultó antes (A-93), **cuando** lo escribo, **entonces** aparece sin gastar una consulta y queda guardado como cliente verificado de mi empresa.
3. **Dado** un documento que nadie consultó, **cuando** lo escribo, **entonces** se consulta el servicio, aparece con su nombre (o razón social y dirección) y queda guardado como cliente verificado.
4. **Dado** un RUC, **cuando** aparece, **entonces** veo su estado y condición; si está de baja o no habido veo un aviso claro, pero puedo emitir (A-95 ⚖️).
5. **Dado** que el servicio no encuentra el documento, no responde o dice que se acabó el cupo del día (A-96), **cuando** busco, **entonces** veo por qué y puedo registrar al cliente a mano, sin perder la venta en curso.
6. **Dado** un número con formato inválido (un DNI que no tiene 8 dígitos o un RUC que no tiene 11 o cuyo dígito verificador no cuadra), **cuando** lo escribo, **entonces** veo el error enseguida y no se gasta una consulta.
7. **Dado** un carné de extranjería u otro documento, **cuando** registro al cliente, **entonces** lo hago a mano, como hoy (A-96).

### HU-2 · Clientes verificados que no se pisan a mano (P1)

Como administrador de empresa, quiero que los datos traídos del servicio no se cambien a mano, para confiar en que el nombre y la dirección de mis facturas son los oficiales.

**Por qué esta prioridad:** es lo que hace que «prevalezca» lo consultado (A-92).
**Prueba independiente:** un cliente verificado no deja editar su nombre ni su dirección; uno manual sí, y al consultarlo pasa a verificado con los datos del servicio.

**Escenarios de aceptación**

1. **Dado** un cliente verificado, **cuando** lo edito, **entonces** no puedo cambiar su nombre ni (si es RUC) su dirección; sí sus datos propios de contacto, si los hay.
2. **Dado** un cliente registrado a mano, **cuando** se consulta su documento y el servicio lo encuentra, **entonces** su nombre y dirección se reemplazan por los del servicio y pasa a verificado; el cambio queda en la auditoría.
3. **Dado** un cliente manual cuyo documento el servicio no encuentra, **cuando** lo edito, **entonces** puedo seguir cambiándolo a mano.
4. **Dado** un cliente, **cuando** lo veo en «Clientes» o al vender, **entonces** distingo si es «Verificado» (con la fecha de la consulta) o «Manual».
5. **Dado** que importo clientes desde un archivo (spec 014), **cuando** un documento ya está verificado, **entonces** la importación no cambia su nombre ni su dirección y lo avisa en la vista previa.

### HU-3 · Actualizar los datos de un cliente (P2)

Como vendedor o administrador, quiero volver a consultar a un cliente cuando sé que cambió (por ejemplo, la dirección de una empresa), para que mis comprobantes salgan con los datos al día.

**Por qué esta prioridad:** los datos de un RUC cambian, pero rara vez; se hace a pedido para cuidar el cupo (A-94).
**Prueba independiente:** pulso «Actualizar datos» en un cliente verificado con dirección vieja y queda con la dirección nueva y la fecha de hoy.

**Escenarios de aceptación**

1. **Dado** un cliente con DNI o RUC, **cuando** pulso «Actualizar datos», **entonces** se consulta el servicio (gasta una consulta) y se guardan los datos nuevos, también en la tabla común.
2. **Dado** que el servicio falla o no hay cupo, **cuando** actualizo, **entonces** el cliente queda como estaba y veo el motivo.
3. **Dado** un comprobante ya emitido, **cuando** actualizo al cliente, **entonces** el comprobante no cambia (spec 005: guarda los datos del momento de emitir).

### HU-4 · Ver cuánto se usa el servicio (P3)

Como administrador de la plataforma, quiero ver cuántas consultas se hacen al día, para saber cuándo conviene pagar el plan ilimitado (A-96).

**Por qué esta prioridad:** no cambia el uso diario; solo informa una decisión de costo.
**Prueba independiente:** después de tres consultas nuevas, el panel de la plataforma muestra 3 consultas hoy.

**Escenarios de aceptación**

1. **Dado** el panel de la plataforma (spec 006), **cuando** lo abro, **entonces** veo las consultas al servicio de hoy y de los últimos días, en total.
2. **Dado** que se repite la misma búsqueda (doble toque, reintento), **cuando** el documento se acaba de consultar, **entonces** no se gasta una segunda consulta.
3. **Dado** que el servicio responde que se acabó el cupo, **cuando** cualquier empresa busca, **entonces** lo ya guardado sigue apareciendo y lo demás se registra a mano.

## Casos límite

- **Servicio sin token configurado:** la búsqueda solo mira lo guardado y el registro sigue a mano; nada falla.
- **Respuesta incompleta o con caracteres raros** (tildes, «Ñ», comillas): se guarda tal cual, sin romper el XML ni el PDF.
- **Nombres de DNI:** se arman como «NOMBRES APELLIDO_PATERNO APELLIDO_MATERNO» o en el orden que se decida en el plan, siempre igual.
- **RUC de persona natural (10…):** trae nombre y, a veces, sin dirección; la factura sigue el criterio actual (spec 005).
- **Cliente verificado con datos que cambiaron en el servicio:** solo cambia si alguien pulsa «Actualizar datos» (A-94).
- **Aislamiento:** los clientes siguen siendo de cada empresa; la tabla común solo guarda lo que devuelve el servicio (datos públicos), nunca datos propios de una empresa ni sus ventas (A-93 ⚖️, principio VIII).
- **Sin conexión en la app:** busca solo en lo ya cargado y avisa que la consulta necesita conexión.
- **Venta en curso guardada** (spec 012): el cliente elegido sigue igual aunque luego se actualice.
- **Seguridad:** el token nunca llega a la app ni a una respuesta; la app nunca llama al servicio directamente.

## Requisitos funcionales

- **RF-001:** La búsqueda por DNI o RUC DEBE mirar primero los clientes de la empresa, luego la tabla común y recién después el servicio.
- **RF-002:** Un cliente traído del servicio DEBE quedar verificado, con la fecha de la consulta; su nombre y (si es RUC) su dirección NO DEBEN editarse a mano (A-92).
- **RF-003:** Consultar un documento de un cliente manual DEBE reemplazar su nombre y dirección por los del servicio y dejarlo verificado, con auditoría.
- **RF-004:** El registro manual DEBE seguir disponible para todo documento que el servicio no resuelva y para carné de extranjería y otros (A-96).
- **RF-005:** El sistema DEBE validar el formato (y el dígito verificador del RUC) antes de consultar, sin gastar consultas.
- **RF-006:** En el MVP NO DEBE haber topes de consultas por empresa ni de la plataforma (A-96). Si el servicio responde que no hay cupo, la búsqueda usa solo lo guardado y ofrece el registro manual.
- **RF-007:** Lo que devuelve el servicio DEBE guardarse en una tabla común de la plataforma, reutilizable por todas las empresas (A-93 ⚖️).
- **RF-008:** «Actualizar datos» DEBE volver a consultar a pedido y actualizar el cliente y la tabla común (A-94).
- **RF-009:** Un RUC de baja o no habido DEBE mostrarse con un aviso, sin bloquear la emisión (A-95 ⚖️).
- **RF-010:** Si el servicio falla, vender DEBE seguir funcionando; la falla se avisa y se ofrece el registro manual.
- **RF-011:** El token DEBE ser secreto de la plataforma (A-96): nunca en el repositorio, en las respuestas ni en la app.
- **RF-012:** La importación de clientes (spec 014) NO DEBE cambiar el nombre ni la dirección de un cliente verificado.
- **RF-013:** Cada consulta al servicio DEBE contarse por día, y el panel de la plataforma DEBE mostrar el uso.

## Entidades clave

- **Cliente** (existe): agrega si está verificado y cuándo se consultó.
- **Documento consultado** (nuevo, de la plataforma): lo que devolvió el servicio para un DNI o RUC (nombre o razón social, dirección, estado y condición del RUC) y cuándo.
- **Uso del servicio** (nuevo): cuántas consultas se hicieron cada día (y de qué empresa, para soporte).

## Criterios de éxito

- **CE-001:** Con un RUC no guardado, desde que se termina de escribir hasta ver la razón social pasan **menos de 3 segundos** con datos móviles.
- **CE-002:** Una búsqueda de un documento ya guardado (en la empresa o en la tabla común) **no gasta** consultas: lo comprueba una prueba automática.
- **CE-003:** Con el servicio caído o sin cupo, se emite una boleta con un cliente registrado a mano (prueba automática y verificación manual).
- **CE-004:** Dos búsquedas simultáneas del mismo documento nuevo gastan una sola consulta y dejan un solo cliente (prueba concurrente en PostgreSQL).

## Fuera de alcance

- Otros servicios de consulta. Pasar al plan de pago de apisperu es solo cambiar el token (A-96).
- Topes de consultas por empresa (A-96).
- Consultar carné de extranjería, pasaporte u otros documentos.
- Actualización automática periódica de los datos (A-94).
- Validar el RUC contra SUNAT en el momento de emitir (lo hace SUNAT al recibir).
- Datos de contacto del cliente (teléfono, correo): no los trae el servicio.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-30 | Spec creada a pedido del usuario (búsqueda por DNI o RUC con apisperu.com, primero la base propia) | — |
| 2026-10-30 | A-92 a A-95 | Verificado no se edita a mano; tabla común de la plataforma (el tope por empresa se descartó en A-96); solo «Actualizar datos» vuelve a consultar; RUC de baja o no habido avisa sin bloquear |
| 2026-10-30 | A-96 | Sin topes en el MVP (plan ilimitado si el uso crece); token de la plataforma; registro manual si el servicio falla; se cuenta el uso por día |
