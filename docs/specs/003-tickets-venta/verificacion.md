# Verificación 003: Tickets de venta internos

**Fecha:** 2026-09-28 · **Tarea:** T082 · **Rama:** `003-tickets-venta`

## Recorrido de extremo a extremo (por el proxy de Vite, datos demo)

13/13 pasos correctos. La primera ejecución marcó 4 fallas que eran errores de las expectativas del script (un total mal calculado y un stock inicial supuesto), no de la aplicación; con el stock leído antes de vender, todo coincide.

| # | Paso | Spec | Resultado |
|---|---|---|---|
| 1 | Venta de 3 productos (con descuento y un servicio) | HU-1.1 | ✅ total S/ 78.80, 64 ms de API |
| 2 | Reintento con la misma clave | RF-008 | ✅ mismo ticket, sin descontar otra vez |
| 3 | Leyenda y número propio | HU-2 | ✅ |
| 4 | Sin stock suficiente | HU-1.3 | ✅ 422 con el disponible |
| 5 | La venta rechazada no consume número | RF-002 | ✅ T-000007 → T-000008 |
| 6 | El vendedor solo ve sus tickets | HU-4.2 | ✅ |
| 7 | El vendedor no ve el ticket del administrador | HU-4.2 | ✅ 404 |
| 8 | El vendedor no anula | HU-3.2 | ✅ 403 |
| 9 | Anular devuelve el stock y conserva el número | HU-3.1 | ✅ |
| 10 | No se anula dos veces | HU-3.3 | ✅ |
| 11 | No se revierte a mano una venta de ticket | Plan 003 | ✅ |
| 12 | Totales del día por medio de pago sin anulados | HU-4.1 | ✅ |
| 13 | Otra empresa: 404 idéntico a inexistente | Principio VIII | ✅ |

## Pantallas a 360 px

| Pantalla | Resultado |
|---|---|
| Navegación | ✅ Cabe a 360 px: el vendedor ve Inicio, Vender, Ventas y Productos; el administrador, además, «Más ▾» |
| Vender (vendedor) | ✅ |
| Ventas (vendedor) | ✅ Filtros, totales por medio de pago, anulados tachados |
| Ticket (administrador) | ✅ Leyenda arriba y abajo, «Imprimir» y «Anular» |
| Ticket anulado | ✅ Franja «ANULADO», sin botón de anular |
| Impresión (PDF) | ✅ Solo el ticket, sobre fondo blanco (el fondo gris se imprimía y se corrigió) |

## Pendiente de confirmación del usuario

- **CE-001:** cronometrar una venta de 3 productos en el celular (objetivo: menos de 30 s).
- **Impresión en 80 mm:** Chrome headless ignora `@page` y genera el PDF en tamaño carta. El diálogo de impresión del navegador sí respeta los 80 mm, pero conviene probarlo con una impresora térmica real.
- **Datos de prueba:** el recorrido dejó en la base local tickets de prueba (T-000005 a T-000008, dos de ellos anulados). Se limpian con `php artisan migrate:fresh --seed`.
