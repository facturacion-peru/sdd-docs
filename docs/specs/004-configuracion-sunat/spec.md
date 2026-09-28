# Spec 004: Configuración segura de SUNAT y series

**Estado:** En aclaración · **Versión:** 0.2 · **Fecha:** 2026-09-26
**Depende de:** 001 · **Aclaraciones:** A-07, A-11, A-20, A-21

> Los requisitos marcados con `[A-xx]` siguen la recomendación de esa aclaración y se ajustarán si la decisión es otra.
> La **emisión** de comprobantes está en la spec 005; esta spec deja la empresa lista para emitir.

## Objetivo

Que el administrador de empresa configure la conexión con SUNAT (clave SOL, certificado, ambiente y series) de forma **segura y validada**, de modo que la empresa solo pueda emitir cuando todo está correcto y **sin que ningún secreto salga nunca del servidor**.

## Historias de usuario

### HU-1 · Registrar credenciales y certificado (P1)

Como administrador de empresa, quiero registrar mi usuario y clave SOL y subir mi certificado digital, para que el sistema pueda firmar y enviar en mi nombre.

**Prueba independiente:** subir un certificado válido y otro con contraseña incorrecta, y consultar la configuración.

**Escenarios de aceptación**

1. **Dado** que soy administrador de empresa, **cuando** registro usuario SOL, clave SOL, certificado (`.pfx` o `.pem`) y su contraseña, **entonces** se guardan protegidos y la configuración pasa a "pendiente de validación".
2. **Dado** un certificado con contraseña incorrecta, dañado o vencido, **cuando** lo subo, **entonces** el sistema lo rechaza y dice exactamente cuál es el problema `[A-21]`.
3. **Dado** un certificado emitido para otro RUC, **cuando** lo subo, **entonces** el sistema lo rechaza `[A-21]`.
4. **Dado** una configuración guardada, **cuando** cualquier usuario la consulta, **entonces** ve el usuario SOL parcialmente enmascarado, si hay clave registrada (sí/no), y del certificado su titular, RUC y fecha de vencimiento, pero **nunca** la clave, el certificado ni su contraseña.
5. **Dado** que soy vendedor, **cuando** intento ver o cambiar la configuración SUNAT, **entonces** el sistema lo impide.

### HU-2 · Validar la configuración (P1)

Como administrador de empresa, quiero probar la conexión antes de emitir, para saber si todo funciona.

**Escenarios de aceptación**

1. **Dado** una configuración completa, **cuando** pido validarla, **entonces** el sistema comprueba el certificado y la conexión con SUNAT beta, y la marca "validada" o "con error" con el motivo `[A-20]`.
2. **Dado** una configuración incompleta, **cuando** pido validarla, **entonces** el sistema lista exactamente qué falta.
3. **Dado** SUNAT sin responder, **cuando** valido, **entonces** la configuración no se marca como errónea: se informa que SUNAT no está disponible y que se reintente después.
4. **Dado** una configuración validada, **cuando** cambio la clave SOL o el certificado, **entonces** vuelve a "pendiente de validación" y no se puede emitir hasta validarla de nuevo.

### HU-3 · Series y correlativos (P1)

Como administrador de empresa, quiero definir mis series de factura y boleta, incluso continuando la numeración de mi sistema anterior, para emitir sin duplicar números `[A-21]`.

**Escenarios de aceptación**

1. **Dado** que soy administrador de empresa, **cuando** creo la serie `F001` para facturas indicando que el último número usado fue 150, **entonces** el siguiente comprobante será el 151.
2. **Dado** una serie de factura, **cuando** intento nombrarla sin empezar por `F`, o una de boleta sin empezar por `B`, o con un formato distinto de 4 caracteres, **entonces** el sistema lo rechaza.
3. **Dado** una serie que ya existe en mi empresa, **cuando** intento crearla de nuevo, **entonces** el sistema lo rechaza.
4. **Dado** una serie con comprobantes emitidos, **cuando** intento bajar su correlativo o borrarla, **entonces** el sistema lo impide; solo puedo desactivarla.
5. **Dado** una serie, **cuando** consulto su estado, **entonces** veo el último número emitido y el siguiente.
6. **Dado** una empresa en Nuevo RUS, **cuando** intento crear una serie de factura, **entonces** el sistema lo impide `[A-11]`.

### HU-4 · Estado de la integración (P2)

Como usuario de empresa, quiero ver si puedo emitir comprobantes y en qué ambiente, para no equivocarme.

**Escenarios de aceptación**

1. **Dado** cualquier estado de la configuración (no configurada, pendiente de validación, validada, con error, inactiva), **cuando** entro a la aplicación, **entonces** veo si la emisión SUNAT está disponible y, si no, qué falta.
2. **Dado** el ambiente beta, **cuando** emito o consulto comprobantes, **entonces** la interfaz muestra siempre un distintivo visible de "PRUEBAS — sin valor legal".
3. **Dado** un certificado que vence en 30 días o menos, **cuando** el administrador entra, **entonces** ve un aviso `[A-21]`.

## Casos límite

- Se sube un certificado nuevo mientras hay comprobantes enviándose: los envíos en curso terminan con el certificado anterior.
- La empresa se desactiva: la configuración queda inactiva y no se puede emitir.
- Se intenta cambiar de beta a producción: no está disponible en el MVP `[A-20]`.
- Dos administradores editan la configuración a la vez: gana el último, y ambos cambios quedan auditados.

## Requisitos funcionales

**Credenciales y certificado**

- **RF-001:** El sistema DEBE guardar la clave SOL, el certificado y su contraseña **cifrados**, fuera de cualquier ubicación accesible públicamente y separados por empresa.
- **RF-002:** Ningún secreto DEBE aparecer en respuestas de la API, logs, mensajes de error, registros de auditoría ni en el frontend. Solo se exponen metadatos: usuario enmascarado, titular, RUC y vencimiento del certificado.
- **RF-003:** El sistema DEBE validar el certificado al recibirlo: contraseña correcta, no vencido y RUC coincidente con la empresa `[A-21]`.
- **RF-004:** Los secretos solo DEBEN descifrarse en el servidor, en el momento de firmar o enviar.
- **RF-005:** Solo el administrador de empresa DEBE poder registrar, reemplazar o eliminar credenciales.

**Validación y estados**

- **RF-010:** La configuración DEBE tener los estados: no configurada, pendiente de validación, validada, con error e inactiva.
- **RF-011:** Una empresa solo DEBE poder emitir comprobantes si su configuración está "validada".
- **RF-012:** Cambiar una credencial o el certificado DEBE devolver la configuración a "pendiente de validación".
- **RF-013:** En el MVP, el único ambiente disponible DEBE ser beta `[A-20]`.

**Series**

- **RF-020:** El sistema DEBE gestionar series por tipo de comprobante (factura, boleta) y establecimiento, cada una con su correlativo independiente `[A-07]` `[A-21]`.
- **RF-021:** Las series DEBEN validar el formato SUNAT: 4 caracteres, empezando por `F` para factura y `B` para boleta.
- **RF-022:** Al crear una serie DEBE poder indicarse el último número ya usado. Después, el correlativo solo lo avanza la emisión.
- **RF-023:** El correlativo NO DEBE poder retroceder ni repetirse, ni siquiera con emisiones concurrentes.
- **RF-024:** Una serie con comprobantes NO DEBE borrarse; solo desactivarse.
- **RF-025:** El sistema NO DEBE permitir series de factura a empresas del Nuevo RUS `[A-11]`.

**Auditoría**

- **RF-030:** DEBEN auditarse el registro y el reemplazo de credenciales y certificados, las validaciones y su resultado, y la creación, desactivación y cambio de series, sin registrar valores secretos.

## Entidades clave

- **Configuración SUNAT:** estado, ambiente, fecha y resultado de la última validación de una empresa.
- **Credencial SOL:** usuario y clave SOL cifrada de la empresa.
- **Certificado digital:** archivo cifrado con titular, RUC, vigencia y estado (vigente o reemplazado).
- **Serie:** tipo de comprobante, código, establecimiento, último correlativo y estado.

## Criterios de éxito

- **CE-001:** Un administrador configura y valida SUNAT beta en menos de 5 minutos si tiene sus datos a mano.
- **CE-002:** Una búsqueda de la clave SOL, la contraseña del certificado y el contenido del certificado en respuestas, logs y auditoría, tras un recorrido completo de pruebas, no encuentra nada.
- **CE-003:** Con 50 emisiones concurrentes sobre una misma serie no hay correlativos duplicados ni huecos.

## Fuera de alcance

- Ambiente de producción `[A-20]`.
- Credenciales de la API REST de SUNAT para guías de remisión (GRE).
- Varios establecimientos `[A-07]` y certificados de terceros (OSE/PSE).
- Renovación automática de certificados.
- Series de notas de crédito o débito y de otros comprobantes: llegan con su spec.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-09-26 | El título del borrador incluía "y emisión fiscal" | La emisión pasa a la spec 005; esta cubre solo configuración y series |
| 2026-09-26 | El borrador ocultaba también el usuario SOL | Se muestra enmascarado: no es secreto por sí solo y ayuda a reconocer la cuenta. La clave sí es secreta |
