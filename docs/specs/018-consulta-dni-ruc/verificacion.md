# Verificación 018: Consulta de DNI y RUC

## T002 · Forma real de las respuestas de apisperu (2026-10-30)

Consultas con el token del usuario (`APISPERU_TOKEN` de su `.env`), desde un script fuera del repositorio que nunca imprime el token. Del DNI solo se registraron los nombres de los campos y sus largos, no los datos de la persona.

| Consulta | Respuesta |
|---|---|
| `GET /ruc/20131312955` (SUNAT) | HTTP 200, sin `success`. Campos: `ruc`, `razonSocial`, `nombreComercial` (null), `estado` («ACTIVO»), `condicion` («HABIDO»), `direccion` («AV. GARCILASO DE LA VEGA NRO. 1472 LIMA LIMA LIMA»: ya incluye departamento, provincia y distrito), `departamento`, `provincia`, `distrito`, **`ubigeo`** («150101»), `capital`, `fechaBaja` y listas vacías (`telefonos`, `actEconomicas`, `padrones`…) |
| `GET /ruc/20999999990` (dígito válido, inexistente) | **HTTP 200** con `{"success": false, "message": "No se encontraron resultados."}`, no 404 |
| `GET /dni/<un DNI real>` | HTTP 200 con `success: true`, `dni`, `nombres`, `apellidoPaterno`, `apellidoMaterno`, `codVerifica` (entero) y `codVerificaLetra` |
| `GET /dni/00000001` | HTTP 200 con una persona: no sirve como «inexistente». El DNI no encontrado se supone igual que el RUC (`success: false`) |
| `GET /ruc/…` con un token inválido | **HTTP 401** con `{"message": "Token inválido."}` |

**Ajustes al plan (sin cambio de la spec):**
- `not_found` es HTTP 200 con `success: false`, o 404.
- 401 y 403 (token inválido o vencido) se tratan como `unavailable`, con un aviso en el log sin el token: es una falla de configuración de la plataforma, no del cliente.
- La respuesta de cupo agotado no se pudo provocar; se toma como 429, o un mensaje con «límite» o «consultas».
- `direccion` se guarda tal como llega, porque ya trae el distrito. `ubigeo` llega en el RUC.

## T013 · API con el token real (2026-10-30)

Base local, empresa demo, rama `018-consulta-dni-ruc` de la API y el servicio real de apisperu.

| Prueba | Resultado |
|---|---|
| Búsqueda del RUC 20131312955 (nuevo en la empresa) | ✅ `source: service` en **1,16 s** en el servidor: «SUPERINTENDENCIA NACIONAL DE ADUANAS Y DE ADMINISTRACION TRIBUTARIA - SUNAT», «AV. GARCILASO DE LA VEGA NRO. 1472 LIMA LIMA LIMA», verificado |
| Segunda búsqueda del mismo RUC | ✅ `source: company` en 0,00 s, sin consultar el servicio |
| Factura F001-00000005 con ese cliente | ✅ Aceptada por SUNAT beta («La Factura numero F001-5, ha sido aceptada»), con el RUC y la razón social del servicio |
| Búsqueda con un token inválido | ✅ Sin cliente y `reason: unavailable`: la app ofrece el registro manual |

**Queda para la app** (F012, con el usuario): CE-001 cronometrado en el teléfono con datos móviles (menos de 3 s desde terminar de escribir), CE-003 con el servicio caído, 360 px, TalkBack y modo avión.

## F012 · App y navegador (en curso)

### CE-003 · Servicio caído (2026-11-02, probado por el usuario)

APIs de la rama `018-consulta-dni-ruc` con la base local y apisperu inalcanzable (`APISPERU_BASE_URL=https://127.0.0.1:9`): `:8001` para la app 0.5.0 del teléfono (Poco X3) y el navegador en `:5173`, y `:8002` para el navegador en `:5174`. Un RUC nuevo respondía `reason: unavailable` en 0,13 s.

| Prueba | Resultado |
|---|---|
| DNI nuevo con el servicio caído | ✅ La app dice que el servicio no está disponible y ofrece «Registrar a mano» |
| Registro manual | ✅ Dos clientes con DNI registrados a mano (en el «Domicilio fiscal» y en la «Sucursal Centro»), elegidos en la venta y marcados «Manual» |
| Boleta con el cliente manual | ✅ B001-00000024 y B002-00000005, aceptadas por SUNAT beta |
| Venta en curso | ✅ No se perdió al fallar la búsqueda |
| Lo ya guardado | ✅ Un documento ya registrado sigue saliendo de la base |

**Hallazgo (de la 017, fuera de esta spec):** con vendedor.centro@demo.test, «Factura» aparecía desactivada sin explicación. La «Sucursal Centro» no tenía serie de factura, y el aviso de «Vender» solo sale cuando están bloqueadas boleta y factura a la vez. Se creó una serie de factura para ese local; el aviso queda pendiente de la decisión del usuario.

**Falta:** CE-001 cronometrado, 360 px y modo oscuro, TalkBack y modo avión.
