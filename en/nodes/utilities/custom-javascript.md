# Custom JavaScript Node

The **Custom JavaScript** node allows you to execute custom JavaScript code during flow execution. It is ideal for data transformations, complex calculations, and logic not covered by native nodes.

In addition to plain JavaScript, this node can also work with already-open **Web** and **Mobile** sessions, allowing you to run advanced automation with **Playwright** and **Appium** directly from code.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `custom-js` |
| **Category** | Utilities |
| **Color** | ⚪ Gray (#6b7280) |
| **Input** | `in` |
| **Output** | `out` |

---

## Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Code** | `string` | JavaScript code to be executed |

The code editor uses the **Monaco Editor** (same as VS Code) with:
- Syntax highlighting
- Autocomplete for `steps`, `variables`, `web`, `mobile`, `page`, `app`, `return`, `console.log`
- Syntax validation

---

## Execution Context

The code has access to:

| Variable | Type | Description |
|----------|------|-------------|
| `steps` | `object` | Outputs from all previous nodes |
| `variables` | `object` | Global and runtime variables |
| `expect` | `function` | Playwright assertions for Web scenarios |
| `assert` | `object` | Node.js assertions |
| `fetch` | `function` | Direct HTTP requests via JavaScript |
| `web` | `object` | Namespace for working with already-open Web sessions |
| `mobile` | `object` | Namespace for working with already-open Mobile sessions |
| `page` | `object` | Global alias for the current Web page |
| `context` | `object` | Global alias for the current Web context |
| `browser` | `object` | Global alias for the current browser |
| `app` | `object` | Global alias for the current Mobile session |
| `client` | `object` | Compatibility alias for `app` |

### Accessing data from previous nodes

```javascript
// Output from an HTTP Request
const userData = steps["http-request"].outputs.json;
const userName = userData.name;

// Output from a PostgreSQL query
const rows = steps["postgres-query"].outputs.rows;
const firstUser = rows[0];

// Output from web extraction
const title = steps["web-flow"].outputs.extracts.pageTitle;
```

### Accessing variables

```javascript
const apiUrl = variables.API_URL;
const environment = variables.ENVIRONMENT;
```

---

## Web and Mobile Sessions

The **Custom JavaScript** node does not automatically create a Web or Mobile session. It uses a session that was already opened by a previous node.

### Web session

To use `web`, `page`, `context`, or `browser`, you need an active Web session created by a node such as:
- **Web Flow**
- **Smart Locators**

### Mobile session

To use `mobile`, `app`, or `client`, you need an active Mobile session created by a node such as:
- **Mobile Flow**

### Practical rule

- Use `web` when you want to extend an already-open Web automation
- Use `mobile` when you want to extend an already-open Mobile automation
- If there is no active session, these APIs will fail

---

## Web Automation with Playwright

The `web` namespace exposes the current Web session and advanced utilities powered by **Playwright**.

### `web` namespace methods

| Method | Description |
|--------|-------------|
| `web.hasSession(sessionId?)` | Checks whether there is an active Web session |
| `web.ids()` | Returns the IDs of available Web sessions |
| `web.sessions()` | Returns all available Web sessions |
| `web.current()` | Returns the current Web session |
| `web.session(sessionId)` | Returns a specific Web session |
| `web.page(sessionId?)` | Returns the Playwright `page` |
| `web.context(sessionId?)` | Returns the Playwright `context` |
| `web.browser(sessionId?)` | Returns the Playwright `browser` |
| `web.screenshot(name?, options?, sessionId?)` | Captures a screenshot from the Web session |
| `web.run(fn, sessionId?)` | Runs advanced code against the Web session |

### Object returned by `web.current()` and `web.session()`

| Field / Method | Description |
|----------------|-------------|
| `id` | Internal session ID |
| `sessionId` | Public session ID |
| `page` | Playwright `page` instance |
| `context` | Playwright `context` instance |
| `browser` | Playwright `browser` instance |
| `session` | Raw session object |
| `storageStrategy` | Session storage strategy |
| `persistedKey` | Session persistence key |
| `url()` | Returns the current URL |
| `title()` | Returns the current page title |
| `locator(selector)` | Creates a Playwright locator |
| `screenshot(name?, options?)` | Captures a screenshot from that session |

### Most common `page` methods

The `page` object is a Playwright instance. Common methods include:

| Method | Description |
|--------|-------------|
| `page.goto(url, options?)` | Navigates to a URL |
| `page.url()` | Returns the current URL |
| `page.title()` | Returns the current title |
| `page.reload()` | Reloads the page |
| `page.goBack()` | Goes back in history |
| `page.goForward()` | Goes forward in history |
| `page.locator(selector)` | Creates a locator |
| `page.getByRole(...)` | Finds by accessible role |
| `page.getByText(...)` | Finds by text |
| `page.getByLabel(...)` | Finds by label |
| `page.getByPlaceholder(...)` | Finds by placeholder |
| `page.getByTestId(...)` | Finds by `data-testid` |
| `page.waitForURL(...)` | Waits for a URL change |
| `page.waitForResponse(...)` | Waits for a network response |
| `page.waitForSelector(...)` | Waits for a selector to appear |
| `page.evaluate(...)` | Executes JavaScript in the page |
| `page.screenshot(...)` | Captures a screenshot via Playwright |

### Web screenshots

By default, `web.screenshot()` captures only the **current viewport**, following the same behavior as the Web nodes.

```javascript
await web.screenshot("current-page");
```

To capture the full page:

```javascript
await web.screenshot("full-page", { fullPage: true });
```

### `web.run(...)`

`web.run(...)` is the recommended way to execute advanced code against the Web session. The callback receives:

| Field | Description |
|-------|-------------|
| `page` | Current page |
| `context` | Current context |
| `browser` | Current browser |
| `session` | Current session |
| `sessionId` | Session ID |
| `id` | Internal session ID |
| `storageStrategy` | Storage strategy |
| `persistedKey` | Persistence key |
| `url` | Helper for current URL |
| `title` | Helper for current title |
| `locator` | Helper for locators |
| `screenshot` | Helper for screenshots |
| `expect` | Playwright assertions |
| `assert` | Node.js assertions |

### Web example: validate products and take a screenshot

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

### Web example: use the current session without navigating

```javascript
return await web.run(async ({ page }) => {
  return {
    url: page.url(),
    title: await page.title()
  };
});
```

### Web example: use the global `page` alias

```javascript
await page.goto('https://example.com');
await web.screenshot('example-home');

return {
  title: await page.title(),
  url: page.url()
};
```

---

## Mobile Automation with Appium

The `mobile` namespace exposes the current Mobile session and advanced utilities powered by **Appium**.

### `mobile` namespace methods

| Method | Description |
|--------|-------------|
| `mobile.hasSession(sessionId?)` | Checks whether there is an active Mobile session |
| `mobile.ids()` | Returns the IDs of available Mobile sessions |
| `mobile.sessions()` | Returns all available Mobile sessions |
| `mobile.current()` | Returns the current Mobile session |
| `mobile.session(sessionId)` | Returns a specific Mobile session |
| `mobile.app(sessionId?)` | Returns the main Mobile helper |
| `mobile.client(sessionId?)` | Compatibility alias for `mobile.app()` |
| `mobile.screenshot(name?, sessionId?)` | Captures a screenshot from the Mobile session |
| `mobile.source(sessionId?)` | Returns the current screen XML/source |
| `mobile.run(fn, sessionId?)` | Runs advanced code against the Mobile session |

### Object returned by `mobile.current()` and `mobile.session()`

| Field / Method | Description |
|----------------|-------------|
| `id` | Internal session ID |
| `sessionId` | Public session ID |
| `app` | Main Mobile helper |
| `client` | Compatibility alias for `app` |
| `capabilities` | Session capabilities |
| `session` | Raw session object |
| `screenshot(name?)` | Captures a screenshot from the session |
| `source()` | Returns the screen source/XML |
| `request(method, path, body?)` | Makes a direct call to the Appium session |
| `executeScript(script, args?)` | Executes an Appium script |

### `app` and `client`

The main Mobile name is **`app`**.  
`client` still exists for compatibility, but `app` is the recommended usage.

### Methods exposed in `app`

| Method | Description |
|--------|-------------|
| `createSession(capabilities)` | Creates a new Appium session |
| `deleteSession(sessionId)` | Ends a session |
| `screenshot(sessionId)` | Captures a session screenshot |
| `getPageSource(sessionId)` | Returns the screen XML/source |
| `findElement(sessionId, using, value)` | Finds an element |
| `findElements(sessionId, using, value)` | Finds multiple elements |
| `elementClick(sessionId, elementId)` | Clicks an element |
| `elementSendKeys(sessionId, elementId, text)` | Types into an element |
| `sendKeys(sessionId, text)` | Sends text directly to the session |
| `elementClear(sessionId, elementId)` | Clears a field |
| `elementText(sessionId, elementId)` | Returns element text |
| `elementAttribute(sessionId, elementId, name)` | Returns an element attribute |
| `elementDisplayed(sessionId, elementId)` | Checks whether the element is visible |
| `tapAt(sessionId, x, y)` | Taps at coordinates |
| `doubleTapAt(sessionId, x, y)` | Performs a double tap at coordinates |
| `swipe(sessionId, startX, startY, endX, endY, durationMs?)` | Performs a swipe |
| `pressKeyCode(sessionId, keyCode)` | Presses an Android keycode |
| `req(method, path, body?)` | Makes a raw Appium request |
| `executeScript(sessionId, script, args?)` | Executes an Appium script |
| `launchApp(sessionId)` | Launches the app again |
| `resetApp(sessionId)` | Resets the app |

### Mobile screenshots

```javascript
await mobile.screenshot('mobile-current-screen');
```

### `mobile.run(...)`

`mobile.run(...)` is the recommended way to execute advanced code against the Mobile session. The callback receives:

| Field | Description |
|-------|-------------|
| `app` | Main Mobile helper |
| `client` | Alias for `app` |
| `sessionId` | Session ID |
| `id` | Internal session ID |
| `capabilities` | Current capabilities |
| `session` | Current session |
| `screenshot` | Screenshot helper |
| `source` | Source helper |
| `request` | Direct request helper |
| `executeScript` | Appium script helper |
| `assert` | Node.js assertions |

### Mobile example: tap by coordinates and take a screenshot

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

### Mobile example: click by `accessibility id`

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

### Mobile example: use the global `app` alias

```javascript
const current = mobile.current();
const source = await app.getPageSource(current.sessionId);

return {
  sessionId: current.sessionId,
  sourceLength: source.length
};
```

---

## Returning Values

The code **must return** a value using `return`. The returned value is available in the outputs:

```javascript
// Return a simple value
return "Hello World";

// Return an object
return {
  fullName: steps["Query"].outputs.rows[0].first_name + " " + steps["Query"].outputs.rows[0].last_name,
  isAdmin: steps["Query"].outputs.rows[0].role === "admin"
};

// Return an array
return steps["Query"].outputs.rows.map(row => row.email);
```

### Returning the result from `web.run()` and `mobile.run()`

If you want the value returned inside `web.run(...)` or `mobile.run(...)` to appear in `outputs.result`, the main script must also return that value:

```javascript
const result = await web.run(async ({ page }) => {
  return { url: page.url() };
});

return result;
```

Or directly:

```javascript
return await mobile.run(async ({ app, sessionId }) => {
  await app.tapAt(sessionId, 200, 400);
  return { tapped: true };
});
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `result` | `any` | Value returned by the code |

### Accessing the result

```
{{ steps["custom-js"].outputs.result }}
{{ steps["custom-js"].outputs.result.fullName }}
{{ steps["custom-js"].outputs.result[0] }}
```

---

## Practical Examples

### Transform API data

```javascript
const items = steps["http-request"].outputs.json.items;

// Filter and transform
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

### Generate test data

```javascript
const timestamp = Date.now();
return {
  email: `test-${timestamp}@example.com`,
  username: `user_${timestamp}`,
  password: `Pass${timestamp}!`
};
```

### Complex validation

```javascript
const response = steps["API Check"].outputs.json;
const dbRecord = steps["DB Query"].outputs.rows[0];

const validations = [];

if (response.name !== dbRecord.name) {
  validations.push(`Name mismatch: API="${response.name}" DB="${dbRecord.name}"`);
}

if (response.email !== dbRecord.email) {
  validations.push(`Email mismatch: API="${response.email}" DB="${dbRecord.email}"`);
}

return {
  valid: validations.length === 0,
  errors: validations,
  errorCount: validations.length
};
```

### Format a report

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

### Plain JavaScript

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

## Limitations

- The code runs in a **sandbox environment**
- There is no access to:
  - File system (`fs`)
  - Node.js modules via `require` / `import`
  - `process.env` directly
- For environment variables, use `variables`
- For Web and Mobile interactions, there must already be an active session
- `client` is still supported, but `app` is the recommended Mobile name

> `fetch` is available. Even so, for structured HTTP flows that are easier to maintain, prefer the **HTTP Request** node.

---

## Tips

- Use descriptive node names: "Transform Data", "Generate Credentials", "Validate Response", "Advanced Web Code", "Advanced Mobile Code"
- **Always return** a value — without `return`, the output will be `undefined`
- For debugging, use `console.log()` — messages appear in the node logs
- Monaco Editor offers autocomplete — type `steps.`, `web.`, or `mobile.` to see available options
- For advanced Web automation, prefer `web.run(...)`
- For advanced Mobile automation, prefer `mobile.run(...)`
- Use `web.screenshot()` and `mobile.screenshot()` when you need extra evidence
- Use it to **transform data** between nodes and to solve advanced cases not covered by visual nodes
