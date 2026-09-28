# Evaluación de la API existente

**Fecha:** 2026-09-26 · **Estado:** terminada · **Decisión:** opción B, reiniciar el dominio ([A-01](../aclaraciones.md#a-01))
**Proyecto evaluado:** `Api-de-facturacion-electronica-sunat-Peru` (commit `3adec68` + cambios sin commitear)

Responde a la pregunta de la [visión](../producto/vision.md#2-punto-de-partida-técnico): ¿la API actual sirve como base del SaaS? Se evaluó contra los seis criterios de viabilidad.

## Resumen

**No cumple ninguno de los seis criterios tal como está.** Lo que sí tiene valor es la integración con SUNAT (Greenter), las plantillas de PDF/QR y los datos de ubigeo. El modelo de usuarios, empresas y permisos habría que rehacerlo en cualquier caso, y el inventario y los tickets no existen.

**Recomendación:** conservar el repositorio y el framework, **reiniciar el dominio** (autenticación, empresas, roles, aislamiento) con pruebas desde el primer commit, y **reincorporar** la emisión SUNAT existente solo cuando se cubra con pruebas en la spec 005. Detalle en [Opciones](#opciones).

## Resultado por criterio

| # | Criterio | Resultado | Evidencia |
|---|---|---|---|
| 1 | Separación confiable por empresa | ❌ No existe | Ver hallazgo H-1 |
| 2 | Autorización por usuario y rol | ❌ Modelado, no aplicado | Ver H-2 |
| 3 | Manejo seguro de credenciales | ❌ Texto plano, se filtran a logs | Ver H-3 |
| 4 | Emisión fiscal cubierta por pruebas | ❌ Sin pruebas; la suite falla | Ver H-4 |
| 5 | Contrato OpenAPI mantenible | ⚠️ Parcial | Ver H-5 |
| 6 | Evolucionar sin romper el frontend | ⚠️ Solo por ausencia de consumidores | El frontend aún no consume endpoints de negocio |

## Hallazgos

### H-1 · Sin aislamiento entre empresas (crítico)

- `CompanyController@index` devuelve **todas** las empresas a cualquier usuario autenticado; `show`, `update` y `destroy` operan sobre cualquier `{company}` sin comprobar pertenencia.
- `InvoiceController@index` solo filtra por empresa **si el cliente envía** `company_id`; sin él, lista las facturas de todas las empresas. Lo mismo en `ClientController`.
- `User::canAccessCompany()` existe pero **no se usa en ningún controlador**. Solo `SetupController` compara `company_id` en un punto.
- Los 23 `FormRequest` devuelven `authorize(): true` sin condiciones.

### H-2 · Roles y permisos sin aplicar

- Hay tablas `roles`, `permissions`, `role_permission` y métodos `hasPermission()`/`hasRole()` en `User`, pero ninguna ruta ni controlador los invoca. Todo usuario autenticado puede hacer todo.
- `POST /api/v1/auth/create-user` acepta `user_type` (`system`, `user`, `api_client`) desde la petición.
- `POST /api/setup/migrate` y `POST /api/setup/seed` son **públicos** y ejecutan `migrate --force` y `db:seed`.

### H-3 · Secretos sin protección (crítico)

- `clave_sol`, `certificado_password`, `gre_clave_sol` y los `client_secret` se guardan **en texto plano** (sin cast `encrypted`). `$hidden` evita que salgan en JSON, pero no protege la base de datos.
- `CompanyController@update` registra `'changes' => $company->getChanges()` en el log: **al cambiar la clave SOL, queda escrita en `storage/logs`**.
- `SetupController@configureSunat` guarda el certificado en el disco `public` con un nombre fijo (`certificado/certificado.pem`). Con `storage:link` quedaría **descargable por URL**, y cada empresa **sobrescribe** el certificado de la anterior.

### H-4 · Emisión fiscal sin red de seguridad

- `php artisan test`: **35 fallidos, 1 aprobado**. Las pruebas usan columnas eliminadas (`configuraciones`) y cubren notas de crédito, débito y guías que **ya no tienen rutas**.
- `DocumentService` (2 263 líneas) importa `CreditNote`, `DebitNote`, `DailySummary`, `DispatchGuide`, `Retention` y `VoidedDocument`, modelos que **no existen** en `app/Models`. Esas ramas fallan en tiempo de ejecución.
- Ninguna prueba cubre cálculos de IGV, redondeos ni la construcción del XML.
- La asignación de correlativos va dentro de `DB::transaction` pero **sin `lockForUpdate`**: dos emisiones simultáneas pueden obtener el mismo número.

### H-5 · Contrato

- Existe `php artisan openapi:generate` y el frontend ya genera tipos desde él (78 endpoints). Documenta *requests* pero no *responses*.
- Rutas mezcladas (`/api/setup/*` pública y `/api/v1/setup/*` protegida) y convenciones inconsistentes (`/{id}` frente a *route model binding*).

### Qué sí vale la pena conservar

| Pieza | Por qué |
|---|---|
| `GreenterService`, `PdfService`, `PdfTemplateService` | Construcción de XML con Greenter, firma, envío, plantillas A4/80 mm/50 mm y QR. Es el conocimiento SUNAT más costoso de rehacer. |
| Seeders de ubigeo (`UbiRegion/Provincia/Distrito`) | Datos oficiales necesarios para direcciones fiscales. |
| Reglas de validación de `StoreInvoiceRequest` / `StoreBoletaRequest` | Referencia de los campos que exige SUNAT. |
| Laravel 12 + Sanctum + Pest | El stack de la constitución; no hay motivo para cambiarlo. |

## Opciones

| Opción | Descripción | Riesgo | Esfuerzo |
|---|---|---|---|
| **A. Adaptar** | Corregir controlador por controlador: añadir aislamiento, cifrado, políticas y pruebas al código actual. | Alto: cualquier consulta olvidada filtra datos de otra empresa; se arrastra código muerto. | Alto e incierto |
| **B. Reiniciar el dominio** *(recomendada)* | Mismo repositorio y stack. Se retiran los módulos actuales de empresas, usuarios y permisos, y se construyen de nuevo con aislamiento y pruebas desde el inicio. La emisión SUNAT se reincorpora en la spec 005, envuelta en pruebas. | Medio: requiere disciplina para no copiar los defectos al reincorporar código. | Medio y predecible |
| **C. Proyecto nuevo** | Laravel limpio en otro repositorio; se copian solo las piezas útiles. | Bajo técnicamente; se pierde el historial git y la relación con el fork. | Medio |

B y C son técnicamente equivalentes. B se recomienda porque conserva el historial, el remoto y las piezas reutilizables en su sitio. La decisión depende también de si hay **datos reales** que preservar ([A-02](../aclaraciones.md#a-02)).

## Consecuencias para las specs

- **001** se planifica como construcción nueva, no como adaptación de `Company`/`User` actuales.
- **004** debe definir el almacenamiento cifrado y privado de secretos y certificados; no reutiliza el modelo actual.
- **005** empieza por escribir pruebas del cálculo y del XML con el código existente antes de moverlo (*characterization tests*).
- La deuda de la constitución (endpoints `setup` públicos, CORS) se resuelve en el plan de 001.
