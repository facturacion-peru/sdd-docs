# Spec 011: Panel de inicio

**Estado:** Aprobada (2026-10-03) · **Versión:** 1.0 · **Fecha:** 2026-10-03
**Depende de:** 003, 005, 007, 010 · **Aclaraciones:** A-52, A-53, A-54

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que al entrar a la aplicación cada usuario vea **de un vistazo cómo va el día y qué requiere su atención**, y llegue en un toque a lo que más hace (vender). Hoy el inicio solo saluda, muestra el estado de SUNAT y tres accesos del administrador.

No es un módulo de reportes (fuera del MVP según la visión): son pocos indicadores del día, orientados a actuar.

## Historias de usuario

### HU-1 · Cómo va el día (P1)

Como usuario de empresa, quiero ver al entrar cuánto se vendió hoy y cuántas ventas fueron, para saber cómo va el negocio sin abrir listados (A-52) (A-53).

**Por qué esta prioridad:** es la información que más se consulta y hoy obliga a revisar listados.
**Prueba independiente:** con ventas registradas hoy y ayer, el inicio muestra el total y la cantidad de hoy, que coinciden con los listados.

**Escenarios de aceptación**

1. **Dado** ventas registradas hoy, **cuando** abro el inicio, **entonces** veo el monto total y la cantidad de ventas de hoy, calculados por el sistema según A-54.
2. **Dado** una venta anulada o un comprobante rechazado, **cuando** veo los totales, **entonces** no se cuentan (A-54).
3. **Dado** ventas de los últimos días, **cuando** abro el inicio, **entonces** veo la evolución de los últimos 7 días (A-52).
4. **Dado** un vendedor, **cuando** abre el inicio, **entonces** ve solo lo que A-53 le permite.
5. **Dado** que son las 23:59 o las 00:01 en Lima, **cuando** se calcula «hoy», **entonces** se usa la fecha de Lima, no la del servidor ni la del dispositivo.

### HU-2 · Lo que requiere atención (P1)

Como usuario de empresa, quiero ver los comprobantes pendientes de envío o rechazados y, si soy administrador, las alertas de inventario, para actuar antes de que sea un problema.

**Escenarios de aceptación**

1. **Dado** comprobantes pendientes o rechazados, **cuando** abro el inicio, **entonces** veo cuántos hay y un enlace al listado ya filtrado.
2. **Dado** productos con stock bajo o lotes por vencer, **cuando** el administrador abre el inicio, **entonces** ve cuántos hay y un enlace a las alertas.
3. **Dado** que no hay nada pendiente, **cuando** abro el inicio, **entonces** se indica que todo está al día, sin tarjetas vacías.

### HU-3 · Acciones rápidas y últimas ventas (P2)

Como vendedor con prisa, quiero empezar una venta con un toque desde el inicio y ver mis últimas ventas, para retomar o reimprimir sin buscar.

**Escenarios de aceptación**

1. **Dado** el inicio, **cuando** lo abro en el celular, **entonces** la acción «Nueva venta» es lo más visible de la pantalla.
2. **Dado** ventas recientes, **cuando** abro el inicio, **entonces** veo las últimas 5 con su número, cliente, monto y estado, y puedo abrir cada una.

## Casos límite

- Empresa nueva sin ventas: los indicadores muestran cero y una invitación a registrar la primera venta.
- SUNAT sin configurar: no se muestran indicadores de comprobantes, sí el estado de SUNAT que ya existe.
- Muchas ventas en el día: los totales los calcula el sistema; el inicio no suma listados paginados.
- Sesión vencida mientras el inicio está abierto: se pide volver a entrar como en el resto de la aplicación.
- Datos de otra empresa: nunca entran en los indicadores.

## Requisitos funcionales

- **RF-001:** El sistema DEBE calcular los indicadores del inicio en el servidor, limitados a la empresa del usuario.
- **RF-002:** «Hoy» y «los últimos 7 días» DEBEN calcularse con la fecha de Lima.
- **RF-003:** Los totales DEBEN seguir la regla de A-54.
- **RF-004:** Lo que ve cada rol DEBE seguir A-53.
- **RF-005:** Cada indicador de atención DEBE enlazar al listado correspondiente ya filtrado.
- **RF-006:** El inicio DEBE funcionar a 360 px con la acción de venta visible sin desplazarse.

## Entidades clave

- **Resumen del inicio:** indicadores calculados para un usuario y un día (no se guardan).

## Criterios de éxito

- **CE-001:** Los totales del inicio coinciden con la suma de los listados para el mismo día y usuario (prueba automatizada con ventas, anulaciones, rechazos y notas de crédito).
- **CE-002:** El inicio carga sus indicadores en una sola petición.
- **CE-003:** Un vendedor llega de abrir la aplicación a una venta nueva en un toque.

## Fuera de alcance

- Reportes por rango de fechas, exportaciones y gráficos por producto o vendedor.
- Indicadores del panel de plataforma.
- Actualización en tiempo real (se refresca al entrar o al recargar).

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-03 | A-52 Indicadores | Ventas de hoy, últimos 7 días, por atender, últimas 5 ventas y «Nueva venta» |
| 2026-10-03 | A-53 Qué ve el vendedor | Solo sus ventas; comprobantes por atender de la empresa; sin alertas de inventario |
| 2026-10-03 | A-54 Qué suma como venta | Tickets no anulados + comprobantes no rechazados ni descartados − notas de crédito; con IGV, en soles |
