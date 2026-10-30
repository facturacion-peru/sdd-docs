# Tareas 016: Imagen del producto y logo en S3

**Estado:** Aprobadas (2026-10-29) · **Fecha:** 2026-10-29 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior. 🔒 = requiere confirmación explícita del usuario. ☁️ = necesita la configuración de AWS del usuario.
> Reparto: las tareas **T** (API, operación y cierre) las lleva la sesión de backend; las **F** (frontend), la sesión del frontend.

## Fase 1 · Preparación (API)

- [x] **T001** Rama `016-imagen-producto` desde `main` en la API — Verificación: `git branch --show-current`
- [x] **T002** Configuración, sin cambiar comportamiento:
  - disco `images` (S3, raíz `AWS_ROOT`, `throw => true`);
  - `filesystems.images_disk` (`IMAGES_DISK`, por defecto `images`);
  - en el disco `s3` de las copias, la raíz `AWS_BACKUP_ROOT` y las claves `AWS_BACKUP_*` (por defecto las `AWS_*`);
  - variables documentadas en `.env.example`;
  - `exif` en `PHP_EXTENSIONS` de la CI.

  **Al terminar, avisar al usuario que renombre `AWS_BACKUP_BUCKET` → `AWS_BACKUP_ROOT` y agregue `IMAGES_DISK=public` en su `.env`.** Verificación: suites en verde, `php artisan config:show filesystems.disks.s3.root` con su `.env` y `actionlint`
- [x] **T003** Generador de OpenAPI (pruebas primero): con reglas `file` o `image`, el cuerpo pasa a `multipart/form-data` y sin `maxLength`. `company/logo` queda bien documentado — Verificación: prueba del generador en verde y `public/openapi.json` regenerado

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

> Las lleva la sesión del frontend, según la sección «Frontend» del plan.
> - **Dependencias de la API:** F010 necesita T003 y F016 necesita T018.
> - **En paralelo con la API:** de F001 a F009 no dependen de ella, y F002 puede empezar enseguida.
> - **Verificación completa:** `pnpm type-check && pnpm lint && pnpm test:unit --run && pnpm build-only`.

- [x] **F001** Rama `016-imagen-producto` desde `main` en el frontend — Verificación: `git branch --show-current`
- [x] **F002** Prueba en el emulador (T-F1), sin tocar el código de la rama, con una página de prueba en la variante *debug*. Comprobar:
  - `<input type="file" capture="environment">` abre la cámara y sin `capture` abre el selector de fotos de Android 13+;
  - funciona sin declarar `CAMERA` y sin permiso de almacenamiento;
  - «Atrás» dentro de la cámara y del selector;
  - la foto llega como `File` con su orientación.

  Si no alcanza: proponer `@capacitor/camera` 8.x 🔒 (revisión, `minimumReleaseAge` y aprobación del usuario). Verificación: resultado y decisión en `verificacion.md`
- [ ] **F003** Separar en `core/api` la parte común de las cabeceras (token, `Accept` y `X-App-Version`), que usarán `authMiddleware` y `upload.ts` — Verificación: suite completa en verde, sin cambios de comportamiento
- [ ] **F004** Pruebas de `core/api/upload.ts` (deben fallar):
  - cabeceras;
  - `onProgress`;
  - 422 → `ApiError` con `errors.image`;
  - 503 → error general con el `message`;
  - 401 → `clearToken`, y 426 → `updateRequired`;
  - abortar sin error.

  Luego `uploadForm` con XHR. Verificación: en verde
- [ ] **F005** [P] Pruebas de `shared/images/prepare.ts` (deben fallar):
  - *magic bytes*: HEIC con nombre `.jpg` → rechazado;
  - más de 10 MB → rechazado sin decodificar;
  - dimensiones leídas en la cabecera de JPEG (SOF), PNG (IHDR) y WebP (VP8, VP8L y VP8X), y más de 50 MP → rechazado;
  - 4000×3000 → 1600×1200 en `image/jpeg` y menos de 2 MB, con un segundo intento a calidad 0.7;
  - imagen ilegible → «No se pudo leer la imagen».

  Verificación: en rojo
- [ ] **F006** `prepareProductPhoto` y la validación del logo (formato y 1 MB, sin reducir) — Verificación: F005 en verde
- [ ] **F007** `Device.pickImage(source)` en `types.ts`, `web.ts` y `native.ts`, según F002 ([verificación](verificacion.md)), con pruebas del `<input>` temporal: `camera` → `accept="image/*"` y `capture`; `gallery` → los tres formatos; `cancel` → `null`. En Android:
  - `<queries>` para `IMAGE_CAPTURE` y `external-files-path` `Pictures/` en `file_paths.xml`, **sin** `CAMERA`;
  - borrar la foto temporal de `Pictures/` después de leerla (`@capacitor/filesystem`, ya instalado) — Verificación: pruebas de `core/device` en verde
- [ ] **F008** `ProductImage` y `Product.image` en `inventory/types.ts`, y `CatalogProduct` en `sales/api.ts`. Después, pruebas y `shared/ui/ProductThumb.vue`:
  - marcador sin foto y con `@error`, con `aria-hidden`;
  - `alt` igual al nombre;
  - `loading="lazy"`;
  - la misma caja con foto y sin ella;
  - `thumb_url` en `sm` y `md`, y `url` en `lg`.

  Verificación: en verde y `theme.spec.ts` en verde
- [ ] **F009** Pruebas y `shared/ui/ImagePicker.vue`:
  - vista previa, descartar sin emitir y liberar la URL al desmontar;
  - en la web, abre directo el selector;
  - en la app, `BaseDialog` con «Tomar foto» y «Elegir de la galería», que se cierra con «Atrás» (`dismissTop`);
  - el error de `prepareProductPhoto` se muestra.

  Verificación: en verde
- [ ] **F010** (después de T003) `pnpm api:types`. `inventoryApi.uploadImage` con `uploadForm` y `inventoryApi.removeImage`, con sus pruebas — Verificación: `pnpm type-check` y pruebas en verde
- [ ] **F011** Pruebas y cambios en `ProductFormView.vue` (HU-1, esc. 6 a 8):
  - sección «Foto» solo para el administrador;
  - al crear, guarda y después sube; si falla, va a la ficha con el aviso y «Reintentar»;
  - al editar, sube con su avance, sin doble envío y con «Cancelar»;
  - sin conexión, guarda los datos y avisa.

  Verificación: en verde
- [ ] **F012** Pruebas y cambios en `ProductDetailView.vue` (HU-1, esc. 3 y 5, y HU-2, esc. 3):
  - foto grande;
  - «Cambiar foto» y «Quitar foto» con confirmación, solo para el administrador;
  - 422 y 503 con el mensaje de la API.

  Verificación: en verde
- [ ] **F013** [P] `ProductThumb` en `ProductsView.vue` (`sm`, `grid-cols-1`) y en `ProductCatalog.vue` (`md`, también en el esqueleto), con sus pruebas. El carrito no muestra fotos — Verificación: en verde
- [ ] **F014** [P] Prueba (debe fallar) y cambio en `sale-draft.ts`: `add()` copia solo los campos de `SellableProduct`, y `localStorage` no guarda `image` (HU-2, esc. 4) — Verificación: en verde
- [ ] **F015** [P] Pruebas y cambios en `CompanyView.vue`: el logo se sube con `uploadForm` (con avance), con la validación previa de formato y 1 MB, y sin reducirlo (HU-4, esc. 5) — Verificación: en verde
- [ ] **F016** (después de T018) `pnpm api:types` con el `openapi.json` final. `version` 0.3.0 en `package.json` — Verificación: verificación completa del frontend con código de salida 0 y cifras anotadas, y `pnpm android:dev` compila e instala el APK *debug* (JDK 21)
- [ ] **F017** ☁️ Verificación manual del frontend (parte de T020, con la API de la rama y el bucket de pruebas):
  - 360 px y modo oscuro en «Vender», «Productos» y la ficha;
  - TalkBack en el catálogo;
  - en el emulador y en el teléfono real: cámara, galería, «Atrás», foto vertical (CE-004) y modo avión con el formulario abierto;
  - **CE-001** cronometrado en el teléfono con datos móviles;
  - **CE-002** con `chrome://inspect`.

  Verificación: resultados en `verificacion.md`
- [ ] **F018** README del frontend: reducción de fotos, `pickImage` y `ProductThumb`. Las trampas nuevas del frontend se pasan a T022 — Verificación: revisada

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
