# Plan 012: Vender — carrito persistente y diseño en dos columnas

**Estado:** Aprobado (2026-10-03) · **Fecha:** 2026-10-03 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.3)

## Resumen técnico

Solo frontend: la API ya ofrece todo (`GET /products` paginado con `search` y `status`, y `GET /products/{id}` para todos los usuarios de empresa). La venta en curso pasa de variables locales de `NewSaleView` a un store de Pinia de la feature `sales`, que la guarda en `localStorage` por empresa y usuario. La pantalla se divide en componentes: catálogo, carrito y barra móvil.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | Frontend |
| Rama | `012-vender-mejorado`, desde la rama del frontend con 010 y 011 (usa la barra lateral y los campos unificados) |
| Dependencias nuevas | Ninguna (`@vueuse/core` y Pinia ya están) |
| Cambios de API | Ninguno en el contrato. Solo datos de demostración en `DevelopmentSeeder` (v1.3, rama `012-vender-mejorado` de la API) |

## Chequeo de constitución

| Principio | Cumple | Cómo / justificación de la excepción |
|---|---|---|
| I. Spec primero | Sí | Spec 012 aprobada con A-56 a A-59 |
| II. Corrección fiscal | Sí | Los importes siguen siendo vista previa; al restaurar se refrescan precios del catálogo y el servidor calcula los definitivos |
| III. Contrato de API | Sí | Sin cambios; se usan rutas existentes y tipadas |
| IV. Seguridad | Sí | En el dispositivo solo hay datos de la venta (sin token ni secretos). Clave por empresa y usuario; se borra al cerrar sesión. El cliente elegido queda en el dispositivo de la empresa (anotado para A-24 ⚖️) |
| V. Simplicidad | Sí | Un store y tres componentes de la feature `sales`; sin librería de persistencia |
| VI. Verificable | Sí | Vitest del store y de la vista; capturas a 360 px y escritorio |
| VII. Personas reales | Sí | No se pierde la venta al vencer la sesión ni al recargar; venta por toques; barra fija en el celular |
| VIII. Aislamiento y auditoría | Sí | La venta guardada se separa por empresa y usuario; la API sigue limitando todo por empresa |

## Venta en curso (`features/sales/stores/sale-draft.ts`)

- Estado: líneas (`product` con los datos mostrados, `quantity`, `discount`), `kind`, `seriesId`, `paymentMethod`, cliente del ticket (nombre y documento opcionales), `documentCustomer` y `idempotencyKey`.
- Clave `sunat.sale-draft.<companyId>.<userId>`, formato versionado (`v: 1`). Se guarda con un `watch` profundo; se lee al abrir «Vender». Datos dañados o de otra versión: se descartan. Todo acceso a `localStorage` con try/catch (sin almacenamiento, la venta vive en memoria).
- `clear()` vacía el estado y borra la clave. Se llama al cobrar o emitir con éxito, al «Vaciar carrito» y al cerrar sesión desde `AppLayout` (la capa `app/` puede usar features; `core/` no conoce la venta). El 401 por sesión vencida **no** la borra (RF-003).
- `refresh()` al restaurar: `GET /products/{id}` por línea (`Promise.allSettled`); actualiza precio y disponible; quita los inexistentes o desactivados y devuelve sus nombres para avisar.
- La clave de idempotencia se guarda: un cobro reintentado tras volver a la pantalla no duplica la venta (spec 003, RF-008).

## Pantalla

- `NewSaleView`: orquesta. Arriba, tipo (Ticket / Boleta / Factura). Desde `lg`: cuadrícula `[1fr_22rem]`, con el catálogo a la izquierda y el carrito a la derecha (`sticky`, alto máximo de la ventana y desplazamiento propio).
- `components/ProductCatalog.vue`: `SearchInput` y tarjetas en cuadrícula (nombre, precio, disponible, «En el carrito: N»). Sin disponible: deshabilitada con el motivo. «Cargar más» con la paginación de `/products`.
- `components/SaleCart.vue`: líneas (pasos −/+, cantidad, descuento y quitar), cliente (ticket o `CustomerPicker`), serie, medio de pago, total, «Cobrar» o «Emitir» y «Vaciar carrito» (confirmación con `BaseDialog`).
- Celular: el carrito no se muestra en la columna; una barra fija abajo («N productos · S/ total · Ver carrito») abre `SaleCart` dentro de `BaseDialog`, que ya se muestra como hoja inferior en el celular. Escritorio o celular con `useMediaQuery`, como `AppLayout`.
- v1.1: la ruta `new-sale` lleva `meta.fullWidth`. Desde `lg`, la vista ocupa la altura disponible: el catálogo (con la búsqueda fija arriba) y el carrito se desplazan por separado; el carrito ya no necesita `sticky` ni alto máximo.
- v1.2: el selector Ticket / Boleta / Factura, el distintivo de pruebas y el motivo de bloqueo pasan a la cabecera de `SaleCart` (la vista le pasa las opciones con su estado y escucha `kind-changed`). La barra del celular muestra el tipo elegido.
- v1.3: orden dentro de `SaleCart`: cabecera (Carrito · Vaciar), líneas, qué emitir (con distintivo y motivo), serie y cliente, medio de pago, y el pie fijo con total y botón. Con el carrito vacío solo se ve la invitación a agregar productos.
- `salesApi`: `listProducts({ search, page })` (sustituye a `searchProducts`) y `getProduct(id)`.

## Estrategia de pruebas

| Requisito / escenario | Prueba | Tipo |
|---|---|---|
| RF-001, HU-1 esc. 1 y 4 | `sale-draft.spec.ts`: guarda y restaura por empresa y usuario; otro usuario no la ve | Unitaria |
| Casos límite | `sale-draft.spec.ts`: datos dañados, otra versión y sin almacenamiento | Unitaria |
| RF-002 | `sale-draft.spec.ts`: `refresh` actualiza precios y quita desactivados o inexistentes | Unitaria |
| HU-1 esc. 3 y 6 | `NewSaleView.spec.ts`: al cobrar con éxito se vacía; tras un error de red, la misma clave de idempotencia | Unitaria |
| CE-001 | `NewSaleView.spec.ts`: desmontar y volver a montar conserva el carrito | Unitaria |
| RF-003 | `AppLayout.spec.ts`: cerrar sesión borra la venta en curso; un 401 no | Unitaria |
| RF-004, HU-2 | `NewSaleView.spec.ts`: «Vaciar carrito» confirma; cancelar no cambia nada | Unitaria |
| RF-005, HU-3 | `NewSaleView.spec.ts`: catálogo sin búsqueda, filtro, «Cargar más» y sin disponible deshabilitado | Unitaria |
| RF-006, CE-002, CE-003 | Capturas a 360 px (con barra y con hoja abierta) y en escritorio, en claro y oscuro; venta de 3 productos solo con toques | Manual |

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Venta guardada con precios viejos | `refresh()` al restaurar; el servidor cobra el precio vigente |
| Datos del cliente en el dispositivo | Por usuario y empresa, se borran al cerrar sesión; anotado para A-24 ⚖️ |
| Cobro doble tras volver a la pantalla | La clave de idempotencia se guarda con la venta |
