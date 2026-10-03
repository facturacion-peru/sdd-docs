# Tareas 010: Identidad visual y tema

**Estado:** Implementada (2026-10-03) · **Fecha:** 2026-10-03 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior.

## Fase 1 · Preparación

- [x] **T001** Rama `010-identidad-visual` en el frontend — Verificación: `git branch --show-current`

## Fase 2 · Tema (HU-1)

- [x] **T002** Escala de marca completa «Petróleo» en `@theme` — Verificación: contraste blanco/`brand-600` ≥ 4.5 y `brand-700`/blanco ≥ 4.5 (calculado: 5.1 y 7.1; `danger-600`/blanco 4.8)
- [x] **T003** Prueba `theme.spec.ts` que busca colores de estado sueltos (debe fallar) — Verificación: `pnpm test:unit --run theme` en rojo
- [x] **T004** Tokens `success`, `warning`, `danger` y `subtle`; componentes base y vistas pasan a usarlos — Verificación: T003 en verde
- [x] **T005** Retoques de componentes base (foco de campos, botones, `tabular-nums`) — Verificación: suites en verde

## Fase 3 · Logo (HU-2)

- [x] **T006** `LogoApp.vue` y su prueba — Verificación: `pnpm test:unit --run LogoApp`
- [x] **T007** Logo en `AuthLayout` (pantallas de `features/auth`) — Verificación: captura a 360 px
- [x] **T008** [P] Logo en la cabecera de `AppLayout` — Verificación: captura a 360 px
- [x] **T009** [P] Logo en `PlatformLayout` con cabecera `brand-950` — Verificación: captura a 360 px
- [x] **T010** `public/favicon.svg` y enlace en `index.html`; se retira `favicon.ico` — Verificación: `pnpm build-only` y pestaña del navegador

## Fase 4 · Tipografía (HU-3)

- [x] **T011** `pnpm add @fontsource-variable/inter` y `--font-sans` — Verificación: `pnpm build-only` incluye el `.woff2`; ninguna petición a terceros
  - `pnpm add` fallaba con `ERR_PNPM_TRUST_DOWNGRADE` en `semver@6.3.1` (subdependencia de `vite-plugin-vue-devtools`), no en la fuente. Revisado (backport oficial de npm, publicado después de 7.5.x sin provenance; mismo integrity que el lockfile) y, con aprobación del usuario, excluido solo él en `trustPolicyExclude`. `trustPolicy` sigue activo.

## Fase 5 · Campos (HU-4, spec v1.1)

- [x] **T015** Pruebas de `SearchInput` y `BaseSelect`, y regla de CE-004 en `theme.spec.ts` (deben fallar) — Verificación: en rojo
- [x] **T016** `field.ts`, `BaseSelect.vue` y `SearchInput.vue` — Verificación: pruebas de T015 de componentes en verde
- [x] **T017** Las vistas pasan a `BaseSelect` (17 archivos; `UsersView` con `:model-value`); los `<input>` sueltos a `BaseInput` (prop `narrow` para cantidades e importes) — Verificación: CE-004 en verde
- [x] **T018** Las búsquedas pasan a `SearchInput` con `@clear` — Verificación: pruebas de cada vista en verde
- [x] **T019** `accent-color` y botón de archivo en `main.css` — Verificación: captura
- [x] **T020** Capturas a 360 px de productos (con texto y botón de limpiar), comprobantes e inicio — Verificación: revisión manual. Corrigió el filtro «Cliente» de comprobantes, demasiado angosto en el celular

## Fase 6 · Modo oscuro (HU-5, spec v1.2)

- [x] **T021** Pruebas de `useTheme` y `ThemeToggle` (deben fallar) — Verificación: en rojo
- [x] **T022** `useTheme.ts`, `ThemeToggle.vue` y su uso en las tres cabeceras; aplicar en `main.ts` — Verificación: T021 en verde
- [x] **T023** Fondo claro `#eef2f6` y tokens oscuros en `main.css`; ajustes de papeles (`hover` 800, `danger-700`, velo y tooltip) — Verificación: suites en verde
- [x] **T024** Capturas en claro y oscuro a 360 px y en escritorio (acceso, inicio, vender, productos, plataforma) — Verificación: revisión manual. Corrigió: el nombre del usuario desaparecía en la cabecera a 360 px (en el celular «Cerrar sesión» pasa a ícono con texto para lectores de pantalla), y la tabla `sr-only` del gráfico desbordaba 20 px y dejaba ver su caption bajo el título (ahora `sr-only` va en un `div`)

## Fase 7 · Navegación lateral (HU-6, spec v1.3)

- [x] **T025** Pruebas de `MainNav` vertical y de `AppLayout` (panel, foco, `inert`, roles) (deben fallar) — Verificación: en rojo
- [x] **T026** `MainNav` vertical con íconos y grupo «Gestión» — Verificación: pruebas de `MainNav` en verde
- [x] **T027** `AppLayout` con barra lateral y panel en el celular — Verificación: pruebas de `AppLayout` en verde
- [x] **T028** Capturas en claro y oscuro a 360 px (cerrado y abierto) y en escritorio — Verificación: revisión manual. Corrigió: `:inert="false"` dejaba el atributo en el DOM (el panel quedaba inerte también abierto; ahora `|| undefined`) y el pie del panel no dejaba ver «Series» en 800 px (pie en una fila)

## Cierre

- [x] **T012** CE-001: cambiar la escala de marca y comprobar que cambia toda la app (luego revertir) — hecho con `brand-600` violeta: logo, navegación y enlaces cambiaron sin tocar pantallas
- [x] **T013** Comprobaciones del principio VI en verde (195 pruebas al cerrar la v1.3)
- [x] **T014** Entrada en la bitácora de `AGENTS.md` y estado en `docs/specs/README.md`
