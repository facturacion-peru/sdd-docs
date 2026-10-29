# Spec 016: Imagen del producto y logo en S3

**Estado:** Aprobada (2026-10-29) · **Versión:** 1.0 · **Fecha:** 2026-10-29
**Depende de:** 001, 002, 005, 009, 012, 013 · **Aclaraciones:** A-79 a A-85 (resueltas)

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que cada producto pueda tener **una foto opcional**, para que quien vende lo reconozca de un vistazo en el catálogo de «Vender» y en la lista de productos, sobre todo en el celular. Las imágenes, y también el logo de la empresa, se guardan fuera del servidor, en S3, como pidió el usuario (A-79).

## Historias de usuario

### HU-1 · Poner, cambiar o quitar la foto de un producto (P1)

Como administrador de empresa, quiero subir una foto al crear o editar un producto, cambiarla o quitarla, para que mi catálogo se vea como mis productos reales.

**Por qué esta prioridad:** sin poder cargar fotos, el resto no sirve.
**Prueba independiente:** subo una foto a un producto, aparece en su ficha, la cambio por otra y luego la quito; el producto vuelve a mostrarse sin foto.

**Escenarios de aceptación**

1. **Dado** un producto sin foto, **cuando** subo una imagen válida (A-80), **entonces** el producto la muestra en su ficha y en la lista.
2. **Dado** un producto con foto, **cuando** subo otra, **entonces** se ve la nueva y la anterior deja de guardarse (A-82).
3. **Dado** un producto con foto, **cuando** pulso «Quitar foto» y confirmo, **entonces** el producto queda sin foto y la imagen deja de guardarse.
4. **Dado** un archivo que no es una imagen admitida, o que no se puede reducir o sigue siendo demasiado grande después de reducirlo (A-85), **cuando** lo subo, **entonces** veo un mensaje claro con el formato y el tamaño permitidos, y el producto no cambia.
5. **Dado** que soy vendedor, **cuando** veo un producto, **entonces** veo su foto pero no puedo cambiarla ni quitarla (spec 002, RF-020).
6. **Dado** que elegí una imagen, **cuando** aún no guardé, **entonces** veo una vista previa y puedo descartarla sin que el producto cambie.
7. **Dado** que creo un producto con foto, **cuando** el producto se guarda pero la foto falla, **entonces** el producto queda creado, veo «La foto no se pudo subir» y puedo reintentar desde su ficha.
8. **Dado** una subida lenta, **cuando** está en curso, **entonces** veo el avance, no puedo enviarla dos veces y puedo cancelarla.

### HU-2 · Ver las fotos al vender y en el catálogo (P1)

Como vendedor, quiero ver la foto de cada producto en «Vender» y en la lista de productos para elegir el correcto más rápido.

**Por qué esta prioridad:** es el beneficio principal: menos errores y ventas más rápidas en el celular.
**Prueba independiente:** con 25 productos con foto, abro «Vender» en el celular con datos móviles y las fotos aparecen sin frenar la búsqueda ni el cobro.

**Escenarios de aceptación**

1. **Dado** productos con y sin foto, **cuando** abro el catálogo de «Vender» o la lista de «Productos», **entonces** cada producto con foto la muestra en miniatura y los demás muestran un marcador neutro, con el mismo tamaño para no mover la lista. El carrito, las alertas y los movimientos no muestran fotos.
2. **Dado** que las fotos tardan o no cargan (conexión lenta o almacenamiento caído), **cuando** busco y cobro, **entonces** puedo seguir vendiendo; la foto que falla muestra el marcador.
3. **Dado** un producto, **cuando** abro su ficha, **entonces** veo la foto en tamaño grande.
4. **Dado** una venta en curso guardada (spec 012), **cuando** vuelvo a «Vender» horas después, **entonces** las fotos se ven o aparece el marcador, nunca una imagen rota.

### HU-3 · Tomar la foto con el celular (P2)

Como administrador que usa la app Android, quiero tomar la foto del producto con la cámara o elegirla de la galería, sin pasar por la computadora.

**Por qué esta prioridad:** muchas bodegas cargan su catálogo desde el celular; sin esto, la foto se vuelve un paso de escritorio.
**Prueba independiente:** en el teléfono, tomo una foto desde la ficha del producto y aparece bien orientada en el catálogo.

**Escenarios de aceptación**

1. **Dado** la app Android, **cuando** pulso «Agregar foto», **entonces** puedo elegir entre «Tomar foto» y «Elegir de la galería».
2. **Dado** una foto tomada con el celular en vertical, **cuando** se guarda, **entonces** se ve derecha en todas las pantallas.
3. **Dado** una foto con la ubicación GPS u otros datos del teléfono, **cuando** se guarda, **entonces** esos datos se eliminan (A-81).
4. **Dado** que niego el permiso de cámara, **cuando** quiero tomar la foto, **entonces** veo cómo darlo y puedo elegir de la galería igualmente.
5. **Dado** que estoy eligiendo la foto, **cuando** pulso «Atrás» en Android, **entonces** se cierra el selector y no la pantalla.
6. **Dado** una foto de la galería en un formato no admitido (A-80), **cuando** la elijo, **entonces** veo el mensaje de formato enseguida, sin esperar a la red.

### HU-4 · El logo de la empresa también en S3 (P2)

Como administrador de empresa, quiero que mi logo se guarde en el mismo almacenamiento que las fotos, sin notar ningún cambio al usarlo.

**Por qué esta prioridad:** el usuario pidió todas las imágenes en S3 (A-79). Para quien usa la app no cambia nada, pero deja de depender del disco del servidor.
**Prueba independiente:** subo un logo, aparece en «Mi empresa» y en el PDF de una boleta, igual que antes.

**Escenarios de aceptación**

1. **Dado** que subo o cambio el logo, **cuando** se guarda, **entonces** se ve en «Mi empresa» y en los PDF, como hoy, y el anterior deja de guardarse.
2. **Dado** una empresa que ya tenía logo antes de esta spec, **cuando** se instala la nueva versión, **entonces** su logo pasa a S3 y se sigue viendo, sin que la empresa haga nada.
3. **Dado** que genero varios PDF seguidos, **cuando** llevan logo, **entonces** no tardan más que hoy de forma notoria.
4. **Dado** que el almacenamiento no responde, **cuando** genero un PDF, **entonces** el PDF sale igual, sin logo, y no se pierde la descarga.
5. **Dado** un logo con fondo transparente, **cuando** se guarda, **entonces** se sigue viendo transparente en «Mi empresa» y en los PDF. El logo no pasa por la reducción de las fotos: solo se valida.

## Casos límite

- **Almacenamiento caído al subir:** se avisa y el producto conserva su foto anterior (o ninguna). Guardar los demás datos del producto no depende de la foto.
- **Almacenamiento caído al mostrar:** se ve el marcador; vender, buscar y cobrar siguen funcionando.
- **Otra empresa:** una empresa no puede descubrir ni listar las fotos de otra. Las fotos son públicas para quien tenga su dirección, y esa dirección no se puede adivinar ni deducir del RUC, el producto o la empresa (A-79). El usuario acepta ese costo para fotos de catálogo.
- **Producto desactivado:** conserva su foto (spec 002: se desactiva, no se borra).
- **Foto enorme del celular** (por ejemplo 12 MP): se acepta si está dentro del límite y se guarda en un tamaño adecuado para la pantalla (A-81).
- **Subidas simultáneas** de dos administradores al mismo producto: queda una sola foto, la última, y no quedan imágenes huérfanas guardadas.
- **Ventas pasadas:** tickets, comprobantes, PDF e impresión térmica no llevan la foto del producto. La impresión térmica sigue sin logo (A-77).
- **Exportar e importar (spec 014):** según A-83.
- **Sin conexión en la app:** se guardan los datos del producto y se avisa que la foto necesita conexión.
- **Pantallas angostas y modo oscuro:** a 360 px, la miniatura y el marcador no desbordan la tarjeta del catálogo y se ven bien en los dos temas.
- **Lector de pantalla:** la foto se anuncia con el nombre del producto, y el marcador no se lee.
- **Foto pesada** (3 a 10 MB, del celular o de la computadora): la app, en la web y en Android, la reduce antes de subirla (A-85); el servidor la valida y la procesa igual.
- **Foto de más de 10 MB** (modo de máxima resolución): la app la rechaza antes de abrirla, porque reducirla podría agotar la memoria de un celular de gama baja, y sugiere tomarla en modo normal.
- **Copias de seguridad en el mismo bucket:** las copias de la spec 009 van en otra carpeta del mismo bucket. La app no puede leerlas ni borrarlas, y lo público son solo las imágenes (A-79).

## Requisitos funcionales

- **RF-001:** Un producto PUEDE tener como máximo una foto; es opcional.
- **RF-002:** Solo el administrador de empresa DEBE poder poner, cambiar o quitar la foto; cualquier usuario de la empresa la ve.
- **RF-003:** El sistema DEBE aceptar solo los formatos y el tamaño de A-80 y rechazar lo demás con un mensaje claro.
- **RF-004:** Las fotos y el logo DEBEN guardarse en S3, en la carpeta de imágenes del bucket, separada de la carpeta de las copias de seguridad de la spec 009 (A-79).
- **RF-005:** Una empresa NO DEBE poder descubrir ni listar las fotos de otra. Las direcciones DEBEN ser imposibles de adivinar y no llevar RUC, ids ni nombres (A-79).
- **RF-006:** Al cambiar o quitar una foto, la anterior DEBE dejar de guardarse (A-82).
- **RF-007:** Las pantallas con listas DEBEN usar una versión reducida de la foto, para que el celular no descargue imágenes grandes (A-81).
- **RF-008:** El sistema DEBE eliminar los datos ocultos de las fotos (ubicación, modelo del teléfono) y respetar su orientación (A-81).
- **RF-009:** Poner, cambiar o quitar la foto DEBE quedar en la auditoría de la empresa (principio VIII).
- **RF-010:** Si el almacenamiento falla, vender y gestionar productos DEBE seguir funcionando; solo la foto queda pendiente o sin mostrarse.
- **RF-011:** En la app Android, la cámara y la galería DEBEN pasar por el módulo del dispositivo (principio VII).
- **RF-012:** La app (web y Android) DEBE comprobar el formato antes de subir y reducir la foto (A-85). La app acepta originales de hasta 10 MB (A-80) y rechaza enseguida, con un mensaje, los que pesen más (por ejemplo, una foto en el modo de máxima resolución del celular) o no pueda leer. A la API la foto ya llega reducida, y la API acepta hasta 2 MB. La API sigue siendo la que decide.
- **RF-013:** Las listas DEBEN cargar las fotos solo cuando se acercan a la pantalla y reservar su espacio, para que nada se mueva.
- **RF-014:** El logo de la empresa DEBE guardarse en S3 como las fotos, y los logos existentes DEBEN pasarse a S3 al instalar la versión. Los PDF DEBEN seguir mostrándolo sin volverse notoriamente más lentos, y salir sin logo si el almacenamiento no responde.
- **RF-015:** La subida DEBE pasar por la API, que valida el contenido real del archivo y lo procesa (A-81, A-85). Nadie sube directo al almacenamiento.
- **RF-016 (operación):** El acceso público DEBE limitarse a leer la carpeta de imágenes, sin listar el bucket. La clave de la app DEBE poder escribir solo en esa carpeta, las copias DEBEN usar otra clave o permiso, y el bucket DEBE tener versionado (A-79, A-82). Lo configura el responsable en AWS, y se comprueba en la verificación.

## Entidades clave

- **Foto del producto:** la imagen de un producto de una empresa, con su versión para listas y su versión grande. Se guarda en S3; el producto solo sabe si tiene foto y dónde está.
- **Logo de la empresa:** la imagen que va en «Mi empresa» y en los PDF. Pasa del disco del servidor a S3.

## Criterios de éxito

- **CE-001:** Subir una foto normal tomada con el celular (3 a 6 MB), con datos móviles, tarda **menos de 10 segundos** hasta verla en la ficha.
- **CE-002:** En «Vender» con 25 productos con foto, el celular descarga **menos de 1 MB** de imágenes para la primera página.
- **CE-003:** Una prueba automática confirma que ninguna empresa puede descubrir ni listar las fotos de otra: las respuestas de una empresa nunca traen direcciones de otra, y las direcciones no llevan RUC, ids ni nombres. En la verificación manual, listar el bucket sin credenciales falla.
- **CE-004:** Una foto vertical del celular se ve derecha y sin datos de ubicación.

## Fuera de alcance

- Varias fotos por producto o galerías.
- Editar la foto en la app (recortar, filtros), más allá de lo que haga el propio teléfono.
- Fotos en tickets, comprobantes, PDF o impresión térmica.
- Fotos en el carrito de «Vender», en las alertas de stock y en los movimientos de inventario.
- Buscar productos por foto o leer códigos de barras con la cámara.
- Direcciones firmadas o fotos privadas (A-79: son públicas con dirección imposible de adivinar).
- Subir directo del celular a S3 (A-85).
- Fotos en exportar e importar (A-83).
- Logo en la impresión térmica (A-77).

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-29 | Spec creada a pedido del usuario (foto opcional del producto guardada en S3) | — |
| 2026-10-29 | Sección de frontend aportada por la sesión del frontend | Escenarios HU-1 6–8, HU-2 4, HU-3 5–6, casos límite, RF-012 y RF-013, y A-85 |
| 2026-10-29 | A-79 | Mismo bucket, con carpeta de imágenes (`AWS_ROOT`) y carpeta de copias separadas. Imágenes públicas con dirección imposible de adivinar, sin URLs firmadas. El logo también pasa a S3 (HU-4, RF-014). Clave de la app limitada a la carpeta de imágenes y versionado (RF-016) |
| 2026-10-29 | A-80 a A-83 | Recomendaciones aceptadas: JPEG, PNG y WebP (tamaños corregidos en la v0.4); 800 y 200 px en WebP, girada y sin datos ocultos; borrado al cambiar o quitar, con limpieza de huérfanas; sin fotos en exportar e importar |
| 2026-10-29 | A-84 | Pruebas automáticas con almacenamiento simulado; el bucket de pruebas del usuario solo para la verificación manual |
| 2026-10-29 | A-85 | La app reduce la foto a unos 1 600 px en JPEG y la sube por la API (RF-015) |
| 2026-10-29 | v0.2 | Decisiones transmitidas por la sesión del frontend como del usuario; falta que el usuario apruebe la spec |
| 2026-10-29 | v0.3 | Revisión del frontend: RF-012 (se reducen originales de hasta unos 20 MB y el límite de 5 MB es para lo que llega a la API), reducción también en la web, logo transparente sin reducir (HU-4 esc. 5) y CE-001 |
| 2026-10-29 | v0.4 | A-80 corregida por el usuario: original hasta 10 MB en la app, hasta 2 MB en la API (ya reducida) y el logo hasta 1 MB, como hoy. CE-001 con una foto normal del celular |
| 2026-10-29 | Aprobada (v1.0) | El usuario aprueba la v0.4 sin cambios |
