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
