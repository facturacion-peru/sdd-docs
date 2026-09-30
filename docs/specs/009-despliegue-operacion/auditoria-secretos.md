# Auditoría de secretos antes de publicar (T003)

**Fecha:** 2026-09-30 · **Alcance:** historial completo de los tres repositorios (raíz 51 commits, API 189, frontend 49)

## Resultado

| Repositorio | Hallazgo | Riesgo |
|---|---|---|
| Raíz (`docs/`) | Nada | — |
| Frontend | Solo contraseñas ficticias de las pruebas (`clave-segura-123`, `secreto`) | Ninguno |
| API · nuestros commits (129) | Solo el texto «PRIVATE KEY» dentro de pruebas y del código que lo busca; los certificados de prueba se generan al ejecutar | Ninguno |
| API · commits del autor original (60) | `certificado.pem`, `certificado.pfx`, `ejemplos-postman/certificado_prueba/certificado.pem` y el seeder `CompanySeeder`: **certificado de demostración público** de Greenter/llama.pe (RUC ficticio 20123456789) y credenciales genéricas de SUNAT beta (`MODDATOS`) | Ninguno: son públicos por diseño |
| API · commit `5e146fe` del autor original | **`.env.beta`** con lo que parecen credenciales reales **del autor original**: contraseña de base de datos (20 caracteres), usuario y contraseña de correo SMTP. Se borró en `711159e`, pero sigue en el historial | **Medio**: no son del SaaS, pero publicar el historial las vuelve a exponer. Ya son públicas en el repositorio original |

No hay claves de servicios en la nube, tokens de GitHub ni claves privadas reales en ninguno de los tres repositorios.

## Otros puntos para publicar

- **Licencia:** el proyecto original no tiene archivo `LICENSE`; `composer.json` dice `MIT` (valor por defecto del esqueleto de Laravel) y su README dice «uso libre y bajo responsabilidad de quien lo usa». Conviene añadir un `LICENSE` propio y conservar la atribución que ya está en el README.
- **Correos en los commits:** los commits llevan el correo del autor de cada uno (el tuyo y el del autor original). Al publicar quedan visibles; GitHub permite usar un correo `noreply` para los commits futuros.
- **Ejemplos del autor original:** `ejemplos-postman/` (colección y certificado de demostración) sigue en el árbol. No es sensible.

## Decisión sobre `.env.beta`

Pendiente del usuario (ver la respuesta del agente del 2026-09-30).
