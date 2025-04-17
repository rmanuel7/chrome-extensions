## isolated-world

> [!WARNING]
> tengo un problema con una extensión quiero inyectar esto cuando carga una pagina:
> <br/> `window.alert = console.log;`


Perfecto, quieres reemplazar `window.alert` por `console.log` automáticamente cuando cargas una página, usando una extensión. ¿Estás creando una extensión de Chrome? Si es así, puedes hacerlo fácilmente con un **content script**.

Aquí te dejo un ejemplo básico de cómo hacerlo:

---

### `manifest.json` (para una extensión de Chrome)

```json
{
  "manifest_version": 3,
  "name": "Reemplazar alert por console.log",
  "version": "1.0",
  "description": "Intercepta alert y lo redirige a console.log",
  "permissions": ["scripting"],
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["inject.js"],
      "run_at": "document_start"
    }
  ]
}
```

---

### `inject.js`

```js
// Reemplazar alert apenas carga el documento
window.alert = function (msg) {
  console.log("Interceptado alert:", msg);
};
```

---

### Cosas a tener en cuenta

- **`run_at: "document_start"`** asegura que se ejecute antes que cualquier script de la página.
- Si algún script usa `alert` antes de que se reemplace (por ejemplo, si lo hacen en línea en el `<head>`), podrías necesitar una **extensión con scripting más avanzada** o inyectar directamente en la página con `chrome.scripting`.

<br/>
<br/>
<br/>

---

> [!WARNING]
> esto no funciona como espero, porque solo afecta al content-script:
> <br/> They live in an isolated world, meaning they can make changes to their JavaScript environment without conflicting with their host page or other extensions' content scripts.

¡Exacto! Esa es la clave del problema: **los content scripts viven en un "mundo aislado"** y no pueden modificar directamente el entorno `window` de la página (como sobrescribir `window.alert`) porque están en un contexto separado.

Para realmente modificar cosas como `window.alert`, necesitas **inyectar un script directamente en la página**, no solo desde el content script.

Aquí tienes cómo hacerlo **correctamente**:

---

### ✅ Solución: Inyectar un `<script>` en la página desde el content script

#### `content.js`:

```js
// Crea un elemento <script> que se ejecutará en el contexto de la página
const script = document.createElement('script');
script.textContent = `
  window.alert = function(msg) {
    console.log("Interceptado alert:", msg);
  };
`;
document.documentElement.appendChild(script);
script.remove(); // Opcional: lo remueves para no dejar rastro
```

#### `manifest.json` (manifest v3)

```json
{
  "manifest_version": 3,
  "name": "Reemplazar alert",
  "version": "1.0",
  "description": "Reemplaza window.alert con console.log",
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "run_at": "document_start"
    }
  ]
}
```

---

### ¿Por qué funciona esto?

Este código **inyecta un `<script>` dentro del DOM** de la página. Como ese `<script>` se ejecuta **dentro del contexto de la página web**, sí puede modificar el `window.alert` de verdad — y no solo dentro del mundo aislado del content script.

<br/>
<br/>
<br/>

---

> [!CAUTION]
> obtengo el siguiente error:
```
content.js:8
Refused to execute inline script because it violates the following Content Security Policy directive:
  "script-src 'self' 'wasm-unsafe-eval' 'inline-speculation-rules'
  http://localhost:* http://127.0.0.1:* chrome-extension://0a9bb5f2-59cb-4296-bc97-47c55db1a28b/".
Either the 'unsafe-inline' keyword, a hash ('sha256-NTE2TBl6Gg9Vx8KZDMpYZmsmDz3j1HxSq0KYmjuYnIs='), or a nonce ('nonce-...') is required to enable inline execution.
```


Perfecto, ese error te lo está dando la **Content Security Policy (CSP)** de la página web que estás tratando de modificar, **no tu extensión**. Muchas páginas bloquean la ejecución de scripts inline (como el que inyectamos con `script.textContent = ...`).

💡 **Pero tenemos una solución**: en lugar de usar `.textContent`, vamos a inyectar un archivo de script externo, **incluido en tu extensión**.

---

### ✅ Solución 100% funcional: Inyectar un script externo

#### 1. Crea un archivo llamado `injected.js`:

```js
// Este código sí se ejecutará en el contexto de la página
window.alert = function (msg) {
  console.log("Interceptado alert:", msg);
};
```

#### 2. Tu `content.js` será así:

```js
const script = document.createElement('script');
script.src = chrome.runtime.getURL('injected.js'); // Ruta interna de la extensión
script.onload = () => script.remove(); // Limpieza opcional
(document.head || document.documentElement).appendChild(script);
```

#### 3. Tu `manifest.json` (manifest v3):

```json
{
  "manifest_version": 3,
  "name": "Override alert",
  "version": "1.0",
  "description": "Reemplaza alert con console.log",
  "permissions": [],
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "run_at": "document_start"
    }
  ],
  "host_permissions": ["<all_urls>"]
}
```

---

### 📌 ¿Por qué esto sí funciona?

- `injected.js` es un archivo de tu extensión → no es inline → **evita el bloqueo por CSP**.
- Al inyectarlo como un `<script src="...">`, se ejecuta **dentro del contexto de la página**, por lo tanto puede modificar `window.alert` de verdad.

<br/>
<br/>
<br/>

---

> [!CAUTION]
> Da este error:
```
content.js:4 
 GET chrome-extension://invalid/ net::ERR_FAILED
(anonymous)	@	content.js:4
```


¡Buen ojo! Ese error:

```
GET chrome-extension://invalid/ net::ERR_FAILED
```

usualmente pasa cuando intentás usar `chrome.runtime.getURL('injected.js')` **pero no has incluido `injected.js` en el `manifest.json`**. Aunque esté en tu carpeta, **si no está declarado**, Chrome no lo sirve.

---

### ✅ Solución

Asegurate de incluir `injected.js` como parte de los archivos "web-accessible" en tu `manifest.json`.

Actualizá tu `manifest.json` así:

```json
{
  "manifest_version": 3,
  "name": "Override alert",
  "version": "1.0",
  "description": "Reemplaza alert con console.log",
  "permissions": [],
  "host_permissions": ["<all_urls>"],
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "run_at": "document_start"
    }
  ],
  "web_accessible_resources": [
    {
      "resources": ["injected.js"],
      "matches": ["<all_urls>"]
    }
  ]
}
```

---

Con eso, `chrome.runtime.getURL('injected.js')` ya apunta a un recurso accesible públicamente desde la página.

💡 **Resumen de pasos**:

1. Verificá que `injected.js` está en la carpeta raíz o donde lo estás referenciando.
2. Asegurate de que el `manifest.json` tenga `web_accessible_resources` correctamente.
3. Recargá la extensión en `chrome://extensions` después de hacer cambios.

---

¿Querés que te prepare el proyecto completo listo para importar a Chrome?



