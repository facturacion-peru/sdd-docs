# Plan 010: Identidad visual y tema

**Estado:** Aprobado (2026-10-03) · **Fecha:** 2026-10-03 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.4)

## Resumen técnico

Solo frontend. Los tokens de Tailwind 4 (`@theme` en `src/assets/main.css`) son el lugar central: escala de marca completa, neutros y tres escalas de estado (`success`, `warning`, `danger`). Las pantallas pasan a usar esos tokens. El logo es un componente SVG de `shared/ui` que pinta con clases `fill-brand-*`; el favicon es un SVG con el mismo dibujo. La tipografía Inter se instala como paquete y la empaqueta Vite.

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | Frontend (la API no cambia) |
| Rama | `010-identidad-visual`, creada desde `009-despliegue-operacion` (la 009 aún no está en `main` y el inicio usa su `PilotNotice`) |
| Dependencias nuevas | `@fontsource-variable/inter` (ver abajo) |
| Migraciones | Ninguna |

**`@fontsource-variable/inter`:** solo archivos de fuente y CSS, sin scripts de instalación ni dependencias. Se prefiere a Google Fonts porque no hace peticiones a terceros (privacidad, funciona dentro del APK) y a copiar los archivos a mano porque se actualiza con pnpm. Alternativa descartada: fuente del sistema (se ve distinta en cada dispositivo). Para instalarla hizo falta excluir `semver@6.3.1` de `trustPolicy` (falso positivo revisado, ver T011); la protección sigue activa para todo lo demás.

## Chequeo de constitución

| Principio | Cumple | Cómo / justificación de la excepción |
|---|---|---|
| I. Spec primero | Sí | Spec 010 aprobada con A-49 a A-51 |
| II. Corrección fiscal | N/A | No toca montos; solo la presentación (dígitos tabulares) |
| III. Contrato de API | N/A | Sin cambios de contrato |
| IV. Seguridad | Sí | Una dependencia justificada, sin scripts; protecciones de pnpm intactas; sin peticiones a terceros |
| V. Simplicidad | Sí | Se reutiliza `@theme`; el logo es un componente de `shared/ui` sin llamadas a la API |
| VI. Verificable | Sí | Prueba del logo, prueba de que no hay colores de estado sueltos y las cuatro comprobaciones del frontend |
| VII. Personas reales | Sí | Contraste AA, 360 px, logo accesible, funciona sin red externa (APK) |
| VIII. Aislamiento y auditoría | N/A | Sin datos de negocio |

## Tokens (`main.css`)

| Token | Uso |
|---|---|
| `brand-50…950` | Marca (tema «Petróleo», A-49) |
| `ink`, `ink-muted`, `line`, `surface`, `canvas`, `subtle` | Texto, bordes y fondos neutros |
| `success-*`, `warning-*`, `danger-*` | Estados (50, 100, 200, 300, 600, 700, 800, 900) |
| `--font-sans` | Inter con la del sistema de respaldo |

## Frontend

- `shared/ui/LogoApp.vue`: SVG con props `size` y `with-name`; `role="img"` con el nombre de la app, u oculto si el nombre ya está escrito.
- `public/favicon.svg`: mismo dibujo con los colores en hex (un favicon no lee el CSS). Reemplaza `favicon.ico`.
- Logo en `AuthLayout`, `AppLayout` y `PlatformLayout` (cabecera oscura con `brand-950`).
- `BaseAlert`, `BaseBadge`, `BaseButton`, `BaseInput`, `FormField`, `BaseDialog`, `EnvironmentBadge`, `PilotNotice` y las vistas pasan a los tokens de estado.
- Retoques de los componentes base: anillo de foco en campos, sombra suave en botones primarios, `tabular-nums` en montos.
- `TicketReceipt` queda en blanco y negro (impresión).

### Navegación lateral (HU-6, spec v1.3)

- `AppLayout`: desde `lg` (1024 px), `<aside>` fijo de 16 rem a toda la altura (`sticky`, `h-dvh`); el contenido a su derecha. Por debajo, barra superior de 56 px (botón de menú, logo y empresa) y el mismo `<aside>` como panel deslizable con velo.
- Panel: `aria-expanded` y `aria-controls` en el botón; `inert` mientras está cerrado en el celular (sus enlaces no reciben foco); se cierra con Escape, con el velo y al cambiar de ruta; el foco va al panel al abrir y vuelve al botón al cerrar; bloquea el desplazamiento del fondo (`useScrollLock`). Escritorio/celular con `useMediaQuery` de `@vueuse/core` (ya es dependencia).
- `MainNav` pasa a lista vertical con íconos: Inicio, Vender, Ventas, Productos y, solo para el administrador, el grupo «Gestión» (Alertas, Clientes, Usuarios, Auditoría, Mi empresa, SUNAT, Series). Desaparece el menú «Más»: en la barra lateral hay altura de sobra.
- Pie del panel en una sola fila: avatar, usuario y rol, `ThemeToggle` y «Cerrar sesión» como ícono con texto para lectores de pantalla (así la navegación del administrador cabe en 800 px de alto).
- El Inicio deja de mostrar los accesos «Gestión» (Usuarios, Mi empresa, Auditoría): ya están siempre visibles en la barra lateral.
- `PlatformLayout` no cambia en esta versión.
- v1.4: botón «Contraer menú» / «Expandir menú» (`aria-pressed`) al pie de la barra, solo desde `lg`; contraída mide 4.5 rem y muestra solo íconos (`MainNav` recibe `collapsed`: etiquetas `sr-only` y `title`). Preferencia en `localStorage` (`sunat.sidebar.collapsed`, con try/catch).
- v1.4: rutas con `meta.fullWidth` (hoy «Vender», spec 012) usan todo el ancho y, desde `lg`, todo el alto de la ventana (`h-dvh`), sin el contenedor de 5xl.

### Modo oscuro (HU-5, spec v1.2)

- `main.css`: bajo `:root[data-theme='dark']` se redefinen los mismos tokens (neutros, marca y estados) y `color-scheme: dark` para los controles nativos. Fondo claro: `canvas` pasa a `#eef2f6` (A-55).
- Papeles fijos de cada tono, válidos en claro y en oscuro: `50`–`200` fondos y bordes tenues; `300`–`400` gráficos y acentos; `600` relleno sólido con texto blanco; `700` texto de acento; `800` *hover* del relleno sólido; `900`–`950` superficies siempre oscuras. Por eso el *hover* del botón primario pasa de `700` a `800`, los textos de error de `danger-600` a `danger-700`, el velo de diálogos a `bg-black/60` y el tooltip del gráfico a `bg-ink text-surface`.
- `shared/composables/useTheme.ts`: preferencia `system | light | dark` en `localStorage` (`sunat.theme`, con try/catch). Resuelve «Sistema» con `matchMedia` y escucha sus cambios; siempre deja `data-theme="light|dark"` en `<html>`, así el CSS tiene un solo bloque oscuro. Se aplica en `main.ts` antes de montar, sin parpadeo.
- `shared/ui/ThemeToggle.vue`: botón que alterna Sistema → Claro → Oscuro, con ícono y `aria-label` que dice el tema actual. Va en `AppLayout`, `PlatformLayout` y `AuthLayout`.
- `TicketReceipt` conserva su fondo blanco (es una vista previa del papel).

### Campos (HU-4, spec v1.1)

- `shared/ui/field.ts`: la clase única de campo (`fieldClass`), usada por `BaseInput`, `BaseSelect` y `SearchInput`.
- `shared/ui/BaseSelect.vue`: `<select>` con `v-model`, atributos que pasan al elemento, `invalid` y una flecha propia (`appearance-none`). Sustituye las constantes `selectClass` repetidas en 11 vistas.
- `shared/ui/SearchInput.vue`: `type="search"` con lupa a la izquierda y botón «Limpiar búsqueda» (`aria-label`, 44 px de área táctil) a la derecha cuando hay texto; al pulsarlo vacía el modelo, emite `clear` y devuelve el foco. Se oculta la «x» nativa del navegador para no duplicarla. Las vistas escuchan `@clear` con el mismo manejador que ya usan para buscar.
- Búsquedas que lo usan: productos, clientes, empresas (plataforma), producto al vender, cliente al vender (`CustomerPicker`), ubigeo (`UbigeoPicker`) y el filtro de cliente de comprobantes.
- `main.css`: `accent-color` de marca para casillas y opciones, y botón de archivo con estilo del tema.
- Los `<input>` de texto o número sueltos (cantidades en «Vender», notas de crédito, SUNAT) pasan a `BaseInput` o a `fieldClass`.

## Estrategia de pruebas

| Requisito / escenario | Prueba | Tipo |
|---|---|---|
| RF-002, CE-002 | `shared/ui/__tests__/theme.spec.ts`: recorre `src/**/*.vue` y falla si encuentra `red-`, `green-`, `amber-`, `slate-` (salvo `TicketReceipt`) | Unitaria |
| RF-003, HU-2 esc. 2 | `LogoApp.spec.ts` | Unitaria |
| HU-5, RF-009 | `useTheme.spec.ts`: preferencia guardada, «Sistema» sigue a `matchMedia`, sin almacenamiento no falla | Unitaria |
| HU-6 esc. 5 | `AppLayout.spec.ts`: contraer y expandir, recordado al volver a montar; los enlaces conservan su nombre accesible | Unitaria |
| HU-6, RF-011 | `MainNav.spec.ts` (roles, grupo «Gestión», ruta activa) y `AppLayout.spec.ts` (abre y cierra el panel, Escape devuelve el foco, `inert`, cierra al navegar, cerrar sesión) | Unitaria |
| RF-010 | `ThemeToggle.spec.ts`: alterna y anuncia el tema | Unitaria |
| HU-5 esc. 3 | Capturas en oscuro del acceso, el inicio, productos, vender y plataforma | Manual |
| RF-008, HU-4 esc. 2–4 | `SearchInput.spec.ts`: lupa, botón solo con texto, limpiar emite `clear`, vacía y enfoca | Unitaria |
| RF-007 | `BaseSelect.spec.ts` (v-model e `invalid`) | Unitaria |
| CE-004 | `theme.spec.ts`: ninguna vista contiene `selectClass` ni un `<input>` de texto o `<select>` con clases propias | Unitaria |
| RF-004, CE-003 | Capturas a 360 px de acceso, inicio y plataforma | Manual |
| CE-001 | Cambiar la escala de marca y revisar | Manual |
| RF-006 | Contraste calculado de las combinaciones de marca y estado | Manual (documentado en `tasks.md`) |

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Que una pantalla vuelva a escribir colores sueltos | La prueba `theme.spec.ts` lo detecta |
| Favicon desincronizado del logo | Documentado en el comentario de `LogoApp.vue` |
