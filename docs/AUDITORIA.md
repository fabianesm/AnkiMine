# Auditoría de AnkiMine — Fase 0

> Fecha: 2026-10-23 · Alcance: solo lectura (no se modificó código del repositorio).
> Convenciones: **[verificado]** = comprobado con herramientas (lectura de archivos, ejecución);
> **[deducido]** = se desprende de leer el código, sin prueba de extremo a extremo;
> **[sin verificar]** = no se pudo comprobar en este entorno.

---

## 0. Resumen ejecutivo

1. **Este repositorio no contiene el código de la extensión.** `EmaDevCol/AnkiMine` es un fork de
   `fabianesm/AnkiMine` y solo trae la **web de presentación** (HTML/Bootstrap): `index.html`,
   `english.html`, `privacy_*.html`, `css/`, `js/`, `assets/`. No tiene `LICENSE`, `README` ni
   `manifest.json`. [verificado]
2. El código de la extensión aparece en el **historial de git** como `assets/AnkiMine1.0.rar`
   (añadido en `c458fce`, 2023-06-08; borrado en `73d5c37`, 2023-08-08). Esta auditoría se hizo
   sobre ese paquete (v1.0, **Manifest V2**), extraído fuera del repo. [verificado]
3. La v1.0 es un derivado de **Online Dictionary Helper (ODH)** de Zhenyu "Ninja" Huang (licencia
   **MIT**). El autor de AnkiMine, **Fabian Soto**, declara la licencia **GPL-3.0-or-later** solo en
   `bg/legal.html`; **no incluye el aviso MIT de ODH**, que es obligatorio. [verificado]
4. La v1.0 **no carga en Chromium actual** (MV2 retirado): una extensión MV3 de control cargó y
   AnkiMine v1.0 no. [verificado]
5. **Todos sus diccionarios hacen scraping de diccionarios comerciales** (Cambridge, Collins,
   Larousse). Además, empaqueta `collins.json` (33 MB de Collins COBUILD EN→ZH), y ejecuta con
   `eval` **código remoto** descargado de GitHub o de URLs del usuario. Ambas cosas son incompatibles
   con las reglas 2 y 4. [verificado]
6. Seguridad: el popup es un `iframe.srcdoc` que **hereda el origen de la página anfitriona**
   (verificado en Chromium), con HTML externo sin escapar y `postMessage` sin validar el origen.
   Además, guarda la **contraseña de AnkiWeb en texto plano**.
7. Ninguno de los pares JA→ES y ES→JA existe; no hay soporte de japonés (la detección de palabras
   solo funciona con el alfabeto latino).
8. La versión publicada en Chrome Web Store es la **2.0.0** (julio de 2025, según un agregador de
   terceros) **[sin verificar]**. Su código **no está** en este repo.

---

## 1. Método y evidencias

| Paso | Comando / herramienta | Resultado |
|---|---|---|
| Inventario del repo | `git log --all --stat`, `find` | 16 commits, todos de Fabian Soto (2023-06-08 → 2025-07-16); solo la web |
| Recuperar la extensión | `git show c458fce:assets/AnkiMine1.0.rar` + `bsdtar -xf` (en el scratchpad, fuera del repo) | 94 archivos, 45 154 429 bytes sin comprimir |
| Lectura de código | lectura completa de `manifest.json`, `bg/**`, `fg/**`, `dict/**`, `_locales/**` | ver secciones 3–8 |
| Licencia de ODH | `curl https://raw.githubusercontent.com/ninja33/ODH/master/LICENSE` | MIT, "Copyright (c) 2018 Zhenyu Huang" |
| ODH actual | `curl …/ODH/master/src/manifest.json` | ODH ya migró a MV3 (v1.0.1, `service_worker` + `offscreen` + DOMPurify) |
| Aviso MIT en AnkiMine | `grep -rIl "Zhenyu\|MIT License\|Permission is hereby granted"` | sin coincidencias |
| Mazo incluido | lectura de solo lectura de `collection.anki21` del `.apkg` con `sqlite3` | tipo de nota y campos (sección 6) |
| Coste de los datos | `node` 22: `JSON.parse` de `collins.json` + `wordforms.json` | ≈1,0 s y ≈59 MB de heap (aproximación en Node, no medido en Chrome) |
| Popup `srcdoc` | Playwright + Chromium 153: una página crea un `iframe.srcdoc` como hace `fg/js/popup.js` | `iframe.origin == page.origin`; la página lee el contenido; un `onerror` dentro del iframe se ejecuta con el origen de la página |
| Carga MV2 | Playwright `--load-extension`: control MV3 vs. AnkiMine v1.0 | control: 1 service worker; AnkiMine: 0 páginas de fondo, no carga |
| CRX de la tienda / ToS de diccionarios | `curl` a `clients2.google.com`, WebFetch a cambridge.org y collinsdictionary.com | **bloqueado por la política de red del entorno** → [sin verificar] |

Rutas citadas como `bg/js/…`, `fg/js/…`, `dict/…` son relativas al paquete v1.0 extraído
(no existen en el árbol actual del repo).

---

## 2. Autoría y licencias

### 2.1 Quién es quién

| Componente | Autor | Licencia | Evidencia |
|---|---|---|---|
| Web de presentación (este repo) | Fabian Soto (`fabianesm`) | **Ninguna declarada** → por defecto, todos los derechos reservados | sin `LICENSE`; pie "Copyright © Fabian Soto" |
| Plantilla de la web | Start Bootstrap (Freelancer v7.0.7) | MIT | cabecera de `css/styles.css` y `js/scripts.js` |
| Bootstrap 5.2.3 | The Bootstrap Authors | MIT | cabecera de `css/styles.css`; JS por CDN |
| Font Awesome 6.3.0 Free (CDN) | Fonticons | íconos CC BY 4.0 / código MIT / fuentes OFL | `<script src="https://use.fontawesome.com/...">` |
| Montserrat, Lato (Google Fonts, CDN) | — | OFL | `<link href="https://fonts.googleapis.com/...">` |
| Extensión AnkiMine v1.0 | Fabian Soto (`"author": "Fabian Soto"`) | **GPL-3.0-or-later** (declarada solo en `bg/legal.html`) | `bg/legal.html`, `manifest.json` |
| Base de la v1.0: ODH | Zhenyu "Ninja" Huang | **MIT** | `bg/welcome.html` ("hemos utilizado el código de su extensión… como base"); LICENSE de ODH |
| jQuery 3.0.0 | OpenJS Foundation | MIT | `lib/jquery-3.0.0.min.js` |
| `bg/data/collins.json` | Contenido de Collins COBUILD (HarperCollins) | **Sin licencia de redistribución** | 37 379 entradas EN→ZH con `def_cn`/`ext_cn` |
| `bg/data/wordforms.json` | Desconocido (viene de ODH) | **Desconocida** | ODH lo excluye de su repo y remite a extraerlo de su CRX |
| `bg/sandbox/sign.js` | Desconocido | Desconocida | Algoritmo del token "tk" de Google Translate (no se usa) |
| `fg/Mazo-de-Prueba.apkg` | Fabian Soto + terceros | mixta | contiene audio de Cambridge, la fuente Gowun Dodum (OFL [sin verificar]) y un JPG de origen desconocido |

### 2.2 Qué permite y qué exige

- **GPL-3.0-or-later (código v1.0 de Fabian Soto):** permite usar, estudiar, modificar y
  redistribuir, incluso con fines comerciales. Exige:
  - conservar los avisos de copyright y de licencia;
  - entregar el código fuente correspondiente al distribuir (publicar en la Chrome Web Store
    cuenta como distribuir);
  - marcar los cambios con su fecha (§5a);
  - licenciar el derivado bajo la GPL-3.0 (o posterior);
  - no añadir restricciones adicionales.

  No ofrece garantía.
- **MIT (porciones de ODH):** permite todo, con la condición de **incluir el aviso de copyright y el
  de permiso** en todas las copias o porciones sustanciales. Es compatible con la GPL-3.0.
- **Consecuencia para el fork:**
  - el proyecto derivado debe distribuirse como **GPL-3.0-or-later**;
  - hay que incluir el **aviso MIT de ODH**, que hoy falta (incumplimiento heredado de la v1.0);
  - hay que acreditar a Fabian Soto y a Zhenyu Huang.
- **Ambigüedades a resolver:**
  - la GPL solo se declara en una página HTML del paquete, no en un archivo `LICENSE`;
  - la web no tiene licencia;
  - el código de las versiones 1.1 y 2.0.0 no está disponible y su licencia no está confirmada.

  Recomendación: pedir a Fabian Soto una confirmación por escrito (o que añada un `LICENSE` en
  `fabianesm/AnkiMine`). Esto no es asesoría legal.
- **Nombre y logo:** la GPL no otorga derechos sobre el nombre "AnkiMine" ni sobre el logo. Cambiar
  el nombre visible requiere tu aprobación (ver decisiones pendientes).

### 2.3 Créditos que hay que preservar (insumo para el README y la pantalla "Acerca de")
- Fabian Soto — autor de AnkiMine (https://github.com/fabianesm/AnkiMine).
- Zhenyu "Ninja" Huang — Online Dictionary Helper (https://github.com/ninja33/ODH), MIT.

---

## 3. Mapa de arquitectura actual

### 3.1 Repositorio actual (web)
```
index.html / english.html     landing ES/EN → enlace a la Chrome Web Store
privacy_es.html / privacy_en.html   política de privacidad (13-jul-2025)
css/styles.css                Bootstrap 5.2.3 + tema Freelancer (241 KB)
js/scripts.js                 navbar (tema Freelancer) + clase "active"
assets/                       logo, banderas, GIFs de demo
```

### 3.2 Extensión v1.0 (paquete del historial)

| Archivo | Responsabilidad |
|---|---|
| `manifest.json` | MV2; `background.page`; content scripts en `*://*/*` y `file://*/*`; permisos `webRequest`, `webRequestBlocking`, `storage`, `https://ankiweb.net/*`, `https://ankiuser.net/*`; comando `Alt+Q` (activar/desactivar) |
| `bg/background.html` | Página de fondo **persistente**: jQuery, clientes de Anki, datos, `iframe` sandbox, `backend.js` |
| `bg/js/backend.js` | `AKMBack`: hub de mensajes `api_*`, `formatNote`, elige el destino (AnkiConnect/AnkiWeb), carga los diccionarios en el sandbox, reproduce audio, sincroniza las opciones con todas las pestañas |
| `bg/js/ankiconnect.js` | Cliente de AnkiConnect (`$.ajax` POST `http://127.0.0.1:8765`, versión 6) |
| `bg/js/ankiweb.js` | Cliente de AnkiWeb por **scraping** + reescritura de User-Agent/Origin/Referer con `webRequestBlocking` |
| `bg/js/builtin.js` | Carga `data/collins.json` (ningún diccionario de AnkiMine lo usa) |
| `bg/js/deinflector.js` | Carga `data/wordforms.json` (las llamadas están comentadas en todos los diccionarios) |
| `bg/js/utils.js` | Valores por defecto, `optionsLoad`/`optionsSave`, `localizeHtmlPage` |
| `bg/js/agent.js` | RPC por `postMessage` con callbacks |
| `bg/sandbox/sandbox.{html,js}`, `api.js` | Descarga y **`eval`** los scripts de diccionario; despacha `findTerm` |
| `bg/sandbox/sign.js` | Token de Google Translate (sin uso) |
| `dict/*.js` | 9 diccionarios por scraping + `builtin_select` (marcador sin búsqueda) |
| `fg/js/frontend.js` | `AKMFront`: eventos (mousemove, dblclick, `selectionchange` con espera de 500 ms, tecla), arma la nota y genera el HTML del popup |
| `fg/js/popup.js` | Inyecta `<iframe id="akm-popup">` y pone el contenido en `srcdoc` |
| `fg/js/frame.js`, `spell.js` | Se ejecutan **dentro** del popup: botones "+"/audio, editor `contentEditable` de la oración |
| `fg/js/range.js`, `text.js` | Palabra bajo el cursor (solo alfabeto latino) y extracción de la oración de contexto (incluye PDF.js) |
| `fg/js/api.js` | Envoltorios de `chrome.runtime.sendMessage` |
| `bg/options.html` + `options.js`, `popup.html` + `popup.js` | Ajustes completos y rápidos (jQuery) |
| `bg/welcome.html`, `update.html`, `legal.html`, `instructions.html` | Páginas informativas (bienvenida, actualización, aviso legal, tutorial ES/EN) |
| `_locales/{es,en}` | 73 mensajes cada uno |
| `fg/Mazo-de-Prueba.apkg` | Mazo con el tipo de nota "Minado de Oraciones - AnkiMine" |

### 3.3 Flujo búsqueda → popup → Anki [verificado leyendo el código]
```
PÁGINA (content script, AKMFront)
 1. dblclick | selectionchange (+500 ms) | tecla (Shift por defecto) sobre texto → selectedText()
 2. chrome.runtime.sendMessage({action:'getTranslation'})
        │
FONDO (AKMBack, página persistente)
 3. agent.postMessage('findTerm') ──► IFRAME SANDBOX (Sandbox.backend_findTerm)
                                        │ diccionario activo, p. ej. enes_Cambridge.findTerm
 4.                                     ├─ api.fetch(url) ─► fondo api_Fetch: $.ajax GET cambridge.org…
 5.                                     └─ DOMParser + selectores CSS → [{expression, reading,
                                            definitions[HTML], audios[URL], css}]
        │ callback
PÁGINA
 6. buildNote() + getSentence(maxcontext); renderPopup() llama a isConnected() (AnkiConnect `version`, 100 ms)
 7. Popup.showNextTo() → iframe.srcdoc = HTML  (el iframe hereda el origen de la página)
POPUP (frame.js)
 8. clic en "+" → window.parent.postMessage({action:'addNote', nindex, dindex, context: innerHTML}, '*')
PÁGINA → FONDO api_addNote → formatNote(options) → target.addNote(note)
 9a. AnkiConnect: POST 127.0.0.1:8765 {action:'addNote', version:6, params:{note:{deckName, modelName,
     fields, tags, options:{allowDuplicate}, audio:{url, filename, fields}}}}
 9b. AnkiWeb: POST https://ankiuser.net/edit/save (sesión obtenida por scraping)
10. resultado → popup.sendMessage('setActionState') → ícono ✔/✘ durante 1 s
```

---

## 4. Manifest: MV2 → plan de migración a MV3

Hoy: **`manifest_version: 2`**, `minimum_chrome_version: "50.0.0.0"`. No carga en Chromium 153
[verificado]. El plan de migración (que encaja mejor como reescritura en la Fase 1/2 que como un
port línea por línea):

| Construcción MV2 en v1.0 | Problema en MV3 | Reemplazo propuesto |
|---|---|---|
| `background.page` persistente con DOM (jQuery, `Audio`, `DOMParser`, `iframe` sandbox) | No hay página de fondo; el service worker no tiene DOM y se termina en cualquier momento | `background.service_worker` + documento `offscreen` solo para lo que necesite DOM o audio; estado en `chrome.storage` (no en memoria) |
| `browser_action`, `chrome.browserAction.setBadgeText` | Renombrado | `action`, `chrome.action.setBadgeText` |
| `chrome.extension.getURL`, `chrome.extension.getBackgroundPage()` (las opciones llaman a `akmback()` directamente) | Eliminados | `chrome.runtime.getURL`; mensajería tipada entre la UI y el service worker |
| `webRequest` + `webRequestBlocking` (cabeceras de AnkiWeb) | El bloqueo no está disponible fuera de las instalaciones empresariales | Eliminar la integración con AnkiWeb (ver SEC-4) |
| `eval` de scripts remotos (`lib://` → raw.githubusercontent.com; URLs del usuario) | Va contra la política de código remoto de MV3 y de la tienda | Diccionarios como **datos** (formato Yomitan `.zip`) importados a IndexedDB |
| `web_accessible_resources` como array | Cambió el formato | `[{resources, matches}]`, mínimo necesario |
| Content scripts estáticos en `*://*/*` + `file://*/*` | Contradice la regla 4 (permisos mínimos) | `optional_host_permissions` + `chrome.scripting.registerContentScripts` por sitio / `activeTab` |
| `$.ajax` en el fondo | No hay jQuery ni XHR en el SW | `fetch` |
| `tabs.onUpdated` → enviar opciones a cada pestaña | Ruido y SW despierto | `chrome.storage.onChanged` en el content script |
| `update_url` en el manifest | Lo añade la tienda | Quitar del código fuente |
| CSP por defecto de MV2 | — | CSP por defecto de MV3; sin `unsafe-eval` |

Referencia útil: ODH ya publicó su port a MV3 (`service_worker` + `offscreen` + DOMPurify,
`minimum_chrome_version` 116) bajo MIT [verificado: manifest de ODH].

---

## 5. Fuentes de diccionario

### 5.1 Diccionarios de la v1.0

| Script | Par | Fuente | Método | Estatus de licencia |
|---|---|---|---|---|
| `enes_Cambridge` | EN→ES | dictionary.cambridge.org (english-spanish) | scraping HTML + audio MP3 | Comercial (Cambridge UP); **no abierto** ❌ |
| `esen_Collins` | ES→EN | collinsdictionary.com (spanish-english) | scraping + audio | Comercial (HarperCollins) ❌ |
| `fren_Cambridge` | FR→EN | dictionary.cambridge.org | scraping + audio | ❌ |
| `fren_Collins` | FR→EN | collinsdictionary.com | scraping + audio | ❌ |
| `fres_Larousse` | FR→ES | larousse.fr | scraping + audio | Comercial (Larousse) ❌ |
| `dees_Larousee` | DE→ES | larousse.com | scraping + audio | ❌ |
| `iten_Collins` | IT→EN | collinsdictionary.com | scraping + audio | ❌ |
| `kren_Collins` | KO→EN | collinsdictionary.com | scraping + audio | ❌ |
| `pten_Collins` | PT→EN | collinsdictionary.com | scraping + audio | ❌ |
| `builtin_select` | — | — | marcador sin búsqueda | ✔ (propio) |
| `lib://<nombre>` | varios | raw.githubusercontent.com/ninja33/ODH | **código remoto + `eval`** | MIT el código; las fuentes que raspa no son abiertas ❌ |
| `udfscripts` | cualquiera | URL arbitraria del usuario | **código remoto + `eval`** | desconocido ❌ |

**Cómo se consultan:** una petición GET por búsqueda a la página pública del diccionario (sin
caché), con un timeout de 3 s, analizada con `DOMParser` y selectores CSS frágiles. El audio
enlazado se reproduce desde el fondo y, al añadir la nota, **AnkiConnect lo descarga a la colección
del usuario**.

**Términos de servicio:** no pude leer los términos de Cambridge, Collins ni Larousse (la red del
entorno los bloquea) **[sin verificar]**. Aun así, ninguno tiene una licencia abierta y
redistribuible, así que **por la regla 2 deben retirarse**, junto con sus audios.

### 5.2 Datos empaquetados
- `bg/data/collins.json` (33,3 MB): 37 379 entradas de **Collins COBUILD inglés→chino**,
  redistribuidas sin licencia. **Retirar.** No lo usa ningún diccionario de AnkiMine.
- `bg/data/wordforms.json` (2,1 MB): 82 874 pares de forma inflexionada→lema en inglés, de origen
  desconocido. **Retirar** y sustituir por datos abiertos (p. ej. formas de Wiktextract) o por
  reglas de deinflexión propias. No se usa: las llamadas están comentadas.
- `fg/Mazo-de-Prueba.apkg`: incluye `AKM_enes_Cambridge_hello_1.mp3` (audio de Cambridge
  redistribuido) y un JPG de origen desconocido. **Rehacer el mazo** sin esos archivos.

### 5.3 Cobertura frente a los pares requeridos

| Par | v1.0 | Reemplazo abierto candidato (se detalla en la Fase 1) |
|---|---|---|
| EN→ES | Cambridge (scraping) | Wiktextract/kaikki.org (CC BY-SA), Tatoeba (CC BY 2.0 FR), Open English WordNet (CC BY 4.0) para definiciones monolingües |
| ES→EN | Collins (scraping) | Wiktextract (es/en Wiktionary), Tatoeba |
| JA→ES | **no existe** | JMdict (EDRDG, CC BY-SA 4.0) con glosas en español; si la cobertura es pobre, respaldo JA→EN marcado como tal [cobertura sin verificar] |
| ES→JA | **no existe** | Wiktextract + JMdict inverso, Tatoeba (oraciones ES–JA) |

---

## 6. Integración con Anki

**Acciones de AnkiConnect usadas** (`version: 6`):
- `version` (timeout de 100 ms, en cada popup);
- `deckNames`;
- `modelNames`;
- `modelFieldNames`;
- `addNote` (con `audio: {url, filename, fields}`).

**No se usan:** `canAddNotes`, `findNotes`, `guiBrowse`, `storeMediaFile`, `multi`,
`requestPermission`, `createModel`.

**Manejo de errores** [verificado en el código]:
- `ankiconnect.js:36`: cualquier error HTTP o de red se convierte en `resolve(null)`. No se
  distingue "Anki cerrado" de "resultado nulo".
- Los errores de AnkiConnect (`response.error`) se rechazan y `backend.js:208-219` los captura y
  devuelve `null`. El popup muestra un ✘ genérico durante 1 s, sin explicar la causa (duplicado,
  mazo inexistente, campo inválido, Anki cerrado).
- Si Anki está cerrado, el botón "+" se muestra deshabilitado sin ningún mensaje de ayuda. No hay
  cola ni reintento.

**Duplicados:** se envía `options.allowDuplicate` (por defecto `false`). **No hay verificación
previa:** el duplicado solo se descubre cuando `addNote` falla, y aparece como el mismo ✘ genérico.

**Mapeo de campos:** ocho ranuras fijas (`expression`, `reading`, `extrainfo`, `definition`,
`definitions`, `sentence`, `url`, `audio`), cada una asignada a un nombre de campo de Anki.
- `definition`/`definitions` llevan HTML con un bloque `<style>` antepuesto.
- `sentence` es el `innerHTML` del editor del popup.
- El audio se guarda como `AKM_<dict>_<encodeURIComponent(expr)>_<n>.mp3`.

**Tipo de nota que tienen hoy los usuarios** (del `.apkg` [verificado]):
`Minado de Oraciones - AnkiMine`, con los campos `Note, Front, Back, Sound, Reading, Picture`.
- Mapeo por defecto: `expression→Front`, `reading→Reading`, `definition→Back`,
  `sentence→Note`, `audio→Sound`.
- La extensión nunca rellena `Picture`.
- **Para no romper los mazos existentes, la v2026 debe seguir aceptando este tipo de nota y estos
  nombres de campo.**

**AnkiWeb:** inicia sesión con el usuario y la contraseña guardados, falsifica cabeceras y extrae
datos del HTML de `ankiuser.net`. Es frágil y el propio tutorial lo desaconseja. AnkiWeb cambió de
interfaz, así que es probable que ya no funcione **[sin verificar]**.

---

## 7. Problemas encontrados

Severidad: 🔴 crítica · 🟠 alta · 🟡 media · ⚪ baja.

### 7.1 Seguridad
| ID | Sev. | Hallazgo | Ubicación | Estado |
|---|---|---|---|---|
| SEC-1 | 🔴 | **Ejecución de código remoto:** descarga scripts de `raw.githubusercontent.com/ninja33/ODH` (`lib://`) o de cualquier URL del usuario y los ejecuta con `eval`. El HTML que devuelven se inyecta en todas las páginas (SEC-2), lo que equivale a XSS remoto en cualquier sitio. Además, viola la política de código remoto de MV3 y de la tienda | `bg/sandbox/sandbox.js:17-39`, `options.html:178` | [verificado en el código] |
| SEC-2 | 🟠 | **El popup es un `iframe.srcdoc` con el origen de la página anfitriona.** La página puede leer y alterar el popup (qué buscas y qué definiciones ves), y cualquier HTML inyectado se ejecuta con el origen del sitio. `expression`, `reading`, `extrainfo` y las definiciones se interpolan **sin escapar**: el texto que se obtiene con `innerText` del sitio raspado vuelve a convertirse en HTML. La CSP de la página también se aplica al popup y puede bloquear `frame.js`/`spell.js` en sitios con CSP estricta | `fg/js/popup.js:47-56`, `fg/js/frontend.js:206-241`, `dict/*.js` (`T()` + plantillas) | origen y ejecución [verificado con Playwright]; bloqueo por CSP [sin verificar] |
| SEC-3 | 🟠 | **`postMessage` sin validar el origen ni la fuente:** el content script ejecuta cualquier `api_*` que llegue por `window.message`. Cualquier página puede invocar `addNote` (tras una búsqueda) con un `context` HTML arbitrario y añadir notas a Anki, donde el HTML/JS de las tarjetas se ejecuta en el visor de Anki. También puede invocar `playSound` con cualquier URL o `setFrontendOptions`. Los mensajes salen con `targetOrigin '*'` | `fg/js/frontend.js:135-173`, `fg/js/frame.js:13-20` | [deducido] |
| SEC-4 | 🟠 | **Credenciales de AnkiWeb en texto plano** en `chrome.storage.local` (`id`, `password`) y falsificación de User-Agent/Origin/Referer | `bg/js/utils.js:11-12`, `bg/js/options.js:155-156`, `bg/js/ankiweb.js:216-262` | [verificado en el código] |
| SEC-5 | 🟡 | **Inyección de regex:** `RegExp(expression, 'gi')` sin escapar. Una entrada con `(`, `[`, `+` o `*` lanza `SyntaxError`; el sandbox nunca responde y la búsqueda se queda colgada (y se acumulan callbacks) | `dict/*.js` (p. ej. `enes_Cambridge.js:93,106`, `esen_Collins.js:71,82`) | [deducido] |
| SEC-6 | 🟡 | El fondo ejecuta cualquier `api_*` recibido por `runtime.onMessage` sin validar el remitente ni los parámetros. Un content script puede provocar un `api_Fetch` (GET a cualquier URL desde el fondo), `api_playAudio` de cualquier URL o `api_initBackend` | `bg/js/backend.js:123-132,157-169,221-238` | [deducido] |
| SEC-7 | ⚪ | jQuery 3.0.0 (2016) con CVE conocidos (CVE-2019-11358, CVE-2020-11022, CVE-2020-11023). Se usa solo en las páginas de la extensión | `lib/` | [verificado: versión] |
| SEC-8 | ⚪ | `localizeHtmlPage` usa `innerHTML` con los mensajes i18n (fuente empaquetada; es una mala práctica) | `bg/js/utils.js:67-71` | [verificado] |
| SEC-9 | ⚪ | Web: scripts por CDN sin SRI (Font Awesome, Bootstrap) | `index.html`, `english.html`, `privacy_*.html` | [verificado] |

### 7.2 Privacidad y cumplimiento
| ID | Sev. | Hallazgo |
|---|---|---|
| PRIV-1 | 🟠 | La **política de privacidad es inexacta**: dice que nada se envía a servidores externos, pero cada búsqueda envía la palabra a Cambridge/Collins/Larousse, AnkiWeb recibe las credenciales y los scripts se bajan de GitHub. Dice que no se guardan datos personales, pero se guarda la contraseña. [verificado] |
| PRIV-2 | 🟡 | Content scripts estáticos en `*://*/*` y `file://*/*` ("leer y cambiar todos tus datos en todos los sitios"). Sin `all_frames`, por lo que no funciona dentro de iframes. [verificado] |
| LIC-1 | 🟠 | Falta el aviso MIT de ODH (condición de la licencia MIT). [verificado] |
| LIC-2 | 🟠 | `collins.json` redistribuye contenido protegido de Collins. [verificado] |
| LIC-3 | 🟠 | Scraping de diccionarios comerciales y almacenamiento de su audio en Anki. [verificado] |
| LIC-4 | 🟡 | `wordforms.json` y el JPG del mazo son de origen desconocido; `sign.js` reproduce el token de Google Translate. [verificado] |
| LIC-5 | 🟡 | No hay `LICENSE` en el repo ni en el paquete; la GPL solo aparece en `legal.html`. [verificado] |

### 7.3 Bugs
| ID | Sev. | Hallazgo | Ubicación | Estado |
|---|---|---|---|---|
| BUG-1 | 🔴 | MV2: la v1.0 no carga en Chrome actual | `manifest.json` | [verificado] |
| BUG-2 | 🟡 | `optionsSave` pasa `resolve()` (lo invoca) en vez de `resolve`: la promesa se resuelve antes de guardar y hay condiciones de carrera (p. ej. `popup.js:40-41` guarda dos veces) | `bg/js/utils.js:53` | [verificado en el código] |
| BUG-3 | 🟡 | Con la tecla por defecto (**Shift**), en editores `contentEditable` (Gmail, Docs…) teclear mayúsculas con el ratón sobre texto dispara una búsqueda y **reemplaza la selección**. La exclusión de `contenteditable` está comentada | `fg/js/frontend.js:31-49`, `fg/js/text.js:217-228` | [deducido] |
| BUG-4 | 🟡 | `onSelectionEnd` usa `this.point`, que es `null` si no hubo `mousemove` (selección por teclado o táctil) → `TypeError` | `fg/js/frontend.js:108` | [deducido] |
| BUG-5 | 🟡 | La detección de palabras solo cubre el alfabeto latino (`isAlpha`, cuya clase de caracteres además incluye un `\|` literal): el modo tecla no sirve para JA/ZH/KO, aunque existe un diccionario KO→EN | `fg/js/range.js:37-39` | [deducido] |
| BUG-6 | ⚪ | En AnkiWeb, `return Promise.reject(false)` dentro de un callback: la promesa externa nunca se resuelve (cuelgue) | `bg/js/ankiweb.js:51,86` | [deducido] |
| BUG-7 | ⚪ | `onBgMessage` llama `callback()` dos veces | `fg/js/frontend.js:121,132` | [verificado en el código] |
| BUG-8 | ⚪ | Larousse toma los ejemplos de todo el artículo para cada acepción (se repiten) | `dict/dees_Larousee.js:77-78`, `fres_Larousse.js` | [deducido] |
| BUG-9 | ⚪ | La UI en español mezcla textos en inglés ("Script Name", "Note Tags", "Username", "Data Export"); el popup fija `lang="en"` | `_locales/es`, `fg/js/frontend.js:246` | [verificado] |

### 7.4 Rendimiento
| ID | Hallazgo | Estado |
|---|---|---|
| PERF-1 | La página de fondo persistente carga `collins.json` (33,3 MB) y `wordforms.json` (2,1 MB) al arrancar y **no los usa**. En Node 22, parsearlos cuesta ≈1,0 s y ≈59 MB de heap | aproximación [verificado en Node], no medido en Chrome |
| PERF-2 | Cada búsqueda descarga y analiza una página HTML completa de un tercero (timeout de 3 s), sin caché. El presupuesto de < 100 ms es inalcanzable con este diseño | [deducido]; latencia real [sin verificar: red bloqueada] |
| PERF-3 | Espera fija de 500 ms tras `selectionchange`, más un viaje de ida y vuelta a AnkiConnect (`version`) en cada popup | [verificado en el código] |
| PERF-4 | El popup se reconstruye entero como `srcdoc` en cada búsqueda (recarga su CSS y JS) | [verificado en el código] |
| PERF-5 | `tabs.onUpdated` envía las opciones varias veces por navegación; cada cambio de opciones se difunde a todas las pestañas | [verificado en el código] |
| PERF-6 | Paquete de 45 MB sin comprimir (`collins.json` 33,3 MB; `.apkg` 7,3 MB, casi todo una fuente TTF) | [verificado] |

### 7.5 Accesibilidad
| ID | Hallazgo |
|---|---|
| A11Y-1 | Las acciones del popup ("+", audio) son `<img>` con `click`, sin `alt`, `role`, `tabindex` ni manejo de teclado: no se pueden usar con teclado ni con lector de pantalla. [verificado] |
| A11Y-2 | El `iframe` del popup no tiene `title`, mide 400×300 fijos y no gestiona el foco. [verificado] |
| A11Y-3 | En las opciones, la mayoría de los `<label>` no tienen `for` (atajo, mono/bilingüe, servicio, mazo, formato, tags, duplicados) e "Iniciar sesión" es un `<label>` que hace de botón. [verificado] |
| A11Y-4 | `instructions.html`: 47 `<img>` y solo 1 con `alt`. En la web, las banderas no tienen `alt` y el logo usa `alt="..."`. [verificado] |
| A11Y-5 | Atajo limitado a Shift/Ctrl/Alt mediante el obsoleto `keyCode`. No se respeta `prefers-reduced-motion`. El contraste no se evaluó [sin verificar]. |

---

## 8. Datos en `chrome.storage` (insumo para las migraciones)

Área: **`chrome.storage.local`**, con claves en el nivel superior (`get(null)`). **No hay clave de
versión de esquema.** Los valores por defecto están en `bg/js/utils.js:1-40` [verificado].

| Clave | Tipo | Defecto | Valores / notas |
|---|---|---|---|
| `enabled` | boolean | `true` | activar o desactivar (también con `Alt+Q`) |
| `mouseselection` | boolean | `true` | buscar al seleccionar con el ratón |
| `hotkey` | string | `'16'` | `'0'` off, `'16'` Shift, `'17'` Ctrl, `'18'` Alt (keyCode) |
| `maxcontext` | string | `'2'` | `'1'`–`'3'` oraciones de contexto |
| `maxexample` | string | `'2'` | `'0'`–`'2'` ejemplos por acepción |
| `monolingual` | string | `'0'` | `'0'` bilingüe, `'1'` monolingüe (oculta la traducción) |
| `preferredaudio` | string | `'1'` | `'0'` UK, `'1'` US |
| `services` | string | `'none'` | `'none'` · `'ankiconnect'` · `'ankiweb'` |
| `id` | string | `''` | usuario de AnkiWeb |
| `password` | string | `''` | **contraseña de AnkiWeb en texto plano** |
| `duplicate` | string | `'0'` | `'0'` no permitir, `'1'` permitir |
| `tags` | string | `''` | etiquetas separadas por espacios |
| `deckname` | string | `'Default'` | mazo |
| `typename` | string | `'Minado de Oraciones - AnkiMine'` | tipo de nota |
| `expression` | string | `'Front'` | campo de Anki para la palabra |
| `reading` | string | `'Reading'` | campo para la pronunciación (IPA) |
| `extrainfo` | string | `''` | campo para la info extra (frecuencia) |
| `definition` | string | `'Back'` | campo para la acepción elegida |
| `definitions` | string | `''` | campo para todas las acepciones |
| `sentence` | string | `'Note'` | campo para la oración de contexto |
| `url` | string | `''` | campo para la URL de origen |
| `audio` | string | `'Sound'` | campo para el audio |
| `sysscripts` | string | `'builtin_select'` | lista separada por comas de nombres de script; prefijo `lib://` = remoto |
| `udfscripts` | string | `''` | lista separada por comas de URLs de scripts del usuario |
| `dictSelected` | string | `''` | nombre de clase del diccionario activo (p. ej. `enes_Cambridge`) |
| `dictNamelist` | array | `[]` | estado derivado: `[{objectname, displayname}]` |
| `dictLibrary` | string | — | **heredada** de ODH; `api_initBackend` la copia a `sysscripts` |

Implicaciones para la Fase 1:
- Los valores numéricos se guardan como strings. Hay estado derivado guardado (`dictNamelist`) y
  nombres de diccionarios que dejarán de existir (`enes_Cambridge` → hay que mapearlos a los nuevos
  diccionarios abiertos del mismo par).
- La migración debe **borrar** `id` y `password`, además de ignorar `sysscripts`, `udfscripts` y
  `dictLibrary`.
- ⚠️ `chrome.storage` está **aislado por ID de extensión**. Si el fork se publica con un ID nuevo,
  **no puede leer** la configuración de los usuarios de `pnjgmipbioobeiekighjafkcgkebnmjc`. La
  migración solo es automática si se publica con el mismo ID (transferencia del autor original).
  Si no, hace falta un importador (p. ej. pegar el JSON de `chrome.storage.local.get(null)` de la
  extensión vieja).
- Los mazos y las plantillas viven en Anki, no en la extensión: se conservan si se mantiene la
  compatibilidad con el tipo de nota de la sección 6.
- El esquema de la v2.0.0 de la tienda es desconocido [sin verificar].

---

## 9. Recomendaciones priorizadas

Esfuerzo: **S** ≤ 1 día · **M** 2–5 días · **L** > 1 semana.

| # | Prioridad | Recomendación | Esf. | Resuelve |
|---|---|---|---|---|
| 1 | P0 | Definir la base de código y la licencia: GPL-3.0-or-later + aviso MIT de ODH + créditos; pedir confirmación a Fabian Soto | S | LIC-1, LIC-5 |
| 2 | P0 | Eliminar el código remoto, `eval` y `udfscripts`; diccionarios solo como **datos** (importador del formato Yomitan) | L | SEC-1, MV3 |
| 3 | P0 | Retirar los diccionarios por scraping, `collins.json`, `wordforms.json`, `sign.js` y el audio de Cambridge del mazo; sustituirlos por fuentes abiertas (sección 5.3) | S + L | LIC-2/3/4, PERF-1/2/6 |
| 4 | P0 | Eliminar AnkiWeb (credenciales, `webRequestBlocking`); la migración borra `id`/`password` | S | SEC-4, PRIV-1 |
| 5 | P0 | Popup en **Shadow DOM** (cerrado) o en una página con origen de la extensión; renderizar sin `innerHTML` de datos externos (texto, o HTML saneado con una lista blanca) | M | SEC-2 |
| 6 | P0 | Mensajería tipada: validar `event.source`/origen, esquema de parámetros y lista blanca de acciones | S/M | SEC-3, SEC-6 |
| 7 | P0 | Nueva base MV3 (service worker + offscreen si hace falta), TypeScript estricto | L | BUG-1 |
| 8 | P1 | Cliente de AnkiConnect robusto: errores tipados, `canAddNotes` previo, `requestPermission`, `multi`, `storeMediaFile`, `guiBrowse`, cola sin conexión | M | sección 6 |
| 9 | P1 | Esquema de almacenamiento versionado + migración desde la v1.0 con pruebas + importador para usuarios del ID original | M | sección 8 |
| 10 | P1 | Diccionarios en IndexedDB, búsqueda en un Worker, carga diferida; medir el presupuesto de < 100 ms | M/L | PERF-1..4 |
| 11 | P1 | Japonés: escaneo + deinflexión (estilo Yomitan) y JMdict; furigana | L | BUG-5, sección 5.3 |
| 12 | P1 | Permisos mínimos: `optional_host_permissions` por sitio, `activeTab` | M | PRIV-2 |
| 13 | P1 | Pruebas de regresión para BUG-2..4 y SEC-5 (escapar regex, `contentEditable`, punto nulo) | S | BUG-2..4, SEC-5 |
| 14 | P2 | Accesibilidad: botones reales, teclado, `aria-*`, `label for`, `alt` | M | A11Y-1..5 |
| 15 | P2 | i18n completo es/en/ja, sin textos fijos en el HTML | M | BUG-9 |
| 16 | P2 | Reescribir la política de privacidad según el comportamiento real | S | PRIV-1 |
| 17 | P2 | Web: SRI, `alt`; README con instalación, AnkiConnect, idiomas y créditos | S | SEC-9, A11Y-4 |
| 18 | P2 | Quitar jQuery (cae solo con la reescritura) | S | SEC-7 |

---

## 10. Decisiones pendientes (para ti)

1. **Base de código.** El repo no contiene la extensión. Opciones:
   - (A) usar la v1.0 del historial (GPL declarada) como referencia funcional y reescribir sobre la
     nueva arquitectura — **recomendada**;
   - (B) que extraigas tú el código de la v2.0.0 instalada desde la tienda (licencia sin
     confirmar);
   - (C) pedir a Fabian Soto el repositorio fuente.
2. **Licencia del fork.** Crear `LICENSE` con la GPL-3.0-or-later y `THIRD_PARTY_NOTICES.md` con el
   aviso MIT de ODH. Hoy no existe ningún `LICENSE` que "conservar intacto", y tocarlo requiere tu
   OK.
3. **ID de extensión y migración.** Si el ID es nuevo, no hay migración automática (sección 8).
   ¿Pedimos la transferencia de la ficha o hacemos un importador?
4. **Nombre visible.** ¿Se mantiene "AnkiMine" o se diferencia (p. ej. "AnkiMine Libre") para no
   confundir con la ficha original?
5. **Repositorio y archivos de la web.** El prompt menciona `EmaDevCol/AnkiMineSegundo`, pero esta
   sesión tiene `EmaDevCol/AnkiMine` (fork de `fabianesm/AnkiMine`, el único accesible). ¿Desarrollo
   aquí? ¿La web se queda en la raíz o se mueve a `site/`? Moverla o borrarla requiere tu OK.
6. **Rama.** El prompt pide `v2026`; esta sesión de Claude Code tiene asignada
   `claude/clever-bohr-ltbv1c`. El commit de esta fase está solo en local, en esta última.
7. **AnkiWeb.** Confirmar su eliminación (incompatible con MV3 y guarda credenciales).
8. **Compatibilidad del tipo de nota.** Confirmar que la v2026 debe soportar
   `Minado de Oraciones - AnkiMine` (`Note/Front/Back/Sound/Reading/Picture`) por defecto
   (recomendado).
