# Especificaciones

Una carpeta por capacidad, numerada. Cada una avanza por el ciclo SDD de la [constitución](../constitution.md#flujo-de-trabajo-sdd) y **no pasa a la siguiente fase sin aprobación del usuario**.

## Estado

| # | Spec | Estado | Depende de | Aclaraciones abiertas |
|---|---|---|---|---|
| 001 | [Empresa, usuarios, roles y aislamiento](001-empresa-usuarios-aislamiento/spec.md) | **Integrada en `main`** (falta la revisión visual de T122) · [plan](001-empresa-usuarios-aislamiento/plan.md) · [tareas](001-empresa-usuarios-aislamiento/tasks.md) · [verificación](001-empresa-usuarios-aislamiento/verificacion.md) | — | — (A-11 ⚖️ a confirmar antes de producción) |
| 002 | [Productos, lotes e inventario](002-productos-inventario/spec.md) | **Integrada en `main`** · [plan](002-productos-inventario/plan.md) · [tareas](002-productos-inventario/tasks.md) · [verificación](002-productos-inventario/verificacion.md) | 001 | — |
| 003 | [Tickets de venta internos](003-tickets-venta/spec.md) | **Integrada en `main`** · [plan](003-tickets-venta/plan.md) · [tareas](003-tickets-venta/tasks.md) · [verificación](003-tickets-venta/verificacion.md) | 001, 002 | — (A-17 ⚖️ a confirmar antes de producción) |
| 004 | [Configuración segura de SUNAT y series](004-configuracion-sunat/spec.md) | **Integrada en `main`** · [plan](004-configuracion-sunat/plan.md) · [tareas](004-configuracion-sunat/tasks.md) · [verificación](004-configuracion-sunat/verificacion.md) | 001 | — |
| 005 | [Emisión y consulta de comprobantes en beta](005-emision-comprobantes-beta/spec.md) | **Integrada en `main`** · [plan](005-emision-comprobantes-beta/plan.md) · [tareas](005-emision-comprobantes-beta/tasks.md) · [verificación](005-emision-comprobantes-beta/verificacion.md) | 001, 002, 003, 004 | — (A-22 ⚖️ a confirmar antes de producción) |
| 006 | [Panel de administración de la plataforma](006-panel-plataforma/spec.md) | **Integrada en `main`** · [plan](006-panel-plataforma/plan.md) · [tareas](006-panel-plataforma/tasks.md) · [verificación](006-panel-plataforma/verificacion.md) | 001, 004, 005 | — |
| 007 | [Anulación y corrección de comprobantes](007-notas-credito/spec.md) | **Integrada en `main`** · [plan](007-notas-credito/plan.md) · [tareas](007-notas-credito/tasks.md) · [verificación](007-notas-credito/verificacion.md) | 002, 004, 005 | — (A-39 ⚖️ a confirmar antes de producción) |
| 008 | Paso a producción SUNAT | Por especificar (espera la revisión legal A-24 ⚖️) | 005, 007, 009 | A-24 ⚖️ |
| 009 | [Despliegue y operación](009-despliegue-operacion/spec.md) | **Lista** (T001–T023 integradas en `main`; despliegue al servidor en espera) · [plan](009-despliegue-operacion/plan.md) · [tareas](009-despliegue-operacion/tasks.md) | 001–007 | — |
| 010 | [Identidad visual y tema](010-identidad-visual/spec.md) | **Integrada en `main`** (v1.4) · [plan](010-identidad-visual/plan.md) · [tareas](010-identidad-visual/tasks.md) | 001–007 | — |
| 011 | [Panel de inicio](011-panel-inicio/spec.md) | **Integrada en `main`** · [plan](011-panel-inicio/plan.md) · [tareas](011-panel-inicio/tasks.md) | 003, 005, 007, 010 | — |
| 012 | [Vender: carrito persistente y dos columnas](012-vender-mejorado/spec.md) | **Integrada en `main`** (v1.5) · [plan](012-vender-mejorado/plan.md) · [tareas](012-vender-mejorado/tasks.md) | 003, 005, 010 | — |

Orden de trabajo: **001 → 002 → 003 → 004 → 005**. 003 y 004 son independientes entre sí; 004 puede adelantarse si se prioriza SUNAT.

## Ciclo de vida

| Estado | Significa | Para avanzar |
|---|---|---|
| **Borrador** | Redactándose | Historias, requisitos y criterios completos |
| **En aclaración** | Tiene preguntas abiertas en [`aclaraciones.md`](../aclaraciones.md) | Todas sus aclaraciones resueltas y la spec actualizada |
| **Aprobada** | El usuario aprobó el *qué* | Redactar `plan.md` |
| **Planificada** | El usuario aprobó el *cómo* | Redactar `tasks.md` |
| **Lista** | El usuario aprobó las tareas | Empezar a implementar |
| **En implementación** | Tareas en curso | Todas las tareas y las comprobaciones del principio VI |
| **Completada** | En verde y registrada en la bitácora de `AGENTS.md` | — |

Solo se escribe código de producción para specs en estado **Lista** o posterior.

## Estructura de cada carpeta

```
NNN-nombre-corto/
├── spec.md    # Qué y por qué     ← plantilla: docs/plantillas/spec.md
├── plan.md    # Cómo              ← se crea al aprobar la spec
└── tasks.md   # Pasos verificables ← se crea al aprobar el plan
```

## Reglas

- La spec no menciona frameworks, tablas, endpoints ni componentes; eso va en el plan.
- Las preguntas se registran en `aclaraciones.md` y se citan por su identificador; no se duplican entre specs.
- Si la implementación revela un error en la spec, se corrige la spec primero y se sube su versión.
- Cada criterio de aceptación termina teniendo una prueba o una verificación documentada.
