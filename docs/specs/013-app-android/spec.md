# Spec 013: Aplicación Android

**Estado:** Aprobada (2026-10-06) · **Versión:** 1.0 · **Fecha:** 2026-10-06
**Depende de:** 001–007, 010–012 (la aplicación de empresa ya integrada); 009 para el piloto real · **Aclaraciones:** A-61 a A-67

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que los usuarios de una empresa (administrador y vendedores) usen la aplicación desde una app instalada en su celular Android, con su ícono, a pantalla completa y con la sesión guardada de forma segura, sin pasar por el navegador. Es la misma aplicación web de hoy, no otra: lo que se mejore en una llega a la otra (constitución, principio VII).

Muchos negocios del piloto venden desde el celular: una app instalada abre más rápido, no se pierde entre pestañas y permite usar capacidades del teléfono que el navegador limita (compartir el PDF a la impresora, almacenamiento seguro).

## Historias de usuario

### HU-1 · Instalar y entrar (P1)

Como vendedor o administrador de una empresa, quiero instalar la app en mi Android y entrar con mi cuenta de siempre, para vender sin abrir el navegador (A-61).

**Por qué esta prioridad:** sin instalación y acceso no hay nada más.
**Prueba independiente:** instalar la app en un Android de gama baja, entrar con una cuenta de la empresa demo y ver el Inicio.

**Escenarios de aceptación**

1. **Dado** el archivo o el enlace de instalación, **cuando** instalo la app, **entonces** aparece con el nombre y el ícono del producto y abre a pantalla completa, sin barra de direcciones.
2. **Dado** la app instalada, **cuando** entro con mi correo y contraseña, **entonces** veo las mismas pantallas que en la web según mi rol (Inicio, Vender, Ventas, Productos, y las de administración si soy administrador).
3. **Dado** que cierro la app y la vuelvo a abrir, **cuando** mi sesión sigue vigente, **entonces** entro directo, sin volver a escribir la contraseña.
4. **Dado** una cuenta del administrador de la plataforma, **cuando** intento entrar en la app, **entonces** se me indica que el panel de la plataforma se usa desde la web (A-65).

### HU-2 · Sesión segura en el teléfono (P1)

Como administrador de la empresa, quiero que la sesión quede guardada de forma segura en el teléfono, para que otra app o alguien con el teléfono no pueda copiarla (A-64).

**Escenarios de aceptación**

1. **Dado** una sesión iniciada, **cuando** se guarda, **entonces** queda en el almacenamiento protegido del sistema y no en un lugar legible por otras apps ni en las copias de seguridad del teléfono.
2. **Dado** que cierro sesión, **cuando** vuelvo a abrir la app, **entonces** pide la contraseña y no queda nada de la sesión anterior.
3. **Dado** que la sesión vence a mitad de una venta, **cuando** vuelvo a entrar, **entonces** la venta en curso sigue ahí, como en la web (spec 012).

### HU-3 · Vender, imprimir y compartir desde la app (P1)

Como vendedor, quiero cobrar y entregar el ticket o comprobante desde la app, para atender al cliente sin la computadora (A-63).

**Escenarios de aceptación**

1. **Dado** una venta cobrada o un comprobante emitido, **cuando** pulso «Imprimir», **entonces** se abre el PDF de 80 mm del documento para mandarlo a la impresora o a otra app del teléfono (impresión del sistema o app de la impresora).
2. **Dado** el detalle de un comprobante, **cuando** pulso descargar o compartir (PDF A4 o 80 mm, XML, CDR), **entonces** puedo guardarlo o enviarlo por WhatsApp, correo u otra app del teléfono.
3. **Dado** cualquier pantalla de la app, **cuando** uso el botón «Atrás» de Android, **entonces** cierra el diálogo o la hoja abierta o vuelve a la pantalla anterior; en el Inicio, sale de la app sin perder la venta en curso.

### HU-4 · Sin conexión (P2)

Como vendedor en un lugar con mala señal, quiero saber claramente cuándo no hay conexión, para no creer que una venta se registró cuando no fue así (A-66).

**Escenarios de aceptación**

1. **Dado** que el teléfono no tiene conexión, **cuando** abro la app o intento cobrar, **entonces** veo un aviso claro de que no hay conexión y la venta en curso se conserva.
2. **Dado** que vuelve la conexión, **cuando** reintento el cobro, **entonces** no se duplica la venta (spec 003, RF-008).

### HU-5 · Versiones al día (P2)

Como responsable del SaaS, quiero que los usuarios tengan una versión de la app compatible con la API, para que un cambio en el servidor no deje la app rota en los celulares (A-67).

**Escenarios de aceptación**

1. **Dado** una versión de la app que ya no es compatible con la API, **cuando** se abre, **entonces** avisa que hay que actualizarla y explica cómo, en lugar de fallar con errores sueltos.
2. **Dado** una versión nueva publicada, **cuando** el usuario la instala encima, **entonces** conserva su sesión y su venta en curso.

## Casos límite

- **Pantallas pequeñas y gama baja** (360 px, 2–3 GB de RAM): la app debe abrir y vender con fluidez; es el mismo diseño que ya se revisa a 360 px.
- **Teclado en pantalla:** al escribir cantidades o buscar clientes, el teclado no tapa el campo ni el botón de cobrar.
- **Muesca y barras del sistema:** el contenido no queda debajo de la cámara, la barra de estado ni la barra de gestos.
- **Modo oscuro del teléfono:** la app respeta la elección hecha en la app; si nunca se eligió, sigue la del sistema (spec 010).
- **Servidor no disponible o sin HTTPS:** la app no se conecta a servidores sin cifrar en sus versiones para usuarios (A-62).
- **App en segundo plano mucho tiempo:** al volver sigue en la misma pantalla o, si Android la cerró, la venta en curso se recupera.

## Requisitos funcionales

- **RF-001:** La app Android DEBE ofrecer las mismas pantallas y reglas que la aplicación web para los usuarios de empresa, a partir del mismo código.
- **RF-002:** La sesión DEBE guardarse en el almacenamiento protegido del sistema y borrarse al cerrar sesión; NO DEBE incluirse en las copias de seguridad del teléfono.
- **RF-003:** La app NO DEBE ofrecer el panel de la plataforma (A-65).
- **RF-004:** Imprimir y descargar DEBEN funcionar dentro de la app: abrir o compartir el PDF o archivo con las apps del teléfono (A-63).
- **RF-005:** El botón «Atrás» de Android DEBE cerrar primero diálogos y hojas, luego volver de pantalla y, en la pantalla inicial, salir.
- **RF-006:** La app DEBE avisar cuando no hay conexión sin perder la venta en curso; no registra ventas sin conexión (A-66).
- **RF-007:** Las versiones para usuarios DEBEN conectarse solo por HTTPS a la API del SaaS (A-62).
- **RF-008:** La app DEBE detectar si su versión ya no es compatible con la API y pedir actualizarla (A-67).
- **RF-009:** La app DEBE poder distribuirse a las empresas del piloto según A-61, firmada siempre con la misma clave, que NO DEBE estar en los repositorios.

## Entidades clave

- **Versión de la app:** número de versión de cada entrega y la versión mínima que la API acepta.

## Criterios de éxito

- **CE-001:** En un Android de gama baja real se instala la app, se entra, se vende un ticket y una boleta (SUNAT beta), se imprime o comparte el PDF de 80 mm y se cierra sesión.
- **CE-002:** La app abre en menos de 3 s en un teléfono de gama baja con la sesión guardada (hasta ver el Inicio).
- **CE-003:** El token de sesión no aparece en el almacenamiento web de la app ni en una copia de seguridad del teléfono (verificación documentada).
- **CE-004:** Las comprobaciones del principio VI siguen en verde y la versión web no cambia su comportamiento.

## Fuera de alcance

- iPhone (iOS). El mismo código lo permitirá después, en otra spec.
- Impresión Bluetooth directa a impresoras térmicas (ESC/POS): spec propia (visión, fuera del MVP).
- Ventas sin conexión que se sincronizan después.
- Lector de código de barras con la cámara.
- Notificaciones push.
- Panel de la plataforma en la app.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-06 | A-61 Distribución | APK directo para el piloto; Google Play al pasar a producción. Nombre e identificador antes de la primera entrega |
| 2026-10-06 | A-62 Servidor | Desarrollo por la red local; entregas solo contra la API con HTTPS de la 009 |
| 2026-10-06 | A-63 Imprimir y descargar | Menú «Compartir» de Android; Bluetooth en otra spec |
| 2026-10-06 | A-64 Sesión | Almacenamiento protegido de Android, sin copias de seguridad |
| 2026-10-06 | A-65 Plataforma | Fuera de la app |
| 2026-10-06 | A-66 Sin conexión | Aviso; sin ventas sin conexión |
| 2026-10-06 | A-67 Versiones | Versión mínima desde la API |
