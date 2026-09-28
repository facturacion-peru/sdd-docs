# Visión del producto

**Estado:** vigente · **Origen:** `Mi-idea-sas-facturacion.txt` (2026-09-24) · **Última revisión:** 2026-09-26

Contexto de negocio del SaaS. **No es una especificación**: no se implementa nada directamente desde aquí. Sirve para escribir las specs de `docs/specs/` y para decidir qué entra y qué no en el MVP. Las dudas abiertas están en [`../aclaraciones.md`](../aclaraciones.md).

## 1. Qué es

Un SaaS para pequeñas empresas del Perú que permite:

1. Registrar una empresa y sus usuarios.
2. Emitir comprobantes electrónicos y enviarlos a SUNAT cuando la empresa configure esa integración.
3. Emitir tickets internos para negocios que todavía no quieren o no pueden conectarse a SUNAT.
4. Administrar productos, existencias y lotes de inventario.
5. Operar varias empresas en la misma plataforma, con sus datos aislados.

Prioridades del producto, en este orden: **exactitud tributaria**, **seguridad de credenciales** y **rapidez de uso desde el celular y la computadora**.

## 2. Punto de partida técnico

| Proyecto | Rol |
|---|---|
| `Api-de-facturacion-electronica-sunat-Peru` | API Laravel con avances en facturación SUNAT. Punto de partida **sujeto a evaluación**. |
| `frontend-api-facturacion-electronica-sunat` | Frontend Vue que consume la API por su contrato OpenAPI. |

La decisión de adaptar o reiniciar la API se toma tras la evaluación de [`../investigacion/001-evaluacion-api-existente.md`](../investigacion/001-evaluacion-api-existente.md). No se reescribe por preferencia ni se conserva código solo por el avance aparente.

**Criterios de viabilidad** para aprovechar la API existente:

- Separación confiable por empresa.
- Autorización por usuario y rol.
- Manejo seguro de credenciales y certificados.
- Cálculos y emisión fiscal cubiertos por pruebas.
- Contrato OpenAPI mantenible.
- Capacidad de evolucionar sin romper el frontend.

## 3. Personas

| Persona | Qué hace | Pertenece a una empresa |
|---|---|---|
| **Administrador de la plataforma** | Opera el SaaS: empresas, usuarios globales, estado de integraciones, soporte | No |
| **Administrador de empresa** | Configura su empresa: datos, usuarios, series, productos, inventario, SUNAT | Sí |
| **Vendedor / operador** | Registra ventas y consulta lo que su rol permita | Sí |

El administrador de la plataforma y el administrador de una empresa son roles distintos y no deben confundirse.

## 4. Alcance del MVP

El primer MVP demuestra un flujo completo y pequeño:

1. Crear una empresa y su usuario administrador.
2. Aislar los datos de esa empresa.
3. Registrar productos y entradas de stock por lote.
4. Emitir un ticket interno.
5. Configurar SUNAT en ambiente beta, con una serie.
6. Emitir un comprobante en beta, consultar su estado y mostrar el resultado.
7. Registrar los errores y permitir recuperar una operación pendiente.

**Fuera del MVP** mientras no se especifique expresamente: suscripciones y cobros del SaaS, contabilidad, compras y proveedores, reportes avanzados, varios almacenes, impresión Bluetooth, app móvil nativa, ambiente de producción SUNAT y todos los tipos de comprobante que no sean los del MVP.

## 5. Mapa de capacidades

Cada capacidad se convierte en una spec. El estado vive en [`../specs/README.md`](../specs/README.md).

| # | Capacidad | Punto del MVP |
|---|---|---|
| 001 | Registro de empresa, usuarios, roles y aislamiento multiempresa | 1, 2 |
| 002 | Productos, lotes y movimientos de inventario | 3 |
| 003 | Tickets de venta internos | 4 |
| 004 | Configuración segura de SUNAT y series | 5 |
| 005 | Emisión y consulta de comprobantes en beta | 6, 7 |
| 006 | Panel de administración de la plataforma | Parcial (lo mínimo lo cubre 001) |

## 6. Principios de producto

Lo que la [constitución](../constitution.md) ya exige no se repite; esto es lo específico del negocio:

- **El ticket interno nunca parece un comprobante.** No se titula "boleta" ni "factura", no usa series SUNAT y dice expresamente que no es un comprobante de pago.
- **El stock no cambia en silencio.** Cada cambio es un movimiento con responsable, fecha y motivo; los errores se corrigen con movimientos inversos, nunca borrando.
- **Toda acción sensible queda auditada:** configuración, credenciales, series, comprobantes, inventario y usuarios.
- **Beta y producción nunca se mezclan**, y la interfaz siempre muestra en cuál se está.
- **Tamaño inicial:** al menos 5 empresas cliente. El aislamiento es obligatorio desde el primer incremento, aunque la solución no sea una plataforma multi-tenant compleja.

## 7. Glosario

Términos que se usan igual en specs, código y pantallas.

| Término | Significado |
|---|---|
| **Empresa** | Cliente del SaaS y dueña de sus datos. Unidad de aislamiento (*tenant*). Se identifica por su RUC. |
| **RUC** | Registro Único de Contribuyentes, 11 dígitos. Empieza por `10` (persona natural con negocio) o `20` (persona jurídica). |
| **Régimen tributario** | NRUS, RER, RMT o Régimen General. Condiciona qué comprobantes puede emitir la empresa. |
| **Establecimiento** | Local de la empresa registrado en SUNAT (código de anexo). Las series se asocian a él. |
| **Comprobante** | Comprobante de pago electrónico válido ante SUNAT: factura (`01`), boleta (`03`), etc. |
| **Serie** | Identificador de 4 caracteres de una secuencia de comprobantes: `F001` (factura), `B001` (boleta). |
| **Correlativo** | Número secuencial dentro de una serie. No se repite ni se salta. |
| **CDR** | Constancia de Recepción que SUNAT devuelve: aceptado, aceptado con observaciones o rechazado. |
| **Ambiente** | `beta` (pruebas de SUNAT, sin valor legal) o `producción`. |
| **Clave SOL** | Usuario y contraseña de SUNAT Operaciones en Línea de la empresa. Es un secreto. |
| **Certificado digital** | Archivo `.pfx`/`.pem` con la clave privada para firmar el XML. Es un secreto. |
| **Ticket interno** | Documento de venta **no tributario** con numeración propia. No es comprobante de pago. |
| **Producto** | Bien o servicio que vende la empresa. |
| **Lote** | Conjunto de unidades de un producto que entraron juntas, con fecha y, si aplica, vencimiento. |
| **Movimiento** | Registro inmutable de un cambio de stock: entrada, salida, ajuste, devolución o reversión. |
| **Auditoría** | Registro de quién hizo qué acción sensible, cuándo y sobre qué. |
