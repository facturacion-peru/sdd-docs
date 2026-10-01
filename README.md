# Facturación electrónica SUNAT — SaaS multiempresa

SaaS para pequeñas empresas del Perú: inventario, ventas con ticket interno y comprobantes electrónicos SUNAT (boletas, facturas y notas de crédito), con un panel para administrar la plataforma. Por ahora emite en el **ambiente beta** de SUNAT. El paso a producción es la spec 008.

Este repositorio guarda la **documentación del proyecto**. El desarrollo sigue *Spec-Driven Development*: cada funcionalidad se especifica, planifica y aprueba aquí antes de escribir código.

## Repositorios

| Repositorio | Qué contiene | Stack |
|---|---|---|
| **[sdd-docs](https://github.com/facturacion-peru/sdd-docs)** (este) | Constitución, visión del producto, specs, aclaraciones e investigación | Markdown |
| **[Api-de-facturacion-electronica-sunat-Peru](https://github.com/facturacion-peru/Api-de-facturacion-electronica-sunat-Peru)** | API REST multiempresa, emisión SUNAT, scripts de despliegue y operación | Laravel 12, PHP 8.3+, PostgreSQL, Greenter |
| **[frontend-api-facturacion-electronica-sunat](https://github.com/facturacion-peru/frontend-api-facturacion-electronica-sunat)** | Aplicación web de las empresas y del panel de plataforma (`/plataforma`); futura APK | Vue 3, TypeScript, Vite, pnpm |

La API parte de un fork de [yorchavez9/Api-de-facturacion-electronica-sunat-Peru](https://github.com/yorchavez9/Api-de-facturacion-electronica-sunat-Peru), reescrito con pruebas. El código original que todavía no se reincorporó está en su carpeta `legacy/`.

## Por dónde empezar

| Documento | Para qué |
|---|---|
| [Visión del producto](docs/producto/vision.md) | Problema, usuarios y alcance del SaaS |
| [Constitución](docs/constitution.md) | Principios no negociables y flujo de trabajo SDD |
| [Índice de specs](docs/specs/README.md) | Estado de cada capacidad (spec → plan → tareas → verificación) |
| [Aclaraciones](docs/aclaraciones.md) | Decisiones tomadas por el responsable del producto (`A-NN`) |
| [Evaluación de la API original](docs/investigacion/001-evaluacion-api-existente.md) | Qué se reutilizó del fork y qué no |
| [Plantillas](docs/plantillas) | Base para nuevas specs, planes y tareas |
| [AGENTS.md](AGENTS.md) | Reglas, trampas conocidas y bitácora para quien trabaje en el código (personas o agentes) |

## Entorno local

Los tres repositorios se clonan juntos: los dos proyectos van **dentro** de la carpeta de documentación, que los ignora. Así los enlaces y comandos de `AGENTS.md` funcionan tal cual.

```bash
git clone git@github.com:facturacion-peru/sdd-docs.git facturacion-electronica-sunat
cd facturacion-electronica-sunat
git clone git@github.com:facturacion-peru/Api-de-facturacion-electronica-sunat-Peru.git
git clone git@github.com:facturacion-peru/frontend-api-facturacion-electronica-sunat.git
```

Después, cada proyecto se levanta según su README: [API](https://github.com/facturacion-peru/Api-de-facturacion-electronica-sunat-Peru#puesta-en-marcha) y [frontend](https://github.com/facturacion-peru/frontend-api-facturacion-electronica-sunat#puesta-en-marcha).

## Despliegue y operación

La spec [009](docs/specs/009-despliegue-operacion/spec.md) cubre el servidor, CI/CD con GitHub Actions, las copias cifradas y las alertas. Las guías están en la API, en la rama `009-despliegue-operacion` hasta que se integre: [preparación](https://github.com/facturacion-peru/Api-de-facturacion-electronica-sunat-Peru/blob/009-despliegue-operacion/deploy/PREPARACION.md) y [restauración](https://github.com/facturacion-peru/Api-de-facturacion-electronica-sunat-Peru/blob/009-despliegue-operacion/deploy/RESTAURAR.md).

> Los repositorios son públicos: ningún secreto (claves SOL, certificados, `APP_KEY`, contraseñas) se guarda en ellos.
