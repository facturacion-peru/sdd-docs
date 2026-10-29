# Spec 016: Imagen del producto

**Estado:** Borrador · **Versión:** 0.1 · **Fecha:** 2026-10-29
**Depende de:** 002, 009, 012, 013 · **Aclaraciones:** A-79 a A-85 (abiertas)

> Qué y por qué. Sin frameworks, tablas, endpoints ni componentes: eso va en `plan.md`.

## Objetivo

Que cada producto pueda tener **una foto opcional**, para que quien vende lo reconozca de un vistazo en el catálogo de «Vender» y en la lista de productos, sobre todo en el celular. Las imágenes se guardan fuera del servidor, en un almacenamiento de objetos compatible con S3, como pidió el usuario.

## Historias de usuario

### HU-1 · Poner, cambiar o quitar la foto de un producto (P1)

Como administrador de empresa, quiero subir una foto al crear o editar un producto, cambiarla o quitarla, para que mi catálogo se vea como mis productos reales.

**Por qué esta prioridad:** sin poder cargar fotos, el resto no sirve.
**Prueba independiente:** subo una foto a un producto, aparece en su ficha, la cambio por otra y luego la quito; el producto vuelve a mostrarse sin foto.

**Escenarios de aceptación**

1. **Dado** un producto sin foto, **cuando** subo una imagen válida (A-80), **entonces** el producto la muestra en su ficha y en la lista.
2. **Dado** un producto con foto, **cuando** subo otra, **entonces** se ve la nueva y la anterior deja de guardarse (A-82).
3. **Dado** un producto con foto, **cuando** pulso «Quitar foto» y confirmo, **entonces** el producto queda sin foto y la imagen deja de guardarse.
4. **Dado** un archivo que no es una imagen admitida o es demasiado grande, **cuando** lo subo, **entonces** veo un mensaje claro con el formato y el tamaño permitidos, y el producto no cambia.
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

## Casos límite

- **Almacenamiento caído al subir:** se avisa y el producto conserva su foto anterior (o ninguna). Guardar los demás datos del producto no depende de la foto.
- **Almacenamiento caído al mostrar:** se ve el marcador; vender, buscar y cobrar siguen funcionando.
- **Otra empresa:** una empresa nunca puede ver, adivinar ni listar las fotos de otra (principio VIII).
- **Producto desactivado:** conserva su foto (spec 002: se desactiva, no se borra).
- **Foto enorme del celular** (por ejemplo 12 MP): se acepta si está dentro del límite y se guarda en un tamaño adecuado para la pantalla (A-81).
- **Subidas simultáneas** de dos administradores al mismo producto: queda una sola foto, la última, y no quedan imágenes huérfanas guardadas.
- **Ventas pasadas:** tickets, comprobantes, PDF e impresión térmica no llevan la foto del producto.
- **Exportar e importar (spec 014):** según A-83.
- **Sin conexión en la app:** se guardan los datos del producto y se avisa que la foto necesita conexión.
- **Pantallas angostas y modo oscuro:** a 360 px, la miniatura y el marcador no desbordan la tarjeta del catálogo y se ven bien en los dos temas.
- **Lector de pantalla:** la foto se anuncia con el nombre del producto, y el marcador no se lee.
- **Foto pesada en el celular** (4 a 8 MB): según A-85.

## Requisitos funcionales

- **RF-001:** Un producto PUEDE tener como máximo una foto; es opcional.
- **RF-002:** Solo el administrador de empresa DEBE poder poner, cambiar o quitar la foto; cualquier usuario de la empresa la ve.
- **RF-003:** El sistema DEBE aceptar solo los formatos y el tamaño de A-80 y rechazar lo demás con un mensaje claro.
- **RF-004:** Las fotos DEBEN guardarse en un almacenamiento de objetos compatible con S3, separado de las copias de seguridad de la spec 009 (A-79).
- **RF-005:** Una empresa NO DEBE poder acceder a las fotos de otra (A-79).
- **RF-006:** Al cambiar o quitar una foto, la anterior DEBE dejar de guardarse (A-82).
- **RF-007:** Las pantallas con listas DEBEN usar una versión reducida de la foto, para que el celular no descargue imágenes grandes (A-81).
- **RF-008:** El sistema DEBE eliminar los datos ocultos de las fotos (ubicación, modelo del teléfono) y respetar su orientación (A-81).
- **RF-009:** Poner, cambiar o quitar la foto DEBE quedar en la auditoría de la empresa (principio VIII).
- **RF-010:** Si el almacenamiento falla, vender y gestionar productos DEBE seguir funcionando; solo la foto queda pendiente o sin mostrarse.
- **RF-011:** En la app Android, la cámara y la galería DEBEN pasar por el módulo del dispositivo (principio VII).
- **RF-012:** La app DEBE comprobar el formato y el tamaño antes de subir, con el mismo límite de A-80. La API sigue siendo la que decide.
- **RF-013:** Las listas DEBEN cargar las fotos solo cuando se acercan a la pantalla y reservar su espacio, para que nada se mueva.

## Entidades clave

- **Foto del producto:** la imagen de un producto de una empresa, con su versión para listas y su versión grande. Se guarda fuera del servidor; el producto solo sabe si tiene foto y dónde está.

## Criterios de éxito

- **CE-001:** Subir una foto de 5 MB desde el celular con datos móviles tarda **menos de 10 segundos** hasta verla en la ficha.
- **CE-002:** En «Vender» con 25 productos con foto, el celular descarga **menos de 1 MB** de imágenes para la primera página.
- **CE-003:** Una prueba automática confirma que ninguna empresa obtiene la foto de otra, ni con su dirección exacta.
- **CE-004:** Una foto vertical del celular se ve derecha y sin datos de ubicación.

## Fuera de alcance

- Varias fotos por producto o galerías.
- Editar la foto en la app (recortar, filtros), más allá de lo que haga el propio teléfono.
- Fotos en tickets, comprobantes, PDF o impresión térmica.
- Fotos en el carrito de «Vender», en las alertas de stock y en los movimientos de inventario.
- Buscar productos por foto o leer códigos de barras con la cámara.
- Mover el logo de la empresa a S3, salvo que A-79 lo incluya.

## Clarificaciones

| Fecha | Aclaración | Decisión |
|---|---|---|
| 2026-10-29 | Spec creada a pedido del usuario (foto opcional del producto guardada en S3) | — |
| 2026-10-29 | Sección de frontend aportada por la sesión del frontend | Escenarios HU-1 6–8, HU-2 4, HU-3 5–6, casos límite, RF-012 y RF-013, y A-85 |
