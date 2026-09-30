# Spec 007: Anulación y corrección de comprobantes

**Estado:** Borrador para aprobación · **Versión:** 0.2 · **Fecha:** 2026-09-30
**Depende de:** 002, 004, 005 · **Aclaraciones:** A-22 ⚖️, A-39 ⚖️, A-40, A-41, A-42

> ⚖️ Los motivos y el uso de la nota de crédito deben revisarse con un contador antes de producción (A-39).

## Objetivo

Que una empresa pueda **corregir o anular** una factura o boleta ya aceptada por SUNAT (por devolución, error o anulación de la venta) con el instrumento que exige SUNAT, y que el stock y los reportes reflejen la corrección. Hoy, en beta, un comprobante emitido no se puede anular (A-22); en producción eso no es aceptable.

También resuelve un vacío de la 005: qué pasa con la venta y el stock de un comprobante **rechazado** por SUNAT (A-42).

## Historias de usuario

### HU-1 · Anular una venta con nota de crédito (P1)

Como usuario de la empresa (administrador o vendedor), quiero anular un comprobante aceptado emitiendo una nota de crédito, para dejar sin efecto una venta que no debió registrarse (A-39) (A-41).

**Escenarios de aceptación**

1. **Dado** una factura o boleta aceptada, **cuando** emito una nota de crédito por «anulación de la operación» con un motivo escrito, **entonces** la nota referencia al comprobante, lleva los mismos importes, se envía a SUNAT como la 005 y el comprobante queda marcado como «anulado con nota de crédito».
2. **Dado** la anulación, **cuando** indico si la mercadería volvió, **entonces** el stock se repone o no según lo que indiqué (A-40).
3. **Dado** un comprobante ya anulado, **cuando** intento anularlo otra vez, **entonces** el sistema lo impide.

### HU-2 · Devolución total o parcial (P1)

Como usuario de la empresa (administrador o vendedor), quiero registrar la devolución de todos o algunos productos de un comprobante, para reembolsar al cliente y reponer el stock (A-39) (A-40).

**Escenarios de aceptación**

1. **Dado** una factura o boleta aceptada, **cuando** elijo qué líneas y qué cantidades se devuelven, **entonces** se emite una nota de crédito por «devolución» con los importes de esas cantidades, calculados con las reglas de la 005, y el stock de esos productos se repone.
2. **Dado** devoluciones anteriores, **cuando** registro otra, **entonces** no puedo devolver más de lo que queda por devolver de cada línea.
3. **Dado** una devolución de todas las cantidades, **cuando** se acepta, **entonces** el comprobante queda «devuelto totalmente».

### HU-3 · Comprobante rechazado por SUNAT (P1)

Como administrador, quiero resolver un comprobante que SUNAT rechazó, para que la venta y el stock no queden en un estado incoherente (A-42).

**Escenarios de aceptación**

1. **Dado** un comprobante rechazado, **cuando** lo doy de baja interna con un motivo, **entonces** queda «descartado», su stock se repone y su número queda registrado como usado (no se reutiliza).
2. **Dado** un comprobante descartado, **cuando** el vendedor vuelve a emitir la venta con los datos corregidos, **entonces** se emite un comprobante nuevo con otro número.

### HU-4 · Consultar notas y estado de corrección (P2)

Como usuario de empresa, quiero ver en cada comprobante sus notas de crédito y cuánto queda vigente, para saber qué se devolvió o anuló.

**Escenarios de aceptación**

1. **Dado** un comprobante con notas, **cuando** lo abro, **entonces** veo cada nota con su estado SUNAT, su motivo, sus importes y el saldo vigente del comprobante.
2. **Dado** una nota de crédito, **cuando** la abro, **entonces** veo el comprobante que modifica y descargo su PDF, XML y CDR.

## Casos límite

- Nota de crédito sobre un comprobante pendiente o rechazado: no se permite; primero debe estar aceptado (o descartarse, HU-3).
- SUNAT no responde al enviar la nota: queda pendiente y se reintenta como en la 005; el comprobante se marca como anulado o devuelto recién cuando la nota es aceptada.
- SUNAT rechaza la nota: el comprobante no cambia y la nota queda rechazada con su motivo; el stock repuesto por esa nota se revierte.
- Devolución de un producto que ya no está activo en el catálogo: se permite; el stock se repone igual.
- Dos administradores devuelven la misma línea a la vez: la segunda devolución respeta lo que ya se devolvió.

## Requisitos funcionales

- **RF-001:** El sistema DEBE emitir notas de crédito electrónicas (tipo 07) que referencien una factura o boleta aceptada de la misma empresa (A-39).
- **RF-002:** Los motivos permitidos DEBEN ser 01 (anulación de la operación), 06 (devolución total) y 07 (devolución por ítem) del catálogo 09 de SUNAT (A-39) ⚖️. Cada nota lleva además una explicación escrita.
- **RF-003:** La nota DEBE usar una serie propia del tipo del comprobante que modifica (empieza por F para facturas y por B para boletas) y su propio correlativo.
- **RF-004:** Los importes de la nota DEBEN calcularse en el servidor con las mismas reglas de la 005, a partir de las líneas y cantidades del comprobante original.
- **RF-005:** La suma de lo devuelto o anulado NO DEBE superar lo emitido, por línea y en total.
- **RF-006:** El envío, los reintentos, los estados, el XML, el CDR y el PDF de la nota DEBEN seguir la 005 (RF-010 a RF-014, RF-007).
- **RF-007:** El stock DEBE reponerse según A-40, con un movimiento de inventario que referencie a la nota.
- **RF-008:** Cualquier usuario de la empresa DEBE poder emitir notas de crédito sobre los comprobantes de su empresa (A-41, A-32); solo el administrador DEBE poder descartar comprobantes rechazados (A-42).
- **RF-009:** Todo DEBE auditarse: emisión de notas, descarte de rechazados y reposición de stock.

## Entidades clave

- **Nota de crédito:** documento electrónico tipo 07 con serie, correlativo, comprobante modificado, motivo, líneas e importes, estado SUNAT, XML y CDR.
- **Estado de corrección del comprobante:** vigente, devuelto parcialmente, devuelto totalmente, anulado o descartado.

## Criterios de éxito

- **CE-001:** Notas de crédito de anulación y de devolución parcial, sobre facturas y boletas, **aceptadas por SUNAT beta**.
- **CE-002:** Tras cualquier combinación de devoluciones, el stock y el saldo vigente del comprobante cuadran con lo emitido menos lo devuelto.
- **CE-003:** Un administrador registra la devolución de un producto en menos de 1 minuto.

## Fuera de alcance

- Notas de débito (A-39).
- Comunicación de baja y resumen diario (A-39).
- Descuentos o bonificaciones posteriores a la venta mediante nota de crédito (A-39).
- Reembolso del dinero: el sistema registra la nota; el pago al cliente se hace fuera.
- Paso a producción (spec 008).

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-09-30 | Spec creada: corrección de comprobantes, fuera del MVP beta por A-22 | — |
| 2026-09-30 | A-39 ⚖️ | Solo nota de crédito con motivos 01, 06 y 07 |
| 2026-09-30 | A-40 | La devolución repone el stock; en la anulación se pregunta (por defecto, sí) |
| 2026-09-30 | A-41 | También el vendedor emite notas de crédito, auditado |
| 2026-09-30 | A-42 | El administrador descarta los rechazados y se repone el stock |
