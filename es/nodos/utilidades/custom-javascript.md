# Nodo Custom JavaScript

El nodo **Custom JavaScript** permite ejecutar código JavaScript personalizado durante la ejecución del flujo. Es ideal para transformaciones de datos, cálculos complejos y lógica no cubierta por los nodos nativos.

Además de JavaScript puro, este nodo también puede trabajar con sesiones **Web** y **Mobile** ya abiertas, permitiendo ejecutar automatizaciones avanzadas con **Playwright** y **Appium** directamente desde el código.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `custom-js` |
| **Categoría** | Utilidades |
| **Color** | ⚪ Gris (#6b7280) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Configuración

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Código** | `string` | Código JavaScript a ejecutar |

El editor de código usa el **Monaco Editor** (el mismo de VS Code) con:
- Syntax highlighting
- Autocomplete para `steps`, `variables`, `web`, `mobile`, `page`, `app`, `return`, `console.log`
- Validación de sintaxis

---

## Contexto de Ejecución

El código tiene acceso a:

| Variable | Tipo | Descripción |
|----------|------|-------------|
| `steps` | `object` | Outputs de todos los nodos anteriores |
| `variables` | `object` | Variables globales y de runtime |
| `expect` | `function` | Assertions de Playwright para escenarios Web |
| `assert` | `object` | Assertions de Node.js |
| `fetch` | `function` | Requisiciones HTTP directas vía JavaScript |
| `web` | `object` | Namespace para trabajar con sesiones Web ya abiertas |
| `mobile` | `object` | Namespace para trabajar con sesiones Mobile ya abiertas |
| `page` | `object` | Alias global de la página Web actual |
| `context` | `object` | Alias global del contexto Web actual |
| `browser` | `object` | Alias global del browser actual |
| `app` | `object` | Alias global de la sesión Mobile actual |
| `client` | `object` | Alias de compatibilidad para `app` |

### Acceder a datos de nodos anteriores

```javascript
// Output de un HTTP Request
const userData = steps["http-request"].outputs.json;
const userName = userData.name;

// Output de una consulta PostgreSQL
const rows = steps["postgres-query"].outputs.rows;
const firstUser = rows[0];

// Output de extracción web
const title = steps["web-flow"].outputs.extracts.pageTitle;
```

### Acceder a variables

```javascript
const apiUrl = variables.API_URL;
const environment = variables.ENVIRONMENT;
```

---

## Sesiones Web y Mobile

El nodo **Custom JavaScript** no crea automáticamente una sesión Web o Mobile. Usa una sesión que ya fue abierta por un nodo anterior.

### Sesión Web

Para usar `web`, `page`, `context` o `browser`, necesitas una sesión Web activa creada por un nodo como:
- **Web Flow**
- **Smart Locators**

### Sesión Mobile

Para usar `mobile`, `app` o `client`, necesitas una sesión Mobile activa creada por un nodo como:
- **Mobile Flow**

### Regla práctica

- Usa `web` cuando quieras extender una automatización Web ya abierta
- Usa `mobile` cuando quieras extender una automatización Mobile ya abierta
- Si no hay una sesión activa, estas APIs fallarán

---

## Automatización Web con Playwright

El namespace `web` expone la sesión Web actual y utilidades avanzadas basadas en **Playwright**.

### Métodos del namespace `web`

| Método | Descripción |
|--------|-------------|
| `web.hasSession(sessionId?)` | Verifica si existe una sesión Web activa |
| `web.ids()` | Retorna los IDs de las sesiones Web disponibles |
| `web.sessions()` | Retorna todas las sesiones Web disponibles |
| `web.current()` | Retorna la sesión Web actual |
| `web.session(sessionId)` | Retorna una sesión Web específica |
| `web.page(sessionId?)` | Retorna la `page` de Playwright |
| `web.context(sessionId?)` | Retorna el `context` de Playwright |
| `web.browser(sessionId?)` | Retorna el `browser` de Playwright |
| `web.screenshot(name?, options?, sessionId?)` | Captura screenshot de la sesión Web |
| `web.run(fn, sessionId?)` | Ejecuta código avanzado sobre la sesión Web |

### Objeto retornado por `web.current()` y `web.session()`

| Campo / Método | Descripción |
|----------------|-------------|
| `id` | ID interno de la sesión |
| `sessionId` | ID público de la sesión |
| `page` | Instancia Playwright `page` |
| `context` | Instancia Playwright `context` |
| `browser` | Instancia Playwright `browser` |
| `session` | Objeto bruto de la sesión |
| `storageStrategy` | Estrategia de storage de la sesión |
| `persistedKey` | Clave de persistencia de la sesión |
| `url()` | Retorna la URL actual |
| `title()` | Retorna el título actual |
| `locator(selector)` | Crea un locator de Playwright |
| `screenshot(name?, options?)` | Captura screenshot de esa sesión |

### Métodos más usados en `page`

El objeto `page` es una instancia de Playwright. Métodos comunes:

| Método | Descripción |
|--------|-------------|
| `page.goto(url, options?)` | Navega a una URL |
| `page.url()` | Retorna la URL actual |
| `page.title()` | Retorna el título actual |
| `page.reload()` | Recarga la página |
| `page.goBack()` | Vuelve en el historial |
| `page.goForward()` | Avanza en el historial |
| `page.locator(selector)` | Crea un locator |
| `page.getByRole(...)` | Busca por role accesible |
| `page.getByText(...)` | Busca por texto |
| `page.getByLabel(...)` | Busca por label |
| `page.getByPlaceholder(...)` | Busca por placeholder |
| `page.getByTestId(...)` | Busca por `data-testid` |
| `page.waitForURL(...)` | Espera cambio de URL |
| `page.waitForResponse(...)` | Espera una respuesta de red |
| `page.waitForSelector(...)` | Espera a que aparezca un selector |
| `page.evaluate(...)` | Ejecuta JavaScript dentro de la página |
| `page.screenshot(...)` | Captura screenshot vía Playwright |

### Screenshots Web

Por defecto, `web.screenshot()` captura solo el **viewport actual**, siguiendo el mismo comportamiento de los nodos Web.

```javascript
await web.screenshot("pagina-actual");
```

Para capturar la página completa:

```javascript
await web.screenshot("pagina-completa", { fullPage: true });
```

### `web.run(...)`

`web.run(...)` es la forma recomendada de ejecutar código avanzado sobre la sesión Web. El callback recibe:

| Campo | Descripción |
|-------|-------------|
| `page` | Página actual |
| `context` | Contexto actual |
| `browser` | Browser actual |
| `session` | Sesión actual |
| `sessionId` | ID de la sesión |
| `id` | ID interno de la sesión |
| `storageStrategy` | Estrategia de storage |
| `persistedKey` | Clave de persistencia |
| `url` | Helper para la URL actual |
| `title` | Helper para el título actual |
| `locator` | Helper para locators |
| `screenshot` | Helper para screenshots |
| `expect` | Assertions de Playwright |
| `assert` | Assertions de Node.js |

### Ejemplo Web: validar productos y capturar screenshot

```javascript
return await web.run(async ({ page, expect }) => {
  await page.goto('https://automationexercise.com/products', {
    waitUntil: 'domcontentloaded'
  });

  const products = page.locator('.features_items .product-image-wrapper');
  await expect(products).toHaveCount(34);

  await web.screenshot('web-products');

  return {
    url: page.url(),
    title: await page.title(),
    productCount: await products.count()
  };
});
```

### Ejemplo Web: usar la sesión actual sin navegar

```javascript
return await web.run(async ({ page }) => {
  return {
    url: page.url(),
    title: await page.title()
  };
});
```

### Ejemplo Web: usar el alias global `page`

```javascript
await page.goto('https://example.com');
await web.screenshot('example-home');

return {
  title: await page.title(),
  url: page.url()
};
```

---

## Automatización Mobile con Appium

El namespace `mobile` expone la sesión Mobile actual y utilidades avanzadas basadas en **Appium**.

### Métodos del namespace `mobile`

| Método | Descripción |
|--------|-------------|
| `mobile.hasSession(sessionId?)` | Verifica si existe una sesión Mobile activa |
| `mobile.ids()` | Retorna los IDs de las sesiones Mobile disponibles |
| `mobile.sessions()` | Retorna todas las sesiones Mobile disponibles |
| `mobile.current()` | Retorna la sesión Mobile actual |
| `mobile.session(sessionId)` | Retorna una sesión Mobile específica |
| `mobile.app(sessionId?)` | Retorna el helper Mobile principal |
| `mobile.client(sessionId?)` | Alias de compatibilidad para `mobile.app()` |
| `mobile.screenshot(name?, sessionId?)` | Captura screenshot de la sesión Mobile |
| `mobile.source(sessionId?)` | Retorna el XML/source de la pantalla actual |
| `mobile.run(fn, sessionId?)` | Ejecuta código avanzado sobre la sesión Mobile |

### Objeto retornado por `mobile.current()` y `mobile.session()`

| Campo / Método | Descripción |
|----------------|-------------|
| `id` | ID interno de la sesión |
| `sessionId` | ID público de la sesión |
| `app` | Helper Mobile principal |
| `client` | Alias de compatibilidad para `app` |
| `capabilities` | Capabilities de la sesión |
| `session` | Objeto bruto de la sesión |
| `screenshot(name?)` | Captura screenshot de la sesión |
| `source()` | Retorna el source/XML de la pantalla |
| `request(method, path, body?)` | Hace una llamada directa a la sesión Appium |
| `executeScript(script, args?)` | Ejecuta un script de Appium |

### `app` y `client`

El nombre principal para Mobile es **`app`**.  
`client` sigue existiendo por compatibilidad, pero el uso recomendado es `app`.

### Métodos expuestos en `app`

| Método | Descripción |
|--------|-------------|
| `createSession(capabilities)` | Crea una nueva sesión Appium |
| `deleteSession(sessionId)` | Cierra una sesión |
| `screenshot(sessionId)` | Captura screenshot de la sesión |
| `getPageSource(sessionId)` | Retorna el XML/source de la pantalla |
| `findElement(sessionId, using, value)` | Busca un elemento |
| `findElements(sessionId, using, value)` | Busca varios elementos |
| `elementClick(sessionId, elementId)` | Hace click en un elemento |
| `elementSendKeys(sessionId, elementId, text)` | Escribe en un elemento |
| `sendKeys(sessionId, text)` | Envía texto directamente a la sesión |
| `elementClear(sessionId, elementId)` | Limpia un campo |
| `elementText(sessionId, elementId)` | Retorna el texto del elemento |
| `elementAttribute(sessionId, elementId, name)` | Retorna un atributo del elemento |
| `elementDisplayed(sessionId, elementId)` | Verifica si el elemento está visible |
| `tapAt(sessionId, x, y)` | Toca en coordenadas |
| `doubleTapAt(sessionId, x, y)` | Ejecuta double tap en coordenadas |
| `swipe(sessionId, startX, startY, endX, endY, durationMs?)` | Ejecuta swipe |
| `pressKeyCode(sessionId, keyCode)` | Presiona un keycode Android |
| `req(method, path, body?)` | Hace una llamada bruta a Appium |
| `executeScript(sessionId, script, args?)` | Ejecuta un script Appium |
| `launchApp(sessionId)` | Abre nuevamente la app |
| `resetApp(sessionId)` | Resetea la app |

### Screenshots Mobile

```javascript
await mobile.screenshot('mobile-current-screen');
```

### `mobile.run(...)`

`mobile.run(...)` es la forma recomendada de ejecutar código avanzado sobre la sesión Mobile. El callback recibe:

| Campo | Descripción |
|-------|-------------|
| `app` | Helper Mobile principal |
| `client` | Alias para `app` |
| `sessionId` | ID de la sesión |
| `id` | ID interno de la sesión |
| `capabilities` | Capabilities actuales |
| `session` | Sesión actual |
| `screenshot` | Helper para screenshots |
| `source` | Helper para source |
| `request` | Helper para llamadas directas |
| `executeScript` | Helper para scripts Appium |
| `assert` | Assertions de Node.js |

### Ejemplo Mobile: tocar por coordenadas y capturar screenshot

```javascript
return await mobile.run(async ({ app, sessionId }) => {
  const source = await app.getPageSource(sessionId);

  await app.tapAt(sessionId, 200, 400);
  await mobile.screenshot('mobile-after-tap');

  return {
    sessionId,
    sourceLength: source.length,
    tapped: true
  };
});
```

### Ejemplo Mobile: click por `accessibility id`

```javascript
return await mobile.run(async ({ app, sessionId }) => {
  const elementId = await app.findElement(sessionId, 'accessibility id', 'login-button');
  await app.elementClick(sessionId, elementId);

  await mobile.screenshot('login-button-clicked');

  return {
    clicked: true,
    elementId
  };
});
```

### Ejemplo Mobile: usar el alias global `app`

```javascript
const current = mobile.current();
const source = await app.getPageSource(current.sessionId);

return {
  sessionId: current.sessionId,
  sourceLength: source.length
};
```

---

## Retornar Valores

El código **debe retornar** un valor usando `return`. El valor retornado queda disponible en los outputs:

```javascript
// Retornar un valor simple
return "Hello World";

// Retornar un objeto
return {
  fullName: steps["Query"].outputs.rows[0].first_name + " " + steps["Query"].outputs.rows[0].last_name,
  isAdmin: steps["Query"].outputs.rows[0].role === "admin"
};

// Retornar un array
return steps["Query"].outputs.rows.map(row => row.email);
```

### Retornar el resultado de `web.run()` y `mobile.run()`

Si quieres que el valor retornado dentro de `web.run(...)` o `mobile.run(...)` aparezca en `outputs.result`, el script principal también debe retornar ese valor:

```javascript
const result = await web.run(async ({ page }) => {
  return { url: page.url() };
});

return result;
```

O directamente:

```javascript
return await mobile.run(async ({ app, sessionId }) => {
  await app.tapAt(sessionId, 200, 400);
  return { tapped: true };
});
```

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `result` | `any` | Valor retornado por el código |

### Acceder al resultado

```
{{ steps["custom-js"].outputs.result }}
{{ steps["custom-js"].outputs.result.fullName }}
{{ steps["custom-js"].outputs.result[0] }}
```

---

## Ejemplos Prácticos

### Transformar datos de una API

```javascript
const items = steps["http-request"].outputs.json.items;

// Filtrar y transformar
const activeItems = items
  .filter(item => item.active)
  .map(item => ({
    id: item.id,
    name: item.name.toUpperCase(),
    price: (item.price * 1.1).toFixed(2)
  }));

return {
  items: activeItems,
  count: activeItems.length,
  totalPrice: activeItems.reduce((sum, item) => sum + parseFloat(item.price), 0)
};
```

### Generar datos de prueba

```javascript
const timestamp = Date.now();
return {
  email: `test-${timestamp}@ejemplo.com`,
  username: `user_${timestamp}`,
  password: `Pass${timestamp}!`
};
```

### Validación compleja

```javascript
const response = steps["API Check"].outputs.json;
const dbRecord = steps["DB Query"].outputs.rows[0];

const validations = [];

if (response.name !== dbRecord.name) {
  validations.push(`Diferencia de nombre: API="${response.name}" DB="${dbRecord.name}"`);
}

if (response.email !== dbRecord.email) {
  validations.push(`Diferencia de email: API="${response.email}" DB="${dbRecord.email}"`);
}

return {
  valid: validations.length === 0,
  errors: validations,
  errorCount: validations.length
};
```

### Formatear un reporte

```javascript
const runs = steps["Query Runs"].outputs.rows;

const summary = {
  total: runs.length,
  passed: runs.filter(r => r.status === 'success').length,
  failed: runs.filter(r => r.status === 'failed').length,
  passRate: 0
};

summary.passRate = ((summary.passed / summary.total) * 100).toFixed(1) + '%';

return summary;
```

### JavaScript puro

```javascript
const now = new Date().toISOString();
const total = [10, 20, 30].reduce((sum, value) => sum + value, 0);

return {
  now,
  total,
  average: total / 3
};
```

---

## Limitaciones

- El código se ejecuta en un **entorno sandbox**
- No hay acceso a:
  - Sistema de archivos (`fs`)
  - Módulos Node.js vía `require` / `import`
  - `process.env` directamente
- Para variables de entorno, usa `variables`
- Para interacciones Web y Mobile, ya debe existir una sesión activa
- `client` sigue soportado, pero el nombre recomendado para Mobile es `app`

> `fetch` está disponible. Aun así, para flujos HTTP estructurados y más fáciles de mantener, es preferible usar el nodo **HTTP Request**.

---

## Consejos

- Usa nombres descriptivos para el nodo: "Transformar Datos", "Generar Credenciales", "Validar Respuesta", "Código Web Avanzado", "Código Mobile Avanzado"
- **Siempre retorna** un valor — sin `return`, el output será `undefined`
- Para depuración, usa `console.log()` — los mensajes aparecen en los logs del nodo
- Monaco Editor ofrece autocomplete — escribe `steps.`, `web.` o `mobile.` para ver las opciones disponibles
- Para automatización Web avanzada, prefiere `web.run(...)`
- Para automatización Mobile avanzada, prefiere `mobile.run(...)`
- Usa `web.screenshot()` y `mobile.screenshot()` cuando necesites evidencias adicionales
- Úsalo para **transformar datos** entre nodos y para resolver casos avanzados que no fueron cubiertos por los nodos visuales
