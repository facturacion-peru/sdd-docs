# Verificación 002: Productos, lotes e inventario

**Fecha:** 2026-09-28 · **Tarea:** T092 · **Rama:** `002-productos-inventario`

## Recorrido de extremo a extremo (por el proxy de Vite, datos demo)

16/16 pasos correctos:
- Alta de producto.
- Entrada solo con cantidad (lote automático, 51 ms de API) y entrada con costo y lote propio.
- El vendedor ve precio y stock pero no el costo, y recibe 403 en entradas e historial.
- Venta por el servicio: FIFO sobre dos lotes y rechazo sin stock suficiente sin descontar nada.
- Alerta de stock bajo.
- Ajuste negativo, rechazo del ajuste que dejaría el lote en negativo, reversión y rechazo de la doble reversión.
- Historial completo.
- 404 desde otra empresa, idéntico a un recurso inexistente.
- Aviso al desactivar un producto con stock.

## Pantallas a 360 px

Capturas con Chrome headless contra el servidor de Vite, con sesión real de las cuentas demo (2026-09-28):

| Pantalla | Resultado |
|---|---|
| Catálogo (vendedor) | ✅ Precio, disponible, «Stock bajo» y «Servicio»; sin «Nuevo producto», sin filtro de desactivados, sin costo |
| Ficha (administrador) | ✅ Precio, disponible, stock físico y último costo; lotes con vencimiento y costo; historial con «Revertir» |
| Entrada rápida | ✅ Foco en la cantidad; vencimiento visible porque el producto lo controla; datos opcionales plegados |
| Alertas | ✅ Stock bajo y próximos a vencer con días restantes |
| Nuevo producto | ✅ Campos legibles; ayuda del stock mínimo visible |

**Observación:** con seis entradas, la navegación del administrador no cabe a 360 px: «Usuarios» y «Auditoría» quedan detrás de un desplazamiento horizontal. Se puede usar, pero conviene agrupar entradas cuando lleguen las ventas (spec 003).

## Pendiente de confirmación del usuario

- CE-003 (entrada en menos de 30 s): el formulario exige un solo campo y la API responde en unos 50 ms, pero falta que una persona lo cronometre en el celular.
