# Constitución del proyecto

**Versión:** 1.1.2 · **Ratificada:** 2026-09-23 · **Última enmienda:** 2026-09-26

Principios no negociables para la API de facturación electrónica SUNAT y su frontend. Toda especificación, plan y tarea debe cumplirlos. Si algo choca con esta constitución, se cambia el trabajo o se enmienda la constitución (ver [Gobierno](#gobierno)); nunca se ignora en silencio.

Las palabras **DEBE** / **NO DEBE** son obligatorias. **DEBERÍA** admite excepciones justificadas por escrito en el plan.

---

## Principios

### I. La especificación va primero

Ninguna funcionalidad se implementa sin una especificación aprobada.

- El flujo es: **especificar → planificar → dividir en tareas → implementar**, cada paso revisado por el usuario antes del siguiente.
- La especificación describe **qué** y **por qué** (usuarios, escenarios, criterios de aceptación). No menciona frameworks, tablas ni componentes.
- El plan describe **cómo** y DEBE verificar el cumplimiento de esta constitución.
- Las ambigüedades se marcan como `[NECESITA ACLARACIÓN: …]` y se resuelven con el usuario; no se inventan.
- Si la implementación revela que la especificación estaba mal, se corrige primero la especificación.

### II. Corrección fiscal ante todo

Un comprobante mal emitido tiene consecuencias legales y tributarias para el cliente.

- Los cálculos de impuestos (IGV, ISC, ICBPER, redondeos) y las reglas de SUNAT DEBEN vivir **solo en la API**. El frontend muestra resultados; no los recalcula como fuente de verdad.
- La generación de XML, firma, envío a SUNAT, PDF, QR y formatos de impresión (A4, 80 mm, 50 mm) pertenecen a la API.
- Todo cambio que afecte montos, series, correlativos o XML DEBE tener pruebas automatizadas con casos reales (incluidos redondeos y exonerados/inafectos).
- Se prueba contra el entorno **beta** de SUNAT. Nada se envía a producción desde desarrollo.

### III. El contrato de la API es la frontera

API y frontend se comunican únicamente a través de un contrato explícito.

- El contrato es el spec OpenAPI (`php artisan openapi:generate`). El frontend consume tipos generados (`pnpm api:types`) y NO DEBE editarlos a mano.
- Un cambio de contrato DEBE ir en la especificación, y actualizar spec y tipos en el mismo paso.
- Las respuestas siguen un sobre uniforme: `{ success, data, meta }`, donde `meta` es solo paginación y los datos extra van en claves propias.
- Las rutas protegidas viven bajo `/api/v1`. Un cambio incompatible requiere versión nueva o migración acordada.

### IV. Seguridad por defecto

- **Solo pnpm** en proyectos JavaScript. Las protecciones de `pnpm-workspace.yaml` NO DEBEN desactivarse; los paquetes con scripts de instalación se aprueban uno a uno tras revisarlos.
- Toda dependencia nueva se justifica en el plan (qué aporta, mantenimiento, alternativas).
- Ningún endpoint que modifique datos o estado del sistema puede ser público, salvo los **flujos de autenticación** (inicio de sesión, aceptación de invitación y recuperación de contraseña), que DEBEN usar tokens de un solo uso y de vigencia limitada, y tener límite de intentos. La inicialización del sistema se hace por consola, nunca por HTTP.
- Certificados digitales, contraseñas SOL y credenciales NUNCA van al repositorio, a logs ni al frontend.
- Se valida en el servidor toda entrada (`FormRequest`); la validación del frontend es solo ayuda al usuario.
- CORS restringido a orígenes conocidos fuera de desarrollo.

### V. Simplicidad y límites claros

- Se implementa lo que pide la especificación, nada más (YAGNI). Sin abstracciones "por si acaso".
- **API:** controladores delgados; la lógica de negocio va en servicios (`app/Services`); la validación, en `FormRequest`.
- **Frontend:** organización por features. Una feature no importa de otra; `core/` no conoce el dominio; `shared/ui` no llama a la API; cada feature declara sus rutas.
- Se prefiere lo que ya existe en el proyecto antes que añadir piezas nuevas.

### VI. Verificable antes de darse por terminado

- Cada criterio de aceptación de la especificación DEBE tener una prueba o una verificación manual documentada.
- **API:** `php artisan test` (Pest) en verde.
- **Frontend:** `pnpm type-check`, `pnpm lint` y `pnpm test:unit --run` en verde, y `pnpm build-only` compila.
- Una tarea no está completa si alguna de estas comprobaciones falla. Si algo se omitió, se dice explícitamente.

### VII. Pensado para personas y dispositivos reales

- El frontend lo usan **todos los usuarios de una empresa**, incluido su administrador; solo el panel de la plataforma vive aparte. Muchos emiten desde el celular y con prisa: mobile-first, flujos cortos y mensajes de error accionables en español.
- El mismo código DEBE servir para web y para el APK (Capacitor). Todo acceso a capacidades del dispositivo (almacenamiento seguro, impresión Bluetooth) pasa por un módulo aislado y sustituible.
- La sesión expira (token de 24 h): la interfaz DEBE manejar el re-login sin perder el trabajo en curso cuando sea razonable.
- Accesibilidad básica: HTML semántico, etiquetas en formularios y navegación por teclado.

### VIII. Aislamiento por empresa y auditoría

Varias empresas comparten la plataforma; una fuga de datos entre ellas es el peor fallo posible del producto.

- Todo dato de negocio DEBE pertenecer a una empresa.
- Toda lectura, escritura y listado DEBE limitarse a la empresa del usuario autenticado de forma **automática**. NO DEBE depender de que cada consulta recuerde filtrar ni de parámetros enviados por el cliente.
- El acceso a un recurso de otra empresa DEBE responderse igual que un recurso inexistente.
- Cada endpoint que exponga datos de negocio DEBE tener una prueba automatizada de acceso cruzado entre dos empresas.
- Las acciones sensibles (usuarios, roles, configuración, credenciales, series, comprobantes, inventario) DEBEN quedar en un registro de auditoría inmutable, sin valores de secretos.
- El administrador de la plataforma no pertenece a ninguna empresa, no opera ventas y nunca ve secretos de las empresas.

---

## Restricciones técnicas

| Ámbito | API | Frontend |
|---|---|---|
| Lenguaje / runtime | PHP 8.2+, Laravel 12 | TypeScript 5.9, Node ≥ 24.12 |
| Gestor de paquetes | Composer | **pnpm 11** (nunca npm/npx/yarn) |
| SUNAT | Greenter 5.1 | — |
| Autenticación | Sanctum, *personal access tokens* | Bearer token vía middleware de `openapi-fetch` |
| Pruebas | Pest | Vitest + Vue Test Utils |
| Estilos | Tailwind 4 (panel Blade) | Tailwind 4 |

Cambiar una de estas piezas requiere una enmienda.

## Flujo de trabajo SDD

La documentación vive en `docs/`:

```
docs/
├── constitution.md          # Este archivo
├── producto/vision.md       # Contexto de negocio y alcance del MVP (no es una spec)
├── aclaraciones.md          # Registro único de preguntas abiertas y decisiones
├── investigacion/           # Evaluaciones y estudios que alimentan los planes
├── plantillas/              # Plantillas de spec, plan y tareas
└── specs/
    ├── README.md            # Índice, estado y ciclo de vida
    └── NNN-nombre-corto/    # Una carpeta por funcionalidad, numerada
        ├── spec.md          # Qué y por qué
        ├── plan.md          # Cómo, y cumplimiento de la constitución
        └── tasks.md         # Tareas pequeñas, ordenadas y verificables
```

1. **Especificar:** el usuario describe la funcionalidad → se redacta `spec.md` desde la plantilla → las dudas van a `aclaraciones.md` y se resuelven → aprobación.
2. **Planificar:** `plan.md` con decisiones técnicas, cambios de contrato y un chequeo de cada principio → aprobación.
3. **Tareas:** `tasks.md` con pasos pequeños, cada uno con su verificación → aprobación.
4. **Implementar:** tarea por tarea, marcando el avance en `tasks.md`.
5. **Cerrar:** comprobaciones del principio VI en verde y entrada en la bitácora de `AGENTS.md`.

## Deuda conocida frente a esta constitución

Código heredado que aún no cumple. No se amplía; se corrige cuando una especificación lo toque o con una propia.

Detalle y evidencia en [`investigacion/001-evaluacion-api-existente.md`](investigacion/001-evaluacion-api-existente.md).

- **Principio IV / II:** el código heredado en `legacy/` (fuera del autoload, sin rutas) guarda la clave SOL y el certificado sin cifrar y asigna correlativos sin bloqueo. No se ejecuta; se rehace al reincorporarlo en las specs 004 y 005.
- **Principio II / VI:** no hay pruebas del cálculo de impuestos ni del XML; se escriben en la spec 005 antes de reincorporar la emisión.
- **Principio III:** el spec OpenAPI no documenta los *response bodies*; el sobre se tipa a mano en el frontend, y las pruebas de contrato de la API (`tests/Feature/Contract`) detectan cambios de forma.

**Resuelto por la spec 001 (2026-09-28):** endpoints `setup` públicos retirados; aislamiento entre empresas y roles aplicados y probados en cada ruta; CORS restringido a orígenes configurados; suite heredada retirada junto con el código que probaba.

## Gobierno

- Esta constitución prevalece sobre cualquier otra práctica o documento del proyecto.
- Solo el usuario aprueba enmiendas. Cada enmienda actualiza la versión y la fecha, y se registra abajo.
- Versionado semántico: **MAJOR** si se elimina o redefine un principio; **MINOR** si se añade un principio o sección; **PATCH** para aclaraciones y redacción.
- Cada `plan.md` DEBE incluir una sección "Chequeo de constitución"; las excepciones se justifican ahí.

### Historial de enmiendas

| Versión | Fecha | Cambio |
|---|---|---|
| 1.0.0 | 2026-09-23 | Versión inicial |
| 1.0.1 | 2026-09-26 | PATCH: estructura de `docs/` ampliada (visión, aclaraciones, investigación, plantillas) y deuda conocida actualizada tras la evaluación de la API |
| 1.1.0 | 2026-09-26 | MINOR (A-10): nuevo principio VIII, aislamiento por empresa y auditoría; el principio VII aclara que el frontend incluye al administrador de empresa |
| 1.1.1 | 2026-09-26 | PATCH (plan 001): el principio IV precisa las excepciones públicas como flujos de autenticación con token de un solo uso y límite de intentos; la inicialización pasa a consola |
| 1.1.2 | 2026-09-28 | PATCH (T123): deuda conocida actualizada tras implementar la spec 001 |
