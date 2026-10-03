# Spec 010: Identidad visual y tema

**Estado:** Lista (2026-10-03) · **Versión:** 1.3 · **Fecha:** 2026-10-03
**Depende de:** 001 a 007 (pantallas existentes) · **Aclaraciones:** A-49, A-50, A-51, A-55

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que la aplicación se vea **profesional y coherente** en todas sus pantallas, con una identidad visual (colores, tipografía y logo) definida **en un solo lugar** y fácil de cambiar. El proyecto es público y puede reutilizarse: quien lo adopte debe poder poner su marca sin tocar cada pantalla.

Hoy los colores de marca ya están centralizados, pero los de estado (éxito, aviso, error) se escriben sueltos en cada pantalla, no hay logo, la tipografía es la del sistema y el ícono del navegador es el genérico de la plantilla.

Esta spec pone la **base**. Las mejoras de cada pantalla van en specs propias (la primera, el panel de inicio en la 011).

## Historias de usuario

### HU-1 · Tema centralizado (P1)

Como responsable del producto (o quien reutilice el proyecto), quiero cambiar los colores de la aplicación en un único lugar, para adaptar la marca sin revisar pantalla por pantalla.

**Por qué esta prioridad:** sin esto, cada mejora visual posterior multiplica el trabajo de cambiar la marca.
**Prueba independiente:** cambiar la escala de marca en el lugar central y ver que botones, enlaces, navegación y logo cambian; no quedan colores de estado escritos sueltos fuera de ese lugar (salvo la impresión del ticket).

**Escenarios de aceptación**

1. **Dado** el tema «Petróleo» (A-49), **cuando** se abre cualquier pantalla, **entonces** los elementos de marca usan esa paleta con contraste AA en textos y botones.
2. **Dado** que se cambia la escala de marca en el lugar central, **cuando** se recarga, **entonces** cambian todos los elementos de marca, incluido el logo.
3. **Dado** un mensaje de éxito, aviso o error, **cuando** se muestra, **entonces** usa los colores de estado del tema, definidos también en el lugar central.

### HU-2 · Logo de la aplicación (P1)

Como usuario, quiero reconocer la aplicación por su logo en las pantallas de acceso, en la cabecera y en la pestaña del navegador.

**Prueba independiente:** el logo aparece en las pantallas de acceso, en la cabecera de la empresa, en la del panel de plataforma y como ícono de la pestaña.

**Escenarios de aceptación**

1. **Dado** un logo provisional genérico (A-51), **cuando** se reemplaza su dibujo en un solo lugar, **entonces** cambia en todas las pantallas de la aplicación.
2. **Dado** un lector de pantalla, **cuando** llega al logo, **entonces** lo anuncia una sola vez con el nombre de la aplicación.

### HU-3 · Tipografía legible (P2)

Como usuario que trabaja desde el celular, quiero una tipografía clara y uniforme en cualquier dispositivo, para leer montos y datos sin esfuerzo.

**Escenarios de aceptación**

1. **Dado** cualquier dispositivo, **cuando** se abre la aplicación, **entonces** el texto usa la misma tipografía (A-50), servida por la propia aplicación, sin pedirla a terceros.
2. **Dado** una columna de montos, **cuando** se muestra, **entonces** las cifras quedan alineadas (dígitos de ancho fijo).

### HU-4 · Campos de formulario uniformes y búsqueda clara (P1)

Como usuario, quiero que todos los campos se vean y respondan igual, y que las búsquedas tengan un ícono de lupa y un botón para limpiarlas, para reconocerlas al instante y volver a la lista completa con un toque.

**Prueba independiente:** en cualquier pantalla, campos de texto, listas desplegables, casillas y opciones comparten estilo y foco; en cada búsqueda, el botón de limpiar vacía el texto y la lista vuelve a mostrar todo.

**Escenarios de aceptación**

1. **Dado** un formulario o un filtro, **cuando** paso por sus campos con el teclado, **entonces** todos (texto, número, lista desplegable) tienen la misma altura, borde y anillo de foco.
2. **Dado** una búsqueda, **cuando** la veo, **entonces** tiene una lupa al inicio.
3. **Dado** una búsqueda con texto, **cuando** pulso «Limpiar búsqueda», **entonces** el texto se vacía, el filtro se quita, la lista se recarga y el foco vuelve a la búsqueda.
4. **Dado** una búsqueda vacía, **cuando** la veo, **entonces** no se muestra el botón de limpiar.

### HU-5 · Modo claro y oscuro (P2)

Como usuario, quiero elegir entre un tema claro y uno oscuro, o que siga al de mi dispositivo, para trabajar cómodo de día y de noche (A-55).

**Prueba independiente:** con el selector en «Oscuro», todas las pantallas se ven oscuras y legibles; con «Sistema», siguen al dispositivo; la elección se recuerda al volver.

**Escenarios de aceptación**

1. **Dado** un dispositivo en modo oscuro y la opción «Sistema», **cuando** abro la aplicación, **entonces** se ve en oscuro.
2. **Dado** el selector, **cuando** elijo «Claro» u «Oscuro», **entonces** el tema cambia al instante y se recuerda en ese dispositivo.
3. **Dado** el tema oscuro, **cuando** recorro las pantallas, **entonces** textos, botones, alertas y estados mantienen contraste AA.
4. **Dado** el modo claro, **cuando** veo cualquier pantalla, **entonces** el fondo es gris pizarra suave y las tarjetas resaltan sobre él.

### HU-6 · Navegación lateral (P2)

Como usuario, quiero el menú, el logo, el tema y «Cerrar sesión» en una barra lateral, para que el contenido aproveche toda la altura de la pantalla.

**Prueba independiente:** en escritorio la barra lateral está siempre visible con toda la navegación; en el celular, una barra superior mínima abre la misma navegación como panel.

**Escenarios de aceptación**

1. **Dado** una pantalla ancha, **cuando** uso la aplicación, **entonces** la barra lateral muestra logo, empresa, navegación completa, usuario, tema y «Cerrar sesión», y el contenido ocupa toda la altura.
2. **Dado** el celular, **cuando** pulso el botón de menú, **entonces** se abre el panel con la misma navegación; se cierra con Escape, tocando fuera o al navegar, y el foco vuelve al botón.
3. **Dado** un vendedor, **cuando** abre la navegación, **entonces** ve solo las entradas del día a día; el administrador ve además la gestión de la empresa.
4. **Dado** el panel cerrado en el celular, **cuando** navego con el teclado, **entonces** sus enlaces no reciben el foco.

## Casos límite

- **Sin conexión a terceros:** la tipografía no depende de servicios externos; si no carga, se usa la del sistema sin romper el diseño.
- **Impresión del ticket (80 mm):** sigue en blanco y negro, ajena al tema (también en oscuro).
- **Sin almacenamiento disponible** (navegación privada): el tema sigue al sistema y la elección no se recuerda, sin errores.
- **Panel de plataforma:** conserva una cabecera distinta a la de las empresas para que el administrador sepa siempre dónde está.

## Requisitos funcionales

- **RF-001:** Los colores de marca, neutros y de estado DEBEN definirse en un único lugar.
- **RF-002:** Las pantallas NO DEBEN usar colores de estado escritos sueltos (excepción: la impresión del ticket).
- **RF-003:** El logo DEBE definirse en un único lugar y tomar los colores del tema.
- **RF-004:** El logo DEBE mostrarse en las pantallas de acceso, en la cabecera de la aplicación, en la del panel de plataforma y como ícono de la pestaña.
- **RF-005:** La tipografía DEBE servirse desde la propia aplicación.
- **RF-006:** Los textos y botones DEBEN mantener contraste AA.
- **RF-007:** Los campos de texto, número y listas desplegables DEBEN compartir un único estilo definido en un solo lugar; casillas y opciones DEBEN usar el color de marca.
- **RF-008:** Toda búsqueda DEBE mostrar una lupa y, con texto, un botón accesible para limpiarla que quite el filtro.
- **RF-009:** El tema oscuro DEBE definirse en el mismo lugar central que el claro, como otros valores de los mismos tokens.
- **RF-010:** El selector de tema DEBE estar en la navegación de la aplicación, en la cabecera de la plataforma y en las pantallas de acceso, y ser accesible por teclado.
- **RF-011:** La navegación de la aplicación de empresa DEBE estar en una barra lateral en pantallas anchas y en un panel desplegable en el celular, con las mismas entradas y la misma regla de roles.

## Criterios de éxito

- **CE-001:** Cambiar la escala de marca en el lugar central cambia la marca de toda la aplicación sin tocar otras pantallas (verificación manual documentada).
- **CE-002:** Una búsqueda de colores de estado sueltos en las pantallas no encuentra ninguno fuera de la impresión del ticket.
- **CE-003:** Las pantallas de acceso, el inicio y el panel de plataforma se ven correctamente a 360 px.
- **CE-004:** Ninguna pantalla define su propio estilo de campo de texto o lista desplegable (prueba automatizada).

## Fuera de alcance

- Rediseño pantalla por pantalla (specs propias; la primera es la 011).
- El logo definitivo de la marca (A-51).

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-03 | A-49 Tema | «Petróleo»: verde azulado sobrio con neutros pizarra |
| 2026-10-03 | A-50 Tipografía | Inter, servida por la aplicación |
| 2026-10-03 | A-51 Logo | Provisional y genérico, reemplazable en un solo lugar |
| 2026-10-03 | Pedido del usuario (v1.3) | Barra lateral en escritorio y panel desplegable en el celular (HU-6) |
| 2026-10-03 | A-55 Modo oscuro (v1.2) | Claro y oscuro con selector; fondo claro `#eef2f6` |
| 2026-10-03 | Pedido del usuario (v1.1) | Unificar campos y búsqueda con lupa y botón de limpiar (HU-4) |
