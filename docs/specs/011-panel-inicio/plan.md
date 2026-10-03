# Plan 011: Panel de inicio

**Estado:** Aprobado (2026-10-03) · **Fecha:** 2026-10-03 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

## Resumen técnico

Un endpoint nuevo de solo lectura, `GET /api/v1/dashboard`, calcula en el servidor todos los indicadores del inicio en una sola respuesta (CE-002), limitado por el `TenantContext` y por el rol (A-53). El frontend crea la feature `dashboard` (api, tipos y componentes) y el `HomeView` de `app/views` la compone junto con el `SunatStatusCard` existente. El gráfico de 7 días son barras con HTML y CSS, sin librerías.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | API y frontend |
| Ramas | `011-panel-inicio` en ambos. Frontend desde `010-identidad-visual` (usa los tokens de estado); API desde `009-despliegue-operacion` |
| Dependencias nuevas | Ninguna |
| Migraciones | Ninguna. Se revisan los índices sobre `issued_at` de `tickets` y `sales_documents`; si faltan, se añade una migración solo de índices |

## Chequeo de constitución

| Principio | Cumple | Cómo / justificación de la excepción |
|---|---|---|
| I. Spec primero | Sí | Spec 011 v1.0 aprobada con A-52 a A-54 |
| II. Corrección fiscal | Sí | Los totales los calcula la API con `Decimal::sum()` (no `SUM()` de SQLite); el frontend solo los muestra. Se rotulan como indicador operativo (A-54) |
| III. Contrato de API | Sí | Ruta nueva bajo `/api/v1`, sobre `{ success, data }`, prueba de contrato en `tests/Feature/Contract` y `pnpm api:types` |
| IV. Seguridad | Sí | Solo lectura, autenticada, con `tenant`; sin datos personales más allá del nombre del cliente que ya muestran los listados |
| V. Simplicidad | Sí | Un servicio (`DashboardService`); las alertas pasan de `AlertController` a `InventoryAlertService` para reutilizarlas, sin cambiar su respuesta |
| VI. Verificable | Sí | Pest (SQLite y PostgreSQL), Vitest, y recorrido manual a 360 px |
| VII. Personas reales | Sí | «Nueva venta» primero en el móvil, estados vacíos y fecha de Lima |
| VIII. Aislamiento y auditoría | Sí | Ruta en `TenantRoutes.php` (tipo `list`) y prueba de acceso cruzado: los datos de otra empresa nunca suman. Es lectura: no se audita |

## Reglas de cálculo

- **Día:** fecha de `America/Lima` (`config('app.timezone')`). «Hoy» es `[hoy 00:00, hoy 23:59:59]` sobre `issued_at`; los 7 días van de hoy−6 a hoy, con ceros en los días sin ventas.
- **Ventas (A-54):** tickets `issued` (no anulados) + boletas y facturas en `pending`, `sent`, `accepted` u `observed` (no `rejected` ni `discarded`).
- **Notas de crédito:** las que tienen esos mismos estados restan su `total` el día en que se emiten. No cuentan como venta en la cantidad.
- **Cantidad:** tickets + boletas y facturas que cumplen la regla.
- **Vendedor (A-53):** tickets y comprobantes con `seller_id` = el usuario. Las notas de crédito restan si el comprobante que modifican es suyo. Sin alertas de inventario (`null`).
- **Por atender:** comprobantes de toda la empresa (A-32), con el mismo criterio que el listado: `pending` + `sent` como pendientes y `rejected` como rechazados.
- **Últimas ventas:** las 5 más recientes entre tickets y comprobantes (incluidas las notas), con el mismo alcance por rol.

## Contrato de API

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/api/v1/dashboard` | Administrador y vendedor | Resumen del inicio |

```json
{
  "success": true,
  "data": {
    "date": "2026-10-03",
    "scope": "company",
    "today": { "total": "1250.40", "count": 18 },
    "last_7_days": [{ "date": "2026-09-27", "total": "980.00", "count": 12 }],
    "attention": { "pending_documents": 2, "rejected_documents": 1, "inventory_alerts": 3 },
    "recent_sales": [
      { "kind": "ticket", "id": 41, "number": "T-000041", "customer_name": null, "total": "35.00", "status": "issued", "issued_at": "2026-10-03T10:21:00-05:00" },
      { "kind": "document", "id": 9, "number": "B001-00000009", "document_type": "03", "customer_name": "Cliente varios", "total": "120.00", "status": "accepted", "issued_at": "…" }
    ]
  }
}
```

`scope` vale `company` (administrador) u `own` (vendedor). Los montos son cadenas decimales con 2 decimales. `inventory_alerts` es `null` para el vendedor y suma stock bajo, lotes por vencer y lotes vencidos.

## API

- `app/Services/DashboardService.php`: calcula el resumen con las reglas de arriba.
- `app/Services/InventoryAlertService.php`: lógica actual de `AlertController::index`. El controlador la usa sin cambiar su respuesta, y el panel solo la cuenta.
- `app/Http/Controllers/Api/DashboardController.php` (invocable) y ruta en el grupo de empresa (no en el de solo administrador).
- La respuesta se tipa a mano en el frontend (el OpenAPI no documenta respuestas), protegida por `tests/Feature/Contract/DashboardResourceTest.php`.

## Frontend

- `features/dashboard/`: `api.ts`, `types.ts` y los componentes `TodaySummary.vue`, `WeekChart.vue` (barras HTML/CSS, ocultas a lectores de pantalla, con una tabla de datos para ellos), `AttentionList.vue` y `RecentSales.vue`.
- `app/views/HomeView.vue`: compone el panel (su prueba está en `app/views/__tests__`). En el móvil va primero «Nueva venta» (botón grande), luego Hoy, Por atender, 7 días y Últimas ventas, y al final el `SunatStatusCard` y los accesos del administrador. En escritorio, una cuadrícula de 2 columnas.
- Los enlaces van por nombre de ruta, sin importar otras features: `sales-documents` con `?status=pending|rejected`, `/inventario/alertas`, y `ticket-detail` o `sales-document-detail`.
- `SalesDocumentsView` lee `status` de la URL al abrirse (hoy solo lo toma del formulario). Es un cambio pequeño en la feature `sales`.
- Estados de carga (esqueleto), error con «Reintentar» y vacío («Registra tu primera venta»).

## Estrategia de pruebas

| Requisito / escenario | Prueba | Tipo |
|---|---|---|
| RF-001, CE-001 | `DashboardTest`: con tickets, anulados, comprobantes en cada estado y notas de crédito, los totales coinciden con la regla y con los listados | Pest (SQLite y PostgreSQL) |
| RF-002, HU-1 esc. 5 | Ventas a las 23:59 y 00:01 de Lima con `Carbon::setTestNow` | Pest |
| RF-004, A-53 | El vendedor ve solo lo suyo y `inventory_alerts` es `null`; el administrador ve todo | Pest |
| Principio VIII | `TenantRoutes.php` (`list`) más una prueba de que las ventas de otra empresa no suman | Pest |
| Contrato | `DashboardResourceTest` | Pest |
| Alertas sin cambios | Las pruebas actuales de alertas siguen en verde tras extraer el servicio | Pest |
| HU-2, HU-3, casos límite | Componentes y `HomeView` (carga, vacío, error, vendedor y administrador) | Vitest |
| RF-005 | `SalesDocumentsView` aplica `?status=` de la URL | Vitest |
| RF-006, CE-003 | Capturas a 360 px y recorrido manual | Manual |

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Lentitud con muchas ventas | Consultas por rango de `issued_at` con índice; se mide con 10 000 tickets en PostgreSQL |
| Confundir el indicador con un reporte contable | Rótulo «Ventas de hoy (incluye IGV, descuenta notas de crédito)» y A-54 en la spec |
| Diferencias SQLite/PostgreSQL en sumas | `Decimal::sum()` en PHP y la suite de PostgreSQL |
