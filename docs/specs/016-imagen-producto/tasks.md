# Tareas 016: Imagen del producto y logo en S3

**Estado:** Borrador para aprobación · **Fecha:** 2026-10-29 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior. 🔒 = requiere confirmación explícita del usuario. ☁️ = necesita la configuración de AWS del usuario.
> Reparto: las tareas **T** (API, operación y cierre) las lleva la sesión de backend; las **F** (frontend), la sesión del frontend.

## Fase 1 · Preparación (API)

- [ ] **T001** Rama `016-imagen-producto` desde `main` en la API — Verificación: `git branch --show-current`
- [ ] **T002** Configuración, sin cambiar comportamiento:
  - disco `images` (S3, raíz `AWS_ROOT`, `throw => true`);
  - `filesystems.images_disk` (`IMAGES_DISK`, por defecto `images`);
  - en el disco `s3` de las copias, la raíz `AWS_BACKUP_ROOT` y las claves `AWS_BACKUP_*` (por defecto las `AWS_*`);
  - variables documentadas en `.env.example`;
  - `exif` en `PHP_EXTENSIONS` de la CI.

  **Al terminar, avisar al usuario que renombre `AWS_BACKUP_BUCKET` → `AWS_BACKUP_ROOT` y agregue `IMAGES_DISK=public` en su `.env`.** Verificación: suites en verde, `php artisan config:show filesystems.disks.s3.root` con su `.env` y `actionlint`
- [ ] **T003** Generador de OpenAPI (pruebas primero): con reglas `file` o `image`, el cuerpo pasa a `multipart/form-data` y sin `maxLength`. `company/logo` queda bien documentado — Verificación: prueba del generador en verde y `public/openapi.json` regenerado

## Fase 2 · API: proceso de imágenes

- [ ] **T004** `tests/Support/TestImages`: genera JPEG (con EXIF de orientación y GPS armados a mano), PNG con alfa, WebP, GIF y archivos falsos, con sus pruebas — Verificación: en verde
- [ ] **T005** Pruebas de `ImageProcessor` (deben fallar):
  - orientación 6 y 8 → la salida queda derecha;
  - la salida no trae `Exif` ni GPS;
  - alfa conservado;
  - 800 y 200 px sin agrandar;
  - WebP de salida;
  - contenido falso → excepción.

  Verificación: en rojo
- [ ] **T006** `App\Images\ImageProcessor` — Verificación: T005 en verde
- [ ] **T007** [P] Pruebas y `App\Images\LogoProcessor`: PNG con transparencia → PNG con alfa, reducción a 800 px de ancho y sin metadatos — Verificación: en verde

## Fase 3 · API: fotos de producto (HU-1, HU-2)

- [ ] **T008** Migraciones `products.image_key` e `image_deletions`, y modelo `ImageDeletion` — Verificación: migran en SQLite y PostgreSQL
- [ ] **T009** Pruebas de las rutas y entradas en `TenantRoutes.php` (deben fallar):
  - subir, cambiar y quitar, con los objetos en el disco simulado;
  - `DELETE` sin foto da 200;
  - 422 por formato, tamaño, dimensiones o contenido;
  - 403 para el vendedor y 404 para otra empresa;
  - auditoría `product.image_updated` y `product.image_removed`;
  - forma de `image` en `GET`, y direcciones con UUID, sin RUC, id ni nombre.

  Verificación: en rojo
- [ ] **T010** `ProductImageRequest`, `App\Images\ImageStore`, `ProductImageService` (bloqueo, borrado posterior e `image_deletions`), las dos rutas en `ProductController` e `image` en `ProductResource` — Verificación: T009 en verde
- [ ] **T011** Pruebas de RF-010 con un disco que lanza excepciones (deben fallar): `POST` da 503 y el producto no cambia; `GET` sigue respondiendo; un borrado fallido va a `image_deletions`. Luego el manejo — Verificación: en verde
- [ ] **T012** [P] Pruebas y comando `images:cleanup` (reintentos, se rinde a los 10 y deja aviso), programado a las 02:45 en `routes/console.php` — Verificación: en verde
- [ ] **T013** Prueba de subidas simultáneas al mismo producto en `tests/Integration` (PostgreSQL): queda una imagen y ningún objeto huérfano — Verificación: en verde en PostgreSQL
- [ ] **T014** CE-003: producto de la empresa B con foto en `CrossTenantAccessTest`, con su clave entre los `bMarkers`; forma de `image` en `tests/Feature/Contract` — Verificación: en verde

## Fase 4 · API: logo en S3 (HU-4)

- [ ] **T015** Pruebas del logo (deben fallar):
  - la subida guarda PNG con alfa en el disco de imágenes y borra el anterior;
  - `logo_url` usa el disco de imágenes, o `public` si es una ruta antigua;
  - el PDF lleva el logo, lo conserva tras borrarlo del disco (caché) y sale sin logo con 200 si el disco falla.

  Verificación: en rojo
- [ ] **T016** `CompanyLogo` (`url` y `dataUri` con caché), `CompanyService::updateLogo` con `LogoProcessor` e `ImageStore`, validación de 4000×4000 en `UpdateCompanyLogoRequest`, y `DocumentPdf` con `CompanyLogo` — Verificación: T015 en verde, y `PrintParityTest` y las pruebas de PDF en verde
- [ ] **T017** Pruebas y comando `images:move-logos` (idempotente, sigue si uno falla), y su llamada en `deploy/release.sh` después de `migrate` — Verificación: en verde y `shellcheck deploy/release.sh`
- [ ] **T018** `php artisan openapi:generate`, suites SQLite y PostgreSQL y Pint — Verificación: códigos de salida 0 y cifras anotadas

## Fase 5 · Frontend

> **Pendiente de la sesión del frontend** (tareas F001 en adelante, según la sección «Frontend» del plan). Depende de T003 y T018 de la API para `pnpm api:types`. T-F1 (prueba de cámara y galería en el emulador) puede ir en paralelo con la API.

## Cierre

- [ ] **T019** ☁️ Configuración de AWS del usuario (plan, «Operación en AWS»): política del bucket, claves IAM, versionado y `.env` — Verificación: el usuario confirma
- [ ] **T020** ☁️ Verificación manual contra el bucket de pruebas (plan, pasos 1 a 6): CE-001, CE-002, CE-004, RF-016 (listar sin credenciales, la clave de la app fuera de `AWS_BACKUP_ROOT`, `backup:run` y el versionado) y el logo con `images:move-logos` — Verificación: resultados anotados en `verificacion.md`
- [ ] **T021** Comprobaciones del principio VI en verde (API SQLite y PostgreSQL, Pint; frontend completo) — Verificación: códigos de salida 0 y cifras anotadas
- [ ] **T022** Documentación:
  - README de la API (disco de imágenes, variables y comandos);
  - trampas nuevas en `AGENTS.md`;
  - bitácora;
  - estado en `docs/specs/README.md`.

  El README del frontend lo actualiza su sesión. Verificación: revisada
- [ ] **T023** 🔒 Merge local de `016-imagen-producto` a `main` en ambos repositorios y commit de la raíz — Verificación: suites en verde sobre `main`
