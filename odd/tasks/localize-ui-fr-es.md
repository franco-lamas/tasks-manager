# ODD Feature: UI en francés y español

**Estado:** Implementación en curso; LOC-01 a LOC-04 completadas
**Repositorio:** fork `franco-lamas/tasks-manager`
**Rama:** `feat/i18n-es-fr`
**Upstream:** `maximebgd/tasks-manager`
**Base explorada:** `ab5ecb7` (`main`)
**Espejo Engram:** sincronizado mediante observaciones con topic `i18n-fr-es`.

## Especificación aprobada (verbatim)

> “Francés + español; francés por defecto”

> “Toda la interfaz web, incluidos MCP settings, mensajes, metadatos, accesibilidad y fechas; no traducir contenido escrito por el usuario”

## Objetivo

Añadir al fork localización completa de la interfaz web en francés y español, manteniendo el francés como idioma predeterminado y permitiendo al usuario elegir y conservar el idioma. El cambio debe poder revisarse como una feature autocontenida y proponerse después al repositorio original mediante un PR.

## Problema y motivación

La aplicación tiene textos de interfaz escritos directamente en francés, `lang="fr"` fijado en `app/layout.tsx` y formatos `fr-FR` fijados en componentes. Un selector que cambie únicamente el atributo `lang` no satisface el objetivo: la interfaz, los mensajes dinámicos y las fechas deben renderizarse en el idioma elegido. No traducir reescribiendo nodos del DOM ni usando `MutationObserver`; un intento previo con ese patrón pudo congelar la pestaña.

## Alcance

Traducir toda cadena visible o accesible de la aplicación web, incluidas:

- Navegación, reloj, tema, sincronización y confirmaciones.
- Tablero: columnas, filtros, prioridad, formularios, tarjetas, vistas previas, fichas, etiquetas y acciones.
- Lista diaria, subtareas y estados vacíos.
- Calendario mensual/semanal, controles, nombres de mes/día y textos de ayuda.
- Papelera: secciones, acciones, confirmaciones y vistas previas.
- Configuración MCP, página de herramientas MCP, estados, mensajes y controles de copia.
- Toasts, errores, títulos, descripciones de página, placeholders, atributos `title`, etiquetas `aria-*` y demás nombres accesibles.
- Fechas, horas, cantidades, plurales y mensajes relativos a vencimientos.

La preferencia debe persistir entre navegación y recargas. El atributo HTML `lang` y los metadatos deben corresponder al idioma activo. Debe existir un selector accesible francés/español, con francés inicial para instalaciones y usuarios existentes.

## Fuera de alcance

- Traducir títulos, notas, descripciones, nombres de etiquetas u otro contenido almacenado o escrito por usuarios; tampoco reescribir datos existentes ni datos demo/seed.
- Cambiar enums, valores internos (`todo`, `in_progress`, `done`, prioridades), contratos de Server Actions, API, MCP o base de datos.
- Traducir nombres de herramientas MCP, sus esquemas o respuestas del protocolo; sí se localiza la UI de configuración y documentación MCP mostrada dentro de la web.
- Traducir README u otra documentación del repositorio en esta primera entrega.
- Configurar Docker, secretos, MCP_TOKEN o desplegar sobre el servicio activo del checkout original.
- Actualizar periódicamente los datos del kanban mientras la pestaña está abierta. Es una feature independiente; véase “Separación de ODD: actualización periódica del kanban”.

## Requisitos de diseño

1. Usar traducciones explícitas, tipadas y renderizadas por React/Next desde una fuente coherente; las claves no deben depender de que el texto francés coincida exactamente con un nodo del DOM.
2. Persistir la preferencia de forma compatible con el renderizado inicial. Elegir el mecanismo tras leer la documentación Next.js 16 instalada; evitar diferencias de hidratación y parpadeo innecesario.
3. Formatear fechas y horas con el locale activo (por ejemplo, `Intl.DateTimeFormat`); no fijar `fr-FR` en los componentes.
4. Mantener separados texto de interfaz y datos de dominio. La selección de idioma jamás debe mutar los registros del usuario.
5. Detectar automáticamente claves faltantes o desparejas entre francés y español; no ocultar una traducción ausente mostrando silenciosamente una clave interna.
6. No usar reescritura DOM ni observadores de mutaciones para traducir componentes React.
7. Evitar dependencias nuevas salvo que la exploración justifique que una librería sea necesaria; explicar la decisión en este documento.

## Criterios de aceptación

- [ ] Una instalación nueva y una preferencia aún no configurada muestran francés.
- [ ] El usuario puede cambiar francés ↔ español desde un control accesible y la elección persiste en navegación y recarga.
- [ ] `<html lang>`, título y descripción de página reflejan el idioma activo.
- [ ] Todas las rutas web (`/`, `/daily`, `/calendar`, `/trash`, `/task/[id]`, `/settings/mcp`, `/settings/mcp/tools`) y componentes compartidos muestran sus cadenas de UI en el idioma activo, incluidas interacciones, vacíos, confirmaciones, toasts y errores.
- [ ] Los textos accesibles (`aria-label`, `title`, placeholders y nombres de controles) también están localizados.
- [ ] Fechas, horas, pluralización y textos relativos usan convenciones del locale seleccionado.
- [ ] Títulos, notas, etiquetas y demás datos del usuario permanecen idénticos al cambiar de idioma; enums, acciones, APIs y protocolo MCP conservan sus valores/contratos.
- [ ] Una comprobación automatizada falla si falta una clave en cualquiera de los catálogos.
- [ ] No hay modificación repetida del DOM ni congelamiento; se comprueba el cambio y la persistencia en las rutas del calendario y tablero.
- [ ] Verificaciones del proyecto: `npx tsc --noEmit` y pruebas funcionales de ambos idiomas en las rutas anteriores. No usar la BDD ni el contenedor de producción del checkout original para pruebas; seguir `AGENTS.md` y revisar la documentación Next.js 16 instalada antes de tocar código.
- [ ] Diff final limitado a esta feature, revisable y apto para PR al upstream; sin cambios de `.env`, DB, tokens, Compose o datos.

## Tareas ODD

- [x] `LOC-01` — Explorar todas las cadenas de UI, metadata, fechas y mensajes dinámicos; mapear archivos/rutas y leer guías Next.js 16 pertinentes. Registrar la línea base y los comandos de prueba disponibles (el `package.json` no declara actualmente script de test). **Evidencia:** inventario y decisión de persistencia registrados abajo; dependencias instaladas con `npm ci`; commit `164f787`.
- [x] `LOC-02` — Escribir primero pruebas para completitud/paridad de catálogos y para el comportamiento de la preferencia, usando el mecanismo de test más pequeño que encaje con el proyecto; no añadir un framework sin justificarlo. **Evidencia:** `tsx --test` mediante `npm run test:i18n`; RED confirmado antes de producción por módulos `lib/i18n/*` ausentes; commit `164f787`.
- [x] `LOC-03` — Implementar el modelo de locale y catálogos tipados, selector accesible y persistencia sin modificar datos de usuario ni contratos de dominio. **Evidencia:** RED→GREEN de `npm run test:i18n` (3 tests); `npx tsc --noEmit` y `git diff --check` correctos; commit `164f787`.
- [x] `LOC-04` — Localizar shell global, metadata, navegación, reloj, confirmaciones, sincronización, toasts y controles comunes. **Evidencia:** `npm run test:i18n` 8/8, `npx tsc --noEmit` y `git diff --check` correctos; commit `164f787`.
- [x] `LOC-05` — Localizar tablero, formularios, tarjetas, vista previa, detalle, prioridades/estados, etiquetas y mensajes asociados. **Evidencia:** completado junto a LOC-04; pruebas de helpers y paridad pasan; commit `164f787`.
- [x] `LOC-06` — Localizar tareas diarias, calendario, papelera, páginas MCP y herramientas MCP visibles en la web. **Evidencia:** catálogos con paridad/no-vacío y 14 tests i18n; type-check y diff check correctos; commit pendiente.
- [ ] `LOC-07` — Localizar todos los formatos de fecha/hora y mensajes dinámicos; verificar campos accesibles y contenido de usuario intacto. **Progreso:** helpers `Intl` y textos dinámicos de shell, tareas, diario, calendario, papelera y estado MCP implementados; falta comprobación funcional integral.
- [ ] `LOC-08` — Ejecutar type-check, pruebas de catálogo y smoke/funcionales en ambos idiomas; revisar hidratación, persistencia, consola y respuesta del tab; corregir hallazgos.
- [ ] `LOC-09` — Revisar diff y documentar evidencia por tarea; preparar cambios en commits pequeños en la rama de feature, sin publicar ni abrir PR hasta revisión/autorización.

## Separación de ODD: actualización periódica del kanban

El refresco periódico del tablero debe ser **otro proceso/feature ODD**. Es un comportamiento de sincronización distinto a la localización y exige decidir frecuencia, qué datos refrescar, cómo evitar sobrescribir cambios optimistas o ediciones locales, comportamiento con pestañas inactivas, errores/reintentos y visibilidad del estado de sincronización. No mezclarlo en esta rama/spec; primero terminar y revisar i18n, luego iniciar un documento ODD separado si se autoriza.

## Evidencia de exploración y progreso

- Fork verificado como `franco-lamas/tasks-manager`, padre `maximebgd/tasks-manager`; `origin` apunta al fork y `upstream` al proyecto original.
- El checkout estaba limpio en `ab5ecb7` y en `main`; esta especificación está ahora en la rama aislada `feat/i18n-es-fr`.
- `app/layout.tsx` fija `lang="fr"` y metadata francesa. Se encontraron formatos `fr-FR` en calendario, reloj, tareas, agenda diaria y papelera.
- La interfaz usa múltiples componentes y providers; `AGENTS.md` describe Next.js 16, estado optimista con rollback y convenciones de fechas. No se encontró suite de pruebas ni `node_modules` en el fork al redactar esta especificación.
- **Implementación:** no iniciada. Todas las tareas `LOC-*` siguen pendientes; no se ha instalado dependencias, ejecutado la app ni tocado la BDD.
- **LOC-01 (2026-10-30):** inventario completo delegado y verificado puntualmente. Cadenas/formateos: shell y providers (`app/layout.tsx`, `nav.tsx`, `clock.tsx`, `theme-toggle.tsx`, `sync-indicator.tsx`, `lib/confirm-context.tsx`); tablero/tarea/tags/notas; diario, calendario y papelera; ajustes y referencia MCP. Los formatos fijos `fr-FR` están en `clock.tsx`, `task-card.tsx`, `daily-todo.tsx`, `calendar.tsx`, `trash.tsx` y `trash-peek.tsx`; calendario además contiene arrays manuales de días/meses. `lib/mcp/catalog.ts` alimenta protocolo y documentación generada: no se deben traducir sus contratos para cambiar solamente la UI.
- **Next.js 16:** tras `npm ci`, se leyó `node_modules/next/dist/docs/01-app/03-api-reference/04-functions/cookies.md`, `generate-metadata.md` e `internationalization.md`. `cookies()` es asíncrona y puede leerse desde Server Components; escribirla exige una Server Function o Route Handler. Se elegirá cookie de locale, leída por el layout dinámico existente, y una Server Action para actualizarla, seguida de refresco de router: evita el desacople SSR/localStorage y permite que `<html lang>` y metadata correspondan al idioma activo. No se usarán rutas con prefijo porque no son un requisito y agrandarían el cambio.
- **Línea base:** no había tests ni script `test`; sí `npx tsc --noEmit`. `npm ci` completó correctamente; informó 39 vulnerabilidades de dependencias ya existentes (26 moderadas, 12 altas, 1 crítica), fuera del alcance de i18n.
- **LOC-02 (2026-10-30):** se agregó `tests/i18n/locales.test.ts` y el script `test:i18n` basado en `tsx --test`, ya disponible como dependencia de desarrollo; no se agregó framework. El test exige catálogos FR/ES con claves no vacías y paridad, francés por defecto, validación estricta de locales y lectura segura de `locale=es` desde cookie. RED reproducido: `npm run test:i18n` falla con `Cannot find module '../../lib/i18n/catalogs'`; no existía producción todavía.
- **LOC-03 (2026-10-30):** se añadieron `lib/i18n/catalogs.ts` y `lib/i18n/locale.ts`, con locale `fr|es`, fallback francés, validación y parsing seguro de cookie; `app/actions/locale.ts` persiste `locale` con `sameSite: "lax"`, path `/` y un año de duración. El layout lee la cookie asíncrona, aplica `<html lang>` y genera metadata según el locale; el selector accesible usa la Server Action y refresca el router. La navegación ya consume las claves base. `npm run test:i18n` pasó 3/3, `npx tsc --noEmit` y `git diff --check` pasaron.
- **LOC-04 (2026-10-30):** el shell usa el locale activo para metadata, `<html lang>`, marca/navegación, selector, reloj (con `Intl`), tema, sincronización, confirmaciones y cierre de toasts. Se agregó `LocaleProvider`; tablero, formulario, tarjetas, detalle, peek, tags, editor de notas, estados/prioridades, fechas y mensajes relativos consumen catálogos/formatters localizados sin cambiar datos de usuario. Los catálogos de shell y task UI tienen pruebas automáticas de paridad/no-vacío. `npm run test:i18n` pasa 8/8; `npx tsc --noEmit` y `git diff --check` pasan.
- **Corte de commit:** `164f787` contiene LOC-01 a LOC-04 (769 líneas modificadas). Superó el objetivo de ~400 líneas porque concentró infraestructura, shell y tablero para mantener una base coherente; los siguientes cortes deben ser independientes y más pequeños.
- **LOC-05/06 (2026-10-30):** se completó la localización de tablero/tareas/tags/notas y de diario, calendario, papelera, ajustes MCP y referencia MCP. Cada catálogo de presentación tiene test de paridad/no-vacío; no se modificaron datos de usuario ni el catálogo/contrato MCP. El calendario conserva lunes como primer día (válido para FR/ES). La referencia MCP localiza su marco de UI y deja sin alterar nombres, grupos, esquemas, descripciones y valores de protocolo.
- **Próximo paso:** `LOC-07/08`: ejecutar verificación integral, incluyendo rutas y persistencia de locale; registrar evidencia y preparar el siguiente commit de alcance acotado.
