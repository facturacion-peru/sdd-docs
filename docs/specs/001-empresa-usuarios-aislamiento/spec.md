# Spec 001: Empresa, usuarios, roles y aislamiento multiempresa

**Estado:** Aprobada · **Versión:** 1.0 · **Fecha:** 2026-09-26
**Depende de:** — · **Aclaraciones:** A-01 a A-11, A-25

> Qué y por qué. Las decisiones técnicas van en `plan.md`, que se redacta cuando esta spec esté aprobada.
> Las marcas `[A-xx]` indican la aclaración de la que sale cada requisito; todas están resueltas (ver *Clarificaciones*).

## Objetivo

Que el administrador de la plataforma pueda dar de alta empresas y que cada empresa opere con sus propios usuarios y roles, **sin poder ver ni tocar datos de otra empresa**. Es la base de todas las demás capacidades: sin aislamiento confiable el SaaS no es viable.

## Historias de usuario

### HU-1 · Alta de empresa (P1)

Como administrador de la plataforma, quiero registrar una empresa con sus datos legales y su primer administrador, para que pueda empezar a operar. `[A-03]`

**Por qué esta prioridad:** es el punto de entrada de todo lo demás.
**Prueba independiente:** crear una empresa y comprobar que su administrador puede entrar y ve su empresa vacía.

**Escenarios de aceptación**

1. **Dado** que soy administrador de la plataforma, **cuando** registro una empresa con RUC, razón social, régimen tributario, domicilio fiscal y el correo de su administrador, **entonces** la empresa queda creada y activa, con su establecimiento principal `[A-07]`, y el administrador recibe una invitación.
2. **Dado** un RUC que ya existe en la plataforma, **cuando** intento registrarlo, **entonces** el sistema lo rechaza indicando que la empresa ya está registrada.
3. **Dado** un RUC con formato o dígito verificador inválido, **cuando** intento registrarlo, **entonces** el sistema lo rechaza y explica el motivo.
4. **Dado** un RUC que empieza por `10`, **cuando** registro la empresa, **entonces** el sistema la identifica como persona natural sin preguntarlo; con `20`, como persona jurídica `[A-11]`.

### HU-2 · Acceso de los usuarios (P1)

Como usuario de una empresa, quiero activar mi cuenta desde la invitación, iniciar sesión y cerrarla, para trabajar de forma segura.

**Prueba independiente:** aceptar una invitación, iniciar sesión, llamar a un recurso protegido y cerrar sesión.

**Escenarios de aceptación**

1. **Dado** una invitación vigente, **cuando** defino mi contraseña, **entonces** mi cuenta queda activa y puedo iniciar sesión.
2. **Dado** una invitación vencida o ya usada, **cuando** intento usarla, **entonces** el sistema la rechaza y me indica que pida una nueva.
3. **Dado** credenciales correctas, **cuando** inicio sesión, **entonces** accedo a la aplicación con el contexto de mi empresa y mi rol.
4. **Dado** varios intentos fallidos seguidos, **cuando** sigo intentando, **entonces** el sistema bloquea temporalmente el acceso a esa cuenta.
5. **Dado** una sesión caducada, **cuando** hago cualquier acción, **entonces** se me pide iniciar sesión de nuevo sin perder lo que estaba escribiendo, cuando sea razonable (principio VII).
6. **Dado** un usuario desactivado o de una empresa desactivada, **cuando** intenta iniciar sesión o usar una sesión abierta, **entonces** el sistema lo rechaza.
7. **Dado** que olvidé mi contraseña, **cuando** pido recuperarla con mi correo, **entonces** recibo un enlace de un solo uso y vigencia limitada para definir otra. La respuesta es la misma exista o no el correo `[A-25]`.
8. **Dado** que cambié mi contraseña por recuperación, **cuando** termino, **entonces** se cierran mis demás sesiones abiertas.

### HU-3 · Gestión de usuarios de la empresa (P1)

Como administrador de empresa, quiero invitar usuarios, asignarles un rol y desactivarlos, para controlar quién opera en mi negocio. `[A-05]`

**Prueba independiente:** invitar a un vendedor, comprobar que no accede a la gestión de usuarios y desactivarlo.

**Escenarios de aceptación**

1. **Dado** que soy administrador de empresa, **cuando** invito a una persona con rol vendedor, **entonces** recibe una invitación y, al aceptarla, queda en mi empresa con ese rol.
2. **Dado** que soy vendedor, **cuando** intento gestionar usuarios o cambiar la configuración de la empresa, **entonces** el sistema lo impide.
3. **Dado** que soy el único administrador activo de mi empresa, **cuando** intento desactivarme o quitarme el rol, **entonces** el sistema lo impide.
4. **Dado** un correo que ya pertenece a un usuario de otra empresa, **cuando** lo invito, **entonces** el sistema lo rechaza sin revelar a qué empresa pertenece `[A-04]`.

### HU-4 · Aislamiento entre empresas (P1)

Como empresa cliente, quiero la garantía de que ninguna otra empresa puede ver ni modificar mis datos.

**Prueba independiente:** con dos empresas A y B, un usuario de A intenta leer, modificar y listar cada tipo de recurso de B.

**Escenarios de aceptación**

1. **Dado** un usuario de A, **cuando** pide un recurso de B por su identificador, **entonces** recibe la misma respuesta que para un recurso inexistente, sin ningún dato de B.
2. **Dado** un usuario de A, **cuando** lista cualquier recurso, **entonces** solo ve registros de A, aunque envíe filtros o parámetros que apunten a B.
3. **Dado** un usuario de A, **cuando** intenta crear o modificar un recurso asociándolo a B, **entonces** el sistema lo rechaza.
4. **Dado** el administrador de la plataforma, **cuando** consulta empresas, **entonces** ve sus datos generales y su estado, pero **nunca** sus secretos.

### HU-5 · Datos de la propia empresa (P2)

Como administrador de empresa, quiero consultar y corregir los datos de mi empresa (nombre comercial, contacto, logo), para que aparezcan bien en mis documentos.

**Escenarios de aceptación**

1. **Dado** que soy administrador de empresa, **cuando** edito el nombre comercial, el teléfono, el correo o el logo, **entonces** los cambios se guardan y quedan auditados.
2. **Dado** que soy administrador de empresa, **cuando** intento cambiar el RUC, **entonces** el sistema lo impide: el RUC solo lo corrige el administrador de la plataforma.

### HU-6 · Auditoría de acciones sensibles (P2)

Como administrador de empresa, quiero ver quién hizo cada acción sensible y cuándo, para investigar problemas.

**Escenarios de aceptación**

1. **Dado** que se crea, modifica o desactiva un usuario, o cambian los datos de la empresa, **cuando** consulto la auditoría, **entonces** veo quién, cuándo, qué acción y sobre qué registro.
2. **Dado** un registro de auditoría, **cuando** cualquier usuario intenta modificarlo o borrarlo, **entonces** el sistema no lo permite.
3. **Dado** un cambio que involucra un secreto, **cuando** se audita, **entonces** se registra que cambió, pero **nunca** su valor.

### HU-7 · Gestión mínima de empresas por la plataforma (P2)

Como administrador de la plataforma, quiero listar, desactivar y reactivar empresas, para dar soporte y suspender el servicio si hace falta. El panel completo se especifica en la 006.

**Escenarios de aceptación**

1. **Dado** una empresa activa, **cuando** la desactivo, **entonces** sus usuarios pierden el acceso de inmediato, pero sus datos se conservan.
2. **Dado** una empresa desactivada, **cuando** la reactivo, **entonces** sus usuarios recuperan el acceso.

## Casos límite

- Dos altas simultáneas con el mismo RUC: solo una prospera.
- Invitación a un correo con mayúsculas o espacios distintos: se trata como el mismo correo.
- El administrador de la plataforma intenta operar ventas de una empresa: no puede, su rol no es operativo `[A-05]`.
- Un usuario con la sesión abierta es desactivado: su siguiente petición se rechaza.
- Enumeración de identificadores (probar 1, 2, 3…): no revela qué recursos existen en otras empresas.

## Requisitos funcionales

**Empresa**

- **RF-001:** El sistema DEBE permitir al administrador de la plataforma registrar empresas con RUC, razón social, nombre comercial (opcional), régimen tributario, domicilio fiscal con ubigeo, correo y teléfono de contacto `[A-03]` `[A-11]`.
- **RF-002:** El sistema DEBE validar el formato y el dígito verificador del RUC y rechazar RUC duplicados.
- **RF-003:** El sistema DEBE deducir el tipo de persona a partir del RUC `[A-11]`.
- **RF-004:** El sistema DEBE crear automáticamente el establecimiento principal de cada empresa `[A-07]`.
- **RF-005:** El sistema DEBE permitir desactivar y reactivar empresas, conservando sus datos.

**Usuarios y acceso**

- **RF-010:** El sistema DEBE dar de alta usuarios solo por invitación, con enlace de un solo uso y de vigencia limitada.
- **RF-011:** Cada usuario de empresa DEBE pertenecer a exactamente una empresa; el administrador de la plataforma, a ninguna `[A-04]`.
- **RF-012:** El sistema DEBE permitir iniciar y cerrar sesión, y bloquear temporalmente una cuenta tras intentos fallidos repetidos.
- **RF-013:** El sistema DEBE rechazar a usuarios desactivados y a usuarios de empresas desactivadas, incluso si tienen una sesión abierta.
- **RF-014:** Las sesiones DEBEN caducar; la aplicación DEBE pedir un nuevo inicio de sesión al caducar.
- **RF-015:** El sistema DEBE impedir que una empresa se quede sin administrador activo.
- **RF-016:** El sistema DEBE permitir recuperar la contraseña por correo con un enlace de un solo uso y vigencia limitada, sin revelar si el correo existe, y cerrar las demás sesiones al completarla `[A-25]`.

**Roles**

- **RF-020:** El sistema DEBE tener tres roles fijos: administrador de plataforma, administrador de empresa y vendedor `[A-05]`.
- **RF-021:** Los permisos DEBEN evaluarse en el servidor en cada operación, según el rol y la empresa del usuario.

**Aislamiento**

- **RF-030:** Todo dato de negocio DEBE pertenecer a una empresa.
- **RF-031:** Toda lectura, escritura y listado DEBE limitarse a la empresa del usuario autenticado, sin depender de parámetros enviados por el cliente.
- **RF-032:** El acceso a un recurso de otra empresa DEBE responderse igual que un recurso inexistente.
- **RF-033:** Cada operación que exponga datos de negocio DEBE tener una prueba automatizada de acceso cruzado.

**Auditoría**

- **RF-040:** El sistema DEBE auditar el alta, la modificación y la desactivación de empresas y usuarios, los cambios de rol, los inicios de sesión fallidos y las recuperaciones de contraseña.
- **RF-041:** Los registros de auditoría DEBEN ser inmutables y NO DEBEN contener valores de secretos.

**Deuda heredada** (constitución y [evaluación](../../investigacion/001-evaluacion-api-existente.md))

- **RF-050:** NO DEBE existir ningún endpoint público que ejecute migraciones, seeders o cambios de configuración del sistema.
- **RF-051:** Fuera del entorno de desarrollo, el acceso entre orígenes (CORS) DEBE limitarse a los orígenes conocidos.

## Entidades clave

- **Empresa:** RUC, razón social, nombre comercial, tipo de persona, régimen, domicilio fiscal, contacto, logo, estado.
- **Establecimiento:** local de la empresa con su código de anexo SUNAT y su dirección. En el MVP, solo el principal.
- **Usuario:** persona con correo, nombre, estado y, si es de empresa, su rol en ella.
- **Invitación:** alta pendiente de un usuario, con vigencia y un solo uso.
- **Registro de auditoría:** quién, cuándo, qué acción, sobre qué registro y de qué empresa.

## Criterios de éxito

- **CE-001:** El administrador de la plataforma da de alta una empresa y su administrador en menos de 3 minutos.
- **CE-002:** El 100 % de los endpoints de negocio tienen una prueba de acceso cruzado en verde.
- **CE-003:** Ninguna respuesta de la API, ningún log y ningún registro de auditoría contiene contraseñas ni secretos (verificado por prueba).
- **CE-004:** Un usuario desactivado pierde el acceso en su siguiente petición.

## Fuera de alcance

- Autoservicio público de registro `[A-03]`, suscripciones y cobros.
- Usuarios en varias empresas `[A-04]` y roles personalizables `[A-05]`.
- Varios establecimientos `[A-07]`.
- Credenciales SUNAT, certificados y series: **spec 004**.
- Panel completo de la plataforma: **spec 006**.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-09-26 | Se separa de esta spec la configuración fiscal y las series | Pasan a la 004 para no duplicar requisitos |
| 2026-09-26 | A-01 · Base técnica | Mismo repositorio, dominio reiniciado: el plan construye empresas, usuarios y roles de nuevo |
| 2026-09-26 | A-02 · Datos existentes | No hay datos reales: esquema limpio, sin migración |
| 2026-09-26 | A-03 · Alta de empresa | La crea el administrador de la plataforma y el administrador de empresa recibe una invitación (HU-1 confirmada) |
| 2026-09-26 | A-08 · Dónde administra la empresa | En el frontend Vue; HU-3, HU-5 y HU-6 son pantallas de Vue |
| 2026-09-26 | A-04, A-05, A-06, A-07, A-09, A-10 | Recomendaciones aceptadas: una empresa por usuario, tres roles fijos, base compartida con `company_id`, un establecimiento, tickets y SUNAT independientes, constitución 1.1.0 |
| 2026-09-26 | A-11 ⚖️ | Aceptada como decisión de trabajo; pendiente de confirmar con un contador antes de producción |
| 2026-09-26 | A-25 · Recuperación de contraseña | Entra en el MVP: escenarios HU-2.7 y HU-2.8, RF-016 |
| 2026-09-26 | **Aprobación** | Spec aprobada por el usuario (v1.0) |
