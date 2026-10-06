# Tareas 013: Aplicación Android

**Estado:** Aprobadas (2026-10-06) · **Fecha:** 2026-10-06 · **Plan:** [plan.md](./plan.md)

> Cada tarea es pequeña, deja el proyecto funcionando y tiene su verificación.
> `[P]` = puede hacerse en paralelo con la anterior.

## Fase 1 · Preparación

- [x] **T001** Ramas `013-app-android` en la API y en el frontend, desde `main` — Verificación: `git branch --show-current` en ambos
- [x] **T002** JDK 21 (Temurin) en `~/Android/jdk-21…`, sin tocar el Java del sistema. Comprobar los requisitos de Capacitor 8 (JDK, `minSdk`, `compileSdk`, `targetSdk`) y las plataformas y herramientas que faltan en `~/Android/sdk` — Verificación: `java -version` con el JDK 21 y `sdkmanager --list_installed`
- [x] **T003** Revisar `@aparajita/capacitor-secure-storage` (código nativo de Android: Keystore, sin red ni datos fuera del dispositivo; licencia) y los scripts de instalación de los paquetes de Capacitor — Verificación: notas en `verificacion.md` antes de instalar

## Fase 2 · API (PDF del ticket y versión mínima)

- [x] **T004** Pruebas de `GET /tickets/{ticket}/pdf`: contenido de 80 mm con leyenda, vendedor ajeno sin acceso, otra empresa 404, ruta en `TenantRoutes.php` (deben fallar) — Verificación: `php artisan test --filter=TicketPdf` en rojo
- [x] **T005** PDF del ticket con dompdf (plantilla de 80 mm, como `DocumentPdf`) y su ruta — Verificación: T004 en verde
- [x] **T006** Pruebas del control de versión: sin cabecera pasa; menor que `APP_ANDROID_MIN_VERSION` responde 426 con `min_version`; igual o mayor pasa; sin mínimo configurado pasa; CORS admite `X-App-Version` (deben fallar) — Verificación: `php artisan test --filter=AppVersion` en rojo
- [x] **T007** *Middleware* de versión mínima, configuración y `.env.example`; `X-App-Version` en `config/cors.php` — Verificación: T006 en verde
- [x] **T008** `php artisan openapi:generate`; comprobaciones de la API (`php artisan test`, `composer test:pgsql`, `vendor/bin/pint --test`) — Verificación: códigos de salida 0

## Fase 3 · Módulo de dispositivo (frontend, sin Android aún)

- [x] **T009** `pnpm api:types`; `ticketsApi.pdf` y el tipo de la respuesta 426 — Verificación: `pnpm type-check`
- [x] **T010** Pruebas de `core/device` (web e implementación nativa con dobles) y de `token-storage` con caché en memoria e `initTokenStorage` (deben fallar) — Verificación: en rojo
- [x] **T011** Instalar `@capacitor/core`, `@capacitor/app`, `@capacitor/filesystem`, `@capacitor/share` y `@aparajita/capacitor-secure-storage` con `pnpm add` (aprobar scripts solo tras T003); `core/device/` y `token-storage` sobre `secureStore`; `initTokenStorage()` en `main.ts` — Verificación: T010 en verde y la suite completa sin cambios de comportamiento en la web

## Fase 4 · Comportamiento en la app (HU-1 a HU-5)

- [x] **T012** Pruebas en modo nativo simulado: descargas por `deliverFile`; «Imprimir» del ticket pide el PDF (sin `window.print`) en `SaleDoneDialog` y `TicketView` (deben fallar) — Verificación: en rojo
- [x] **T013** `download.ts` sobre `deliverFile`; impresión del ticket por PDF en la app — Verificación: T012 en verde
- [x] **T014** Pruebas del botón «Atrás» (cierra el diálogo superior, si no vuelve, en el Inicio o el login minimiza) (deben fallar) — Verificación: en rojo
- [x] **T015** Pila de diálogos abiertos en `shared/ui` (`BaseDialog`) y manejador de «Atrás» en `app/` — Verificación: T014 en verde
- [x] **T016** Pruebas: aviso sin conexión (`offline`/`online`); la cuenta de plataforma no entra en la app; 426 muestra «Actualiza la app» sin borrar sesión ni venta; `X-App-Version` solo en la app (deben fallar) — Verificación: en rojo
- [x] **T017** Aviso sin conexión en `AppLayout`; bloqueo de plataforma en la app; cabecera de versión y pantalla de actualización; márgenes `env(safe-area-inset-*)` — Verificación: T016 en verde

## Fase 5 · Proyecto Android

- [x] **T018** `@capacitor/cli` y `@capacitor/android`; `capacitor.config.ts` (nombre e identificador provisionales); `pnpm cap add android`; `version` 0.1.0 en `package.json` — Verificación: `./gradlew assembleDebug` compila
- [x] **T019** Íconos adaptativos (*vector drawable* desde `public/favicon.svg`), nombre visible, `allowBackup=false` y reglas de extracción, texto plano solo en `src/debug` — Verificación: `aapt dump` del APK de debug y del de release (sin texto plano en release)
- [x] **T020** Modos `android` y `android-dev` de Vite, scripts `android:dev` y `android:release`, firma con *keystore* fuera del repositorio (`keystore.properties` en `.gitignore`) y respaldo de la clave — Verificación: `pnpm android:release` genera el APK firmado (`apksigner verify`) y `git status` no muestra secretos
- [x] **T021** `CORS_ALLOWED_ORIGINS` local con el origen de la app; README del frontend: cómo compilar, instalar, firmar y conservar la clave — Verificación: la app de debug entra contra la API local por la red

## Fase 6 · Verificación en un teléfono real

- [ ] **T022** *(parcial: comportamiento verificado en el emulador, ver `verificacion.md`; falta el teléfono real)* CE-001 en un Android de gama baja: instalar, entrar, ticket y boleta (SUNAT beta), compartir el PDF de 80 mm, descargas, «Atrás», sin conexión, cuenta de plataforma, cerrar sesión, 426 con un mínimo alto. Más teclado, muesca y modo oscuro — Verificación: `verificacion.md` con capturas
- [x] **T023** *(CE-003 verificado en el emulador y en el manifiesto; CE-002 ~2,0 s en un Poco X3 NFC, gama media, aceptado por el usuario con ese margen)* CE-002 (apertura en menos de 3 s con sesión) y CE-003 (sin token en el almacenamiento web ni en una copia del teléfono) — Verificación: mediciones y comprobaciones en `verificacion.md`

## Cierre

- [x] **T024** Comprobaciones del principio VI en verde (frontend y API, SQLite y PostgreSQL, Pint) y OpenAPI y tipos al día — Verificación: códigos de salida 0
- [x] **T025** Bitácora y trampas nuevas en `AGENTS.md` (tabla de proyectos: APK), estado en `docs/specs/README.md`. Pendiente fuera de la spec: nombre e identificador definitivos (A-61) y entrega a empresas cuando exista el servidor de la 009 — Verificación: revisión
