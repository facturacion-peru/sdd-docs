# Verificación 002: Productos, lotes e inventario

**Fecha:** 2026-09-28 · **Tarea:** T092 (parcial) · **Rama:** `002-productos-inventario`

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

## Pendiente

- Capturas a 360 px ya tomadas (catálogo del vendedor, ficha, entrada, alertas, nuevo producto); **aún sin revisar a la vista**.
- CE-003 (entrada en menos de 30 s): el formulario exige un solo campo, pero falta que una persona lo cronometre.
