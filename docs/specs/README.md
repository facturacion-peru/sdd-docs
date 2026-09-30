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
| 006 | [Panel de administración de la plataforma](006-panel-plataforma/spec.md) | **Implementada** (pendiente de merge) · [plan](006-panel-plataforma/plan.md) · [tareas](006-panel-plataforma/tasks.md) · [verificación](006-panel-plataforma/verificacion.md) | 001, 004, 005 | — |

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
