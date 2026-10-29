# Plan 016: Imagen del producto y logo en S3

**Estado:** Aprobado (2026-10-29) · **Fecha:** 2026-10-29 · **Spec:** [spec.md](./spec.md) (versión aprobada: 1.0)

> Reparto: la **API**, la configuración y la operación los escribe la sesión de backend (este documento). La sección **Frontend** la completa la sesión del frontend antes de la aprobación.

## Resumen técnico

**API**
- **Subida:** dos rutas nuevas, `POST` y `DELETE /products/{product}/image`, para el administrador.
- **Proceso con GD**, sin dependencias nuevas:
  - valida el contenido real;
  - gira la foto según su orientación EXIF y le quita los datos ocultos;
  - genera una versión de 800 px y una miniatura de 200 px en WebP.
- **Dónde se guarda:** en S3, bajo `AWS_ROOT/<uuid>.webp`.
- **Respuesta:** el producto devuelve `image: { url, thumb_url } | null`, con direcciones públicas fijas.

**Logo**
- **Dónde se guarda:** pasa a S3 como `AWS_ROOT/<uuid>.png`, en PNG para conservar la transparencia.
- **Logos existentes:** los mueve un comando idempotente que corre en cada despliegue.
- **PDF:** leen el logo de S3 con caché y, si S3 falla, salen sin logo.

**Configuración y operación**
- **Discos:** un disco `images` (S3, raíz `AWS_ROOT`). El disco `s3` de las copias pasa a la raíz `AWS_BACKUP_ROOT` y puede usar una clave propia.
- **Desarrollo local:** el disco local `public`, elegido con `IMAGES_DISK` (A-84).
- **AWS** (política del bucket, claves IAM y versionado): lo configura el usuario, y se comprueba en la verificación (RF-016).

## Contexto técnico

| Aspecto | Decisión |
|---|---|
| Proyectos afectados | API, frontend (incluido `android/`) y operación (AWS) |
| Ramas | `016-imagen-producto` en API y frontend, desde `main` |
| Dependencias nuevas | **Ninguna en la API.** `league/flysystem-aws-s3-v3` 3.35 y `aws/aws-sdk-php` 3.398 ya están (spec 009). GD con WebP, EXIF y fileinfo ya están instalados y en la CI (se agrega `exif` a `PHP_EXTENSIONS`) |
| Migraciones | `products.image_key` (UUID, nulo) y la tabla `image_deletions` (borrados pendientes). `companies.logo_path` se reutiliza |
| Variables nuevas | `AWS_ROOT`, `AWS_BACKUP_ROOT` (hoy el usuario la tiene como `AWS_BACKUP_BUCKET`: **hay que renombrarla**), `AWS_BACKUP_ACCESS_KEY_ID` y `AWS_BACKUP_SECRET_ACCESS_KEY` (opcionales, por defecto las `AWS_*`) e `IMAGES_DISK` (`images` o `public`) |
| Producción | Aún no hay servidor (spec 009), así que no hay copias ni logos reales que mudar fuera de local |

## Chequeo de constitución

| Principio | Cumple | Cómo |
|---|---|---|
| I. Spec primero | Sí | Spec 016 v1.0 aprobada, con A-79 a A-85 |
| II. Corrección fiscal | Sí | Los comprobantes no cambian. El logo de los PDF sale del mismo archivo, ahora en S3; si falla, el PDF sale sin logo, nunca con datos distintos |
| III. Contrato de API | Sí | `ProductImageRequest` documentado en OpenAPI. Se corrige el generador para que documente `multipart/form-data`. `image` se fija en `tests/Feature/Contract` |
| IV. Seguridad | Sí, con un costo aceptado | Se valida el contenido real y las dimensiones (contra bombas de descompresión), y se vuelve a codificar todo, así que no se guarda nada tal cual se subió. Las claves son UUID sin datos. Lectura pública solo de `AWS_ROOT/*`, sin listado, y la clave de la app no toca las copias. **Costo aceptado (A-79):** quien tiene la dirección de una foto la ve |
| V. Simplicidad | Sí | Sin librerías nuevas. Se suben los archivos por la API y no hay subida directa a S3, colas ni CORS. Los borrados pendientes van en una tabla, sin necesidad de listar el bucket |
| VI. Verificable | Sí | Pest con `Storage::fake` (sin red) y una verificación manual contra el bucket de pruebas del usuario, antes de cerrar |
| VII. Personas reales | Sí | Si S3 falla, se puede seguir vendiendo y editando productos (RF-010). Los PDF no se vuelven más lentos, gracias a la caché |
| VIII. Aislamiento y auditoría | Sí | Rutas en `TenantRoutes.php`. Las respuestas de una empresa no traen claves de otra (CE-003). Auditoría: `product.image_updated`, `product.image_removed` y la ya existente `company.logo_updated` |

## Almacenamiento y configuración

`config/filesystems.php`:

| Disco | Para qué | Raíz | Credenciales |
|---|---|---|---|
| `images` (nuevo, `s3`) | Fotos y logos | `AWS_ROOT` | `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` |
| `s3` (existente) | Copias de la spec 009 (`BACKUP_DISK=s3`) | `AWS_BACKUP_ROOT` (nuevo) | `AWS_BACKUP_*`, o las `AWS_*` si no se definen |
| `public` (existente) | Desarrollo local de imágenes (`IMAGES_DISK=public`) y logos antiguos hasta moverlos | — | — |

- **Disco a usar:** `config('filesystems.images_disk')` lee `IMAGES_DISK` y vale `images` por defecto. El código nunca nombra el disco directamente.
- **`images` con `throw => true`:** un fallo de S3 llega como excepción y no como un `false` silencioso.
- **Sin ACL por objeto:** el acceso público lo da la política del bucket, y los buckets nuevos de AWS rechazan los ACL (`BucketOwnerEnforced`). Cada objeto se sube con `ContentType` y `CacheControl: public, max-age=31536000, immutable`: la clave cambia en cada subida, así que la caché nunca queda vieja.
- **`.env.example`:** se agregan las variables `AWS_*` y las nuevas, con comentarios y sin valores reales.

## Fotos de producto

### Proceso (`App\Images\ImageProcessor`, GD)

1. **Entrada:** el archivo ya validado (`ProductImageRequest`, ver más abajo).
2. **Decodificar:** con `imagecreatefromstring`. Si falla, se responde 422: el archivo no es la imagen que dice ser.
3. **Orientación:** en JPEG, `exif_read_data` da la orientación (valores 1 a 8) y se aplica con rotación o espejo. Es una defensa por si la app no la aplicó al reducir (A-85).
4. **Redimensionar:** sin agrandar.
   - **Grande:** cabe en 800×800 y mantiene la proporción.
   - **Miniatura:** cabe en 200×200; la pantalla la recorta con `object-fit`.
5. **Codificar:** en WebP con calidad 80, conservando el canal alfa. Volver a codificar quita todo EXIF, GPS y metadato.

Medida esperada: unos 50 a 100 KB la grande y 10 a 15 KB la miniatura. Para CE-002, 25 miniaturas suman menos de 400 KB.

### Claves y respuesta

- **Columna:** `products.image_key` guarda un UUID v4.
- **Objetos:** `<uuid>.webp` y `<uuid>-thumb.webp`, bajo `AWS_ROOT`.
- **`ProductResource`:** suma `image: { url, thumb_url } | null` con `Storage::disk(images_disk)->url(...)`. Lo usan la lista de «Productos», el catálogo de «Vender» y la ficha, que leen las mismas rutas (`GET /products`).
- **Sin datos en la dirección:** no lleva RUC, id ni nombre. Una prueba lo comprueba con una expresión regular.

### Ciclo de vida (`App\Images\ProductImageService`)

- **Subir o cambiar:**
  1. Procesa y sube los dos objetos nuevos, **fuera** de la transacción.
  2. En la transacción: bloquea la fila del producto (`lockForUpdate`), toma la clave anterior, guarda la nueva y registra la auditoría.
  3. Después del commit, borra los objetos anteriores.
- **Quitar:** igual, sin subir nada.
- **Si un borrado falla, o falla la transacción después de subir:** las claves van a `image_deletions`. `images:cleanup` las reintenta cada día a las 02:45, y deja de intentar a los 10 intentos con un aviso en el log.
  - Así no quedan huérfanas sin necesidad de listar el bucket.
  - Por eso la clave de la app no necesita el permiso de listar (`s3:ListBucket`).
- **Subidas simultáneas:** el bloqueo las ordena. La segunda borra la imagen de la primera, y queda una sola (caso límite de la spec).
- **Producto desactivado:** no se toca su foto (A-82).

## Logo de la empresa

- **Subir:** `CompanyService::updateLogo` pasa por `App\Images\LogoProcessor`.
  - **Validación:** la misma de hoy (PNG o JPEG, hasta 1 MB), más un máximo de 4000×4000.
  - **Proceso:** se decodifica, se reduce a 800 px de ancho si es más grande y se guarda como **PNG con alfa** (la transparencia se conserva). Volver a codificar quita los metadatos.
  - **Dónde:** se guarda como `<uuid>.png` en el disco de imágenes, con el mismo ciclo de vida (borrado posterior e `image_deletions`).
- **Lectura** (`App\Images\CompanyLogo`):
  - `url()`: una clave sin `/` está en el disco de imágenes. Una ruta antigua (`companies/{id}/…`) sigue en `public` hasta moverla, así nunca se ve un logo roto durante el despliegue.
  - `dataUri()`, para los PDF: `Cache::remember('company-logo:'.sha1($key), 1 día)`. La clave cambia con cada logo, así que la caché nunca queda vieja. Si S3 falla, registra un aviso y devuelve `null`: el PDF sale sin logo (RF-014).
- **`DocumentPdf`:** usa `CompanyLogo::dataUri()` en lugar de leer el disco `public`. **`TicketPdf` y la impresión térmica no cambian:** no llevan logo (A-77).
- **Logos existentes:** `php artisan images:move-logos`.
  - Para cada empresa con una ruta antigua: la lee de `public`, la procesa, la sube, actualiza `logo_path` y borra el archivo local.
  - Es idempotente y sigue con las demás si una falla (código de salida distinto de 0 y aviso).
  - `deploy/release.sh` lo corre después de `migrate`.
- **Copias de la spec 009:** siguen incluyendo `storage/app/public`. Ya no tendrá logos, pero no estorba. Las imágenes de S3 no entran en las copias (A-82): las protege el versionado.

## Contrato de API

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | `/api/v1/products/{product}/image` | Administrador | `multipart/form-data` con `image`: JPEG, PNG o WebP, hasta 2 MB y 4000×4000. Responde `200` con el producto (`image` lleno) |
| DELETE | `/api/v1/products/{product}/image` | Administrador | Quita la foto. Responde `200` con el producto (`image: null`). Si no tenía foto, también `200` (idempotente) |
| GET | `/products`, `/products/{product}` | Todos | Suman `image: { url, thumb_url } \| null` |
| POST | `/api/v1/company/logo` | Administrador | La misma ruta. Ahora se documenta como `multipart` y suma la validación de dimensiones |

- **Errores:** `422` si el formato, el tamaño, las dimensiones o el contenido no son válidos. `503` con un mensaje en español si S3 no responde («No se pudo guardar la foto. Inténtalo de nuevo»), y el producto no cambia. `403` para el vendedor y `404` para un producto de otra empresa.
- **Generador de OpenAPI:** cuando las reglas tienen `file` o `image`, el cuerpo pasa a `multipart/form-data` y `max` deja de documentarse como `maxLength` (en archivos son KB). Esto corrige también `company/logo`. Después, `php artisan openapi:generate` y `pnpm api:types`.

## Operación en AWS (RF-016, la hace el usuario)

1. **Bloqueo de acceso público del bucket:** permitir políticas públicas (`BlockPublicPolicy` y `RestrictPublicBuckets` en falso). Los ACL siguen desactivados.
2. **Política del bucket:** solo lectura pública de las imágenes, sin listado:
   ```json
   { "Effect": "Allow", "Principal": "*", "Action": "s3:GetObject", "Resource": "arn:aws:s3:::<bucket>/<AWS_ROOT>/*" }
   ```
3. **Clave IAM de la app:** `s3:PutObject`, `s3:GetObject` y `s3:DeleteObject` solo sobre `arn:aws:s3:::<bucket>/<AWS_ROOT>/*`.
4. **Clave IAM de las copias** (o un permiso aparte): `Put`, `Get` y `Delete` sobre `<AWS_BACKUP_ROOT>/*`, y `s3:ListBucket` con la condición `s3:prefix` = `<AWS_BACKUP_ROOT>/`.
5. **Versionado del bucket activado**, con una regla de ciclo de vida que borre las versiones antiguas a los 30 días (A-82).
6. **`.env`:** renombrar `AWS_BACKUP_BUCKET` → `AWS_BACKUP_ROOT`. Agregar `AWS_BACKUP_ACCESS_KEY_ID` y `AWS_BACKUP_SECRET_ACCESS_KEY` si las copias usan otra clave.

## Frontend

> Lo escribe la sesión del frontend. Rama `016-imagen-producto` del frontend, desde `main`. **Sin dependencias nuevas**, salvo que la prueba de Android (T-F1, más abajo) obligue a usar `@capacitor/camera`, que el usuario tendría que aprobar.

### Tipos y contrato

- **`features/inventory/types.ts`:** `ProductImage = { url: string; thumb_url: string }` y `Product.image: ProductImage | null`. Se tipa a mano, porque OpenAPI no documenta las respuestas.
- **`features/sales/api.ts`:** `CatalogProduct = SellableProduct & { image: ProductImage | null }`, solo para el catálogo. `SellableProduct` no cambia.
- **Cuerpos de las subidas:** salen de `pnpm api:types` después de que la API corrija el generador. Hasta entonces no se tipa nada a mano.

### `core/api/upload.ts` (nuevo)

`fetch` no informa el avance de una subida, y HU-1 (esc. 8) lo pide. Por eso se agrega `uploadForm(path, form, { onProgress, signal })`:

- **Cómo:** usa `XMLHttpRequest`, con las mismas cabeceras que `authMiddleware` (token, `Accept` y `X-App-Version`).
- **Mismas reacciones que el cliente:** con 401 llama a `clearToken`, y con 426 avisa en `updateRequired`.
- **Errores:** devuelve `ApiError`, el mismo que `unwrap`, así que `useApiForm` y `fieldError('image')` funcionan igual.
- **Cancelar:** el `AbortSignal` corta la subida y no muestra error.
- **Para no duplicar:** la parte común de las cabeceras se separa en una función que usan el *middleware* y `upload.ts`.

### Reducir antes de subir (`shared/images/prepare.ts`, A-85 y RF-012)

`prepareProductPhoto(file): Promise<File>`, en este orden:

1. **Formato:** se leen los primeros bytes (*magic bytes*), sin confiar en la extensión. Solo pasan JPEG, PNG o WebP. HEIC y lo demás se rechazan con «Formato no admitido. Usa una foto JPG, PNG o WebP» (HU-3, esc. 6).
2. **Peso:** más de 10 MB se rechaza sin abrir el archivo, con un mensaje que sugiere tomar la foto en modo normal (A-80).
3. **Dimensiones:** se leen en la cabecera (SOF de JPEG, IHDR de PNG, VP8/VP8L/VP8X de WebP). Más de 50 MP se rechaza con el mismo mensaje del paso 2. Así no se decodifica algo que pueda agotar la memoria, aunque pese menos de 10 MB.
4. **Decodificar:** `createImageBitmap(file, { imageOrientation: 'from-image' })`, que gira según el EXIF.
5. **Reducir y codificar:**
   - se dibuja en un canvas de 1600 px como máximo de lado, sin agrandar;
   - fondo blanco para los PNG con transparencia (es una foto de catálogo, no un logo);
   - `toBlob('image/jpeg', 0.85)`;
   - si pasa de 2 MB, se intenta otra vez con calidad 0.7; si sigue pasando, se rechaza.
6. **Resultado:** un `File` JPEG sin EXIF ni GPS, porque el canvas no los copia. Pesa unos 300 a 600 KB.
7. **Si falla la decodificación:** «No se pudo leer la imagen».

**El logo no pasa por aquí** (HU-4, esc. 5): solo se valida el formato (PNG o JPEG) y 1 MB, antes de enviarlo como hoy.

### `core/device`: elegir la foto (RF-011)

Se agrega a `Device`:

```ts
/** Abre la cámara o la galería; null si el usuario cancela. */
pickImage(source: 'camera' | 'gallery'): Promise<File | null>
```

- **`web.ts`:** crea un `<input type="file" accept="image/jpeg,image/png,image/webp">` temporal. Con `camera`, le pone `capture="environment"`; en la computadora el navegador lo ignora y abre el selector de archivos.
- **`native.ts`:** en principio, lo mismo. El WebView de Capacitor (`BridgeWebChromeClient`) atiende `<input type="file">`: con `capture` abre la cámara, y sin `capture` abre el selector de fotos del sistema.
- **T-F1, prueba en el emulador antes de escribir pantallas.** Se comprueba:
  - que «Tomar foto» abre la cámara y «Elegir de la galería» abre el selector de fotos de Android 13+, sin permiso de almacenamiento;
  - si hace falta `android.permission.CAMERA`. Android no lo pide para la cámara del sistema cuando la app **no** lo declara en el manifiesto, y en ese caso no hay permiso que negar (HU-3, esc. 4). Si Capacitor lo exige, se declara y se detecta la negativa;
  - qué pasa con «Atrás» dentro de la cámara y del selector.
- **Si la prueba falla** (no se puede elegir el origen, o no se detecta el permiso negado): se usa `@capacitor/camera` 8.x en `native.ts`, siempre detrás de `pickImage`. Se instala con pnpm, respeta `minimumReleaseAge`, se revisa y se pide la aprobación del usuario. El resultado de T-F1 queda en `verificacion.md`.
- **Ninguna feature** importa plugins ni crea el `<input>` por su cuenta.

### `shared/ui`

| Componente | Papel |
|---|---|
| `ProductThumb.vue` | Props `image`, `name` y `size` (`sm` de 48 px para la lista, `md` de 64 px para el catálogo, `lg` para la ficha). Caja de tamaño fijo con `aspect-square`, así el espacio queda reservado y nada se mueve (RF-013). `<img loading="lazy" decoding="async" :alt="name">` con `object-cover`. Sin foto, o si falla `@error`, muestra un marcador: un ícono sobre `bg-subtle`, con `aria-hidden` (lector de pantalla). `sm` y `md` usan `thumb_url`; `lg` usa `url` |
| `ImagePicker.vue` | Vista previa (`URL.createObjectURL`, liberada al cambiar o desmontar), «Agregar foto», «Cambiar foto» y «Quitar foto». En la app, «Agregar foto» abre un `BaseDialog` con «Tomar foto» y «Elegir de la galería» (HU-3, esc. 1); `useDismissable` cierra el diálogo con «Atrás» (esc. 5). En la web abre directo el selector. Emite el `File` ya preparado, o el mensaje de error de `prepareProductPhoto`. No sube nada |

Los colores salen solo del tema y el ícono usa `text-ink-muted`. Lo comprueba `theme.spec.ts`.

### Pantallas

| Dónde | Cambio |
|---|---|
| `inventory/api.ts` | `uploadImage(id, file, opts)` con `uploadForm` y `removeImage(id)` con `apiClient.DELETE`. Las dos devuelven el `Product` |
| `ProductFormView.vue` | Sección «Foto» con `ImagePicker`, solo para el administrador. **Al crear:** primero se guarda el producto y después se sube la foto. Si la subida falla, se va a la ficha con el aviso «El producto se guardó, pero la foto no se pudo subir» y «Reintentar» (HU-1, esc. 7). **Al editar:** la foto se sube o se quita aparte de «Guardar», con su propio botón, su avance y «Cancelar». Mientras sube, el botón queda deshabilitado (esc. 8). Sin conexión (`onConnectionChange`): se guardan los datos y se avisa que la foto necesita conexión |
| `ProductDetailView.vue` | `ProductThumb` `lg`. El administrador ve «Cambiar foto» y «Quitar foto»; quitarla pide confirmación con `BaseDialog` (HU-1, esc. 3). El vendedor solo ve la foto (esc. 5). Los errores 422 y 503 se muestran con el mensaje de la API |
| `ProductsView.vue` | `ProductThumb` `sm` en cada fila o tarjeta, en un grid `grid-cols-1` para que el nombre truncado no desborde a 360 px |
| `ProductCatalog.vue` («Vender») | `ProductThumb` `md` a la izquierda del nombre, en las tarjetas y en el esqueleto de carga, con el mismo alto. El carrito no muestra fotos |
| `sale-draft.ts` | `add()` copia solo los campos de `SellableProduct`, sin `image`. Hoy guarda en `localStorage` el objeto completo que llega del catálogo (A-60). Así la venta en curso no guarda direcciones, aunque ahora sean fijas: una foto cambiada borra la anterior |
| `CompanyView.vue` | El logo usa `uploadForm` (con avance) y la validación previa de 1 MB y formato. `logo_url` sigue llegando igual |
| `package.json` | `version` 0.3.0 (sale el `versionCode` de Android) |

### Pruebas del frontend (Vitest)

| Requisito / escenario | Prueba |
|---|---|
| RF-012, A-80 | `prepare.spec.ts`: rechaza HEIC por los *magic bytes* aunque se llame `.jpg`, más de 10 MB sin decodificar y más de 50 MP por la cabecera. Un JPEG de 4000×3000 sale en 1600×1200, como `image/jpeg` y en menos de 2 MB (canvas simulado). Las cabeceras de JPEG, PNG y WebP se leen bien |
| HU-1 esc. 6–8 | `ImagePicker.spec.ts`: vista previa y descartar sin emitir. `ProductFormView.spec.ts`: crear con foto que falla → producto creado y aviso; doble envío bloqueado; «Cancelar» aborta sin mostrar error |
| HU-1 esc. 3, 5 | `ProductDetailView.spec.ts`: «Quitar foto» pide confirmación y llama a `DELETE`; el vendedor no ve los botones |
| HU-2, RF-013 | `ProductThumb.spec.ts`: sin foto o con `@error` → marcador con `aria-hidden`; `alt` igual al nombre; `loading="lazy"`; misma caja con foto y sin ella |
| HU-2 esc. 4 | `sale-draft.spec.ts`: lo que se guarda en `localStorage` no trae `image` |
| HU-3 esc. 1, 5 | `ImagePicker.spec.ts` con `device().isNative`: el diálogo ofrece las dos opciones y «Atrás» (`dismissTop`) lo cierra |
| `upload.ts` | Cabeceras, avance, 422 → `ApiError` con errores de campo, 401 → `clearToken`, 426 → `updateRequired`, y abortar sin error |
| Tema | `theme.spec.ts` sigue en verde con los componentes nuevos |

**Verificación manual del frontend** (además de los pasos 1 a 4 de la API):
- 360 px y modo oscuro en «Vender», «Productos» y la ficha;
- lector de pantalla (TalkBack) en el catálogo;
- en el emulador y en el teléfono real: cámara, galería, «Atrás» y modo avión con el formulario abierto;
- **CE-002** con las herramientas del WebView (`chrome://inspect`).

## Estrategia de pruebas

| Requisito / escenario | Prueba | Tipo |
|---|---|---|
| RF-008, A-81 | `ImageProcessorTest`: JPEG con EXIF de orientación 6 → sale girado (alto > ancho); con GPS → la salida no trae `Exif`; PNG con alfa → WebP con alfa; tamaños 800 y 200 sin agrandar las imágenes chicas. Las imágenes y el EXIF se arman en `tests/Support/TestImages` | Unitaria |
| RF-003, A-80 | Un texto con extensión `.jpg`, un GIF, 2,1 MB y 4001 px dan 422 con mensaje en español; JPEG, PNG y WebP válidos dan 200 | Feature |
| HU-1, RF-006 | Subir, cambiar y quitar: objetos nuevos en el disco simulado, los anteriores borrados, `image` en la respuesta. `DELETE` sin foto da 200 | Feature |
| RF-002 | El vendedor recibe 403 en `POST` y `DELETE`, y ve `image` en `GET`. Otra empresa: 404 por `TenantRoutes.php` | Feature |
| RF-005, CE-003 | Las direcciones cumplen `/^…\/[0-9a-f-]{36}(-thumb)?\.webp$/` y no llevan RUC, id ni nombre. `CrossTenantAccessTest`: un producto de B con foto y su clave entre los `bMarkers`, así que ninguna respuesta de A la trae | Feature |
| RF-009 | Auditoría `product.image_updated` y `product.image_removed` | Feature |
| RF-010 | Con un disco que lanza excepciones: `POST` da 503 y el producto queda igual; `GET /products` sigue funcionando; si falla el borrado de la anterior, la nueva queda y la anterior va a `image_deletions` | Feature |
| Huérfanas | `images:cleanup` borra lo pendiente, cuenta los intentos y se rinde a los 10 | Feature |
| Simultáneas | Dos subidas seguidas al mismo producto: queda una imagen y no queda ningún objeto huérfano en el disco | Integration (PostgreSQL) |
| HU-4 esc. 1, 5 | Logo: PNG con transparencia → PNG con alfa en S3; el anterior se borra; `logo_url` desde el disco de imágenes | Feature |
| HU-4 esc. 2 | `images:move-logos`: mueve los logos antiguos, es idempotente y sigue si uno falla | Feature |
| HU-4 esc. 3, 4 | PDF: lleva el logo; tras borrarlo del disco, el segundo PDF lo sigue llevando (caché); con un disco que falla, el PDF sale sin logo y responde 200 | Feature |
| Contrato | `tests/Feature/Contract`: forma de `image`. El generador documenta `multipart/form-data` en las dos rutas con archivo | Feature |
| Spec 015 | `PrintParityTest` y las pruebas de PDF siguen en verde | Suites |
| Suites | SQLite, PostgreSQL y Pint; frontend completo | Suites |

## Verificación manual (antes de cerrar)

Con el bucket de pruebas del usuario (`IMAGES_DISK=images`):

1. Subir una foto desde la web y otra desde el celular. Las dos direcciones abren en el navegador sin sesión, y el celular las muestra en «Vender».
2. **CE-001:** cronometrar la subida de una foto normal del celular con datos móviles (menos de 10 s).
3. **CE-002:** medir con las herramientas del navegador lo que pesa la primera página de «Vender» con 25 fotos (menos de 1 MB).
4. **CE-004:** una foto vertical del celular se ve derecha. Descargar la grande y comprobar que no trae EXIF ni GPS (`exiftool`).
5. **RF-016:**
   - listar el bucket sin credenciales da `AccessDenied`;
   - con la clave de la app, leer o escribir en `AWS_BACKUP_ROOT` da `AccessDenied`;
   - `backup:run` escribe en `AWS_BACKUP_ROOT`;
   - el versionado está activo.
6. Logo: `images:move-logos` con un logo local real, «Mi empresa» y PDF de una boleta con el logo; tiempo del PDF antes y después.

## Riesgos

| Riesgo | Mitigación |
|---|---|
| Una política de AWS mal escrita expone las copias | Copias cifradas (spec 009), lectura pública solo de `AWS_ROOT/*`, claves separadas, y comprobación en la verificación (paso 5) |
| GD agota la memoria con una imagen enorme | Validación de 4000×4000 y 2 MB antes de decodificar. La app ya reduce a unos 1 600 px |
| La orientación sale mal (el navegador la aplica y el servidor la aplica otra vez) | El canvas al reducir ya entrega los píxeles derechos, y al volver a codificar no queda EXIF que el servidor pueda aplicar dos veces. Se prueba con fotos reales en el paso 4 |
| S3 lento hace lentos los PDF | Caché del logo de un día, con la clave como versión. Si falla, PDF sin logo |
| Renombrar `AWS_BACKUP_BUCKET` rompe las copias en local | Aviso explícito al usuario al implementar. Aún no hay copias en producción |
| La caché en base de datos con logos de hasta 1 MB | Unos 1,4 MB por empresa en base64, solo de las que generan PDF. Aceptable; si crece, se mueve la caché a archivos |
| Claves en `image_deletions` que nunca se borran | Aviso en el log a los 10 intentos. `ops:check` (spec 009) puede contarlas más adelante |
