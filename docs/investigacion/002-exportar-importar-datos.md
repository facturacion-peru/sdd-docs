# Evaluación: exportar e importar datos de la empresa

**Fecha:** 2026-10-07 · **Para:** [spec 014](../specs/014-exportar-importar/spec.md) · **Preguntas:** A-68 a A-74 en [`aclaraciones.md`](../aclaraciones.md#014--exportar-e-importar-datos)

Pedido del usuario: métodos para exportar e importar datos del sistema, como ventas, productos y clientes. Antes estaba fuera del MVP (A-26: importación masiva con spec propia).

## Estado actual

- **No hay** ninguna exportación ni importación masiva. Lo más cercano es la descarga de un comprobante (PDF, XML y CDR) o de un ticket (PDF), siempre de uno en uno.
- **No hay** librerías de hojas de cálculo en la API ni en el frontend.
- Todo dato de empresa está aislado por `company_id` (principio VIII). Las exportaciones e importaciones deben pasar por el mismo contexto de empresa y quedar en la auditoría.

## Datos candidatos

| Dato | Lo que guarda hoy | Clave natural (única por empresa) | ¿Exportar? | ¿Importar? |
|---|---|---|---|---|
| **Productos** | Código, nombre, tipo (bien/servicio), unidad, precio con IGV, afectación (10/20/30), stock mínimo, controla vencimiento, activo | `code` | Sí | Sí: es el caso de A-26 (empresas que vienen de otro sistema con cientos de productos) |
| **Stock y lotes** | Lote, ingreso, vencimiento, cantidad inicial y restante, **costo** (solo lo ve el administrador, A-27) | — | Sí, como «stock actual» o «inventario valorizado» | Solo como stock inicial al crear el producto (A-72) |
| **Clientes** | Tipo de documento (DNI, CE, RUC), número, nombre y dirección | tipo + número | Sí (son datos personales: A-24 ⚖️) | Sí |
| **Tickets** | Número, estado, vendedor, cliente (texto, sin vínculo con «clientes»), medio de pago, totales y líneas | número | Sí | **No** (ver abajo) |
| **Comprobantes y notas** | Serie, número, estado SUNAT, cliente, bases, IGV, total, líneas, XML y CDR | serie + número | Sí | **No** (ver abajo) |
| **Movimientos de inventario** | Tipo, cantidad, saldos, motivo y origen | — | Opcional (kárdex) | No |

### Por qué no importar ventas

1. **Comprobantes:** un comprobante existe porque se firmó y se envió a SUNAT con un correlativo de esta empresa (principio II). Si se importan comprobantes de otro sistema, quedarían documentos sin XML ni CDR propios, o con correlativos que chocan con las series de la 004.
2. **Tickets:** importarlos afectaría al stock (cada venta descuenta lotes FEFO/FIFO), a la numeración interna y al panel de inicio (011). Para no descontar stock habría que crear ventas «históricas» que no salen del inventario, un tipo de dato nuevo que el sistema no tiene.
3. **Uso real:** quien migra necesita el catálogo, el stock inicial y los clientes para empezar a vender. El historial de ventas suele quedarse en el sistema anterior, en el reporte que ya exportó.

## Riesgos y requisitos que salen de la constitución

- **Aislamiento y auditoría (VIII):** cada exportación e importación queda registrada (quién, qué, cuántas filas y qué filtros). Una importación nunca toca otra empresa, aunque el archivo traiga datos de otra.
- **Roles (A-05, A-27, A-30):** el costo y las ventas de todos los vendedores solo los ve el administrador de empresa. Si el vendedor exportara, vería solo lo que ya ve en pantalla.
- **Datos personales (A-24 ⚖️):** exportar clientes saca DNI, nombres y direcciones del sistema. Conviene limitarlo al administrador y auditarlo.
- **Corrección (II):** la importación valida con las **mismas reglas** que el alta manual (código único, afectación válida, precio decimal, DNI de 8 dígitos y RUC de 11). Nada entra sin validar.
- **Archivos maliciosos (IV):**
  - **Tamaño:** límite de tamaño y de filas.
  - **Fórmulas al abrir en Excel:** al exportar, neutralizar las celdas que empiezan con `=`, `+`, `-` o `@` (*CSV/formula injection*).
  - **Hojas de cálculo:** leer los XLSX sin evaluar fórmulas ni macros.
- **Decimales:** los importes viajan como texto decimal con punto, nunca como coma flotante (trampa de `SUM()` en SQLite, ya conocida).
- **Celulares (VII):** exportar es descargar o compartir un archivo, lo que ya funciona en la app (A-63). Importar es una tarea de escritorio: en el celular basta con elegir un archivo, sin una vista previa compleja.

## Opciones técnicas (para el plan, no para la spec)

- **Formato:** XLSX y CSV. Para generar y leer XLSX por *streaming* (poca memoria), `openspout/openspout` (MIT, sin extensiones extra). `maatwebsite/excel` trae PhpSpreadsheet entero, más pesado de lo necesario. El CSV puede escribirse con PHP nativo (UTF-8 con BOM para que Excel muestre bien las tildes).
- **Síncrono o en cola:** con 5 empresas y límites de filas, basta con hacerlo dentro de la misma petición. Si una exportación de ventas crece, puede pasar a una cola con aviso, sin cambiar la interfaz.
- **Importación en dos pasos:**
  1. El usuario sube el archivo y la API **valida y devuelve una vista previa**: filas nuevas, filas que actualizan y errores por fila y columna.
  2. El usuario confirma y la API aplica todo en **una transacción**.

  Así no quedan importaciones a medias.
- **Plantilla descargable** con las columnas, ejemplos y valores admitidos (afectación 10/20/30, unidades y tipos de documento), para que el usuario no adivine el formato.

## Alcance sugerido para la spec 014

- **Exportar** (administrador de empresa): productos con stock, clientes y ventas (tickets y comprobantes, con sus líneas) por rango de fechas.
- **Importar** (administrador de empresa): productos (con stock inicial opcional) y clientes, desde la plantilla, con vista previa y confirmación.
- **Fuera de alcance:**
  - importar ventas;
  - formatos oficiales de SUNAT (PLE / registro de ventas: ⚖️, otra spec para el contador);
  - exportar desde el panel de la plataforma (A-37: la plataforma no ve los datos de las empresas);
  - respaldo completo de la empresa (lo cubre la 009).
