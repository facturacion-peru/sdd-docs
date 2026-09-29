# Spec 006: Panel de administración de la plataforma

**Estado:** En aclaración · **Versión:** 0.1 · **Fecha:** 2026-09-29
**Depende de:** 001, 004, 005 · **Aclaraciones:** A-08, A-24 ⚖️, A-35, A-36, A-37, A-38

> Los requisitos marcados con `[A-xx]` siguen la recomendación de esa aclaración y se ajustarán si la decisión es otra.

## Objetivo

Que el administrador de la plataforma opere el SaaS **sin la consola**: dar de alta empresas, corregir sus datos legales, suspenderlas o reactivarlas, y saber qué empresas tienen problemas con la emisión SUNAT para darles soporte. Todo ello **sin ver los datos de negocio de las empresas** ni sus secretos.

Hoy la 001 dejó la API de plataforma (alta, edición, activar y desactivar) y los comandos `platform:create-admin` y `company:create`, pero ninguna pantalla: el administrador de la plataforma que entra a la app ve un aviso que lo manda a la consola.

## Historias de usuario

### HU-1 · Ver las empresas y su estado (P1)

Como administrador de la plataforma, quiero ver todas las empresas con su estado, para saber de un vistazo cuáles operan y cuáles necesitan ayuda.

**Escenarios de aceptación**

1. **Dado** varias empresas, **cuando** abro el panel, **entonces** veo por cada una: RUC, razón social, activa o no, estado de la configuración SUNAT y cuántos comprobantes tiene pendientes o rechazados `[A-37]`.
2. **Dado** muchas empresas, **cuando** busco por RUC o razón social, o filtro por «con problemas de emisión», **entonces** veo solo esas.
3. **Dado** una empresa, **cuando** abro su ficha, **entonces** veo sus datos legales, su administrador (nombre y correo, y si ya aceptó la invitación), cuántos usuarios tiene y la fecha de su última venta o comprobante, pero **nunca** secretos ni el detalle de productos, ventas o clientes `[A-37]`.

### HU-2 · Dar de alta una empresa (P1)

Como administrador de la plataforma, quiero registrar una empresa desde el panel, para no depender de la consola.

**Escenarios de aceptación**

1. **Dado** los datos legales, el domicilio fiscal con ubigeo y el correo de su administrador, **cuando** registro la empresa, **entonces** queda activa con su establecimiento principal y el administrador recibe la invitación (mismas reglas que HU-1 de la 001).
2. **Dado** un RUC ya registrado o inválido, **cuando** lo ingreso, **entonces** el panel lo explica.
3. **Dado** un administrador que no recibió o perdió la invitación, **cuando** la reenvío, **entonces** recibe un enlace nuevo y el anterior deja de valer `[A-36]`.

### HU-3 · Corregir datos legales (P1)

Como administrador de la plataforma, quiero corregir RUC, razón social, régimen o domicilio fiscal de una empresa, porque su administrador no puede hacerlo (001, HU-2.2).

**Escenarios de aceptación**

1. **Dado** una empresa, **cuando** corrijo un dato legal, **entonces** queda auditado quién lo cambió, de qué a qué.
2. **Dado** una empresa con comprobantes emitidos, **cuando** cambio su RUC, **entonces** el panel advierte que la configuración SUNAT vuelve a validarse y los comprobantes ya emitidos conservan el RUC con que se emitieron.

### HU-4 · Suspender y reactivar (P1)

Como administrador de la plataforma, quiero desactivar y reactivar empresas, para suspender el servicio si hace falta.

**Escenarios de aceptación**

1. **Dado** una empresa activa, **cuando** la desactivo con un motivo, **entonces** sus usuarios pierden el acceso de inmediato, sus datos se conservan y el motivo queda auditado.
2. **Dado** una empresa desactivada, **cuando** la reactivo, **entonces** sus usuarios recuperan el acceso.

### HU-5 · Soporte de la emisión (P2)

Como administrador de la plataforma, quiero ver qué comprobantes de las empresas están pendientes o rechazados, para ayudarles a resolverlo `[A-37]`.

**Escenarios de aceptación**

1. **Dado** comprobantes pendientes en varias empresas, **cuando** abro la vista de soporte, **entonces** veo por empresa el número, el estado, el código y el mensaje de SUNAT y los intentos, sin las líneas ni los datos del cliente.
2. **Dado** un comprobante pendiente, **cuando** pido reenviarlo, **entonces** se reintenta igual que con «Reintentar» de la empresa, y queda auditado que lo hizo la plataforma.

### HU-6 · Auditoría de la plataforma (P2)

Como administrador de la plataforma, quiero ver el historial de acciones de la plataforma (altas, cambios de datos legales, suspensiones, reenvíos), para rendir cuentas.

## Casos límite

- Desactivar una empresa con comprobantes pendientes: los reintentos automáticos siguen, porque la venta ya ocurrió y SUNAT debe recibirla.
- El administrador de la plataforma intenta entrar a la app de la empresa o ver sus ventas: no puede (A-05).
- Dos administradores de la plataforma editan la misma empresa a la vez: gana el último, y ambos cambios quedan auditados.
- Se corrige el RUC a uno que ya usa otra empresa: se rechaza.

## Requisitos funcionales

- **RF-001:** El panel DEBE ser accesible solo para administradores de la plataforma `[A-35]`.
- **RF-002:** El panel NO DEBE mostrar secretos (clave SOL, certificado, contraseñas) ni el detalle de productos, ventas, clientes o comprobantes de las empresas, salvo los metadatos de envío de HU-5 `[A-37]`.
- **RF-003:** Toda acción de la plataforma DEBE auditarse con su autor y su motivo cuando corresponda.
- **RF-004:** El alta, la edición, la desactivación y la reactivación DEBEN seguir las reglas de la spec 001.
- **RF-005:** Los administradores de la plataforma se siguen creando por consola (`platform:create-admin`) `[A-36]`.
- **RF-006:** El acceso del administrador de la plataforma DEBE protegerse según A-38.

## Criterios de éxito

- **CE-001:** El administrador de la plataforma da de alta una empresa y su administrador desde el panel en menos de 3 minutos (mismo objetivo que la 001, ahora sin consola).
- **CE-002:** Una búsqueda en todas las respuestas del panel no encuentra secretos ni datos de clientes o ventas de las empresas.
- **CE-003:** Desde el panel se identifica en menos de 1 minuto qué empresas tienen comprobantes pendientes o rechazados.

## Fuera de alcance

- Suscripciones, planes y cobros del SaaS (visión, fuera del MVP).
- Suplantar a un usuario de empresa («entrar como») `[A-37]`.
- Crear o gestionar administradores de la plataforma desde el panel `[A-36]`.
- Métricas de negocio de las empresas (ventas, ingresos).
- Paso a producción de SUNAT (A-20, A-24).

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-09-29 | Spec creada a partir de la visión (persona «administrador de la plataforma») y de lo que la 001 dejó pendiente (HU-7) | — |
