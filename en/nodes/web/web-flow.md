# Web Flow Node

The **Web Flow** node allows you to automate interactions with web pages using **CSS selectors, XPath, data-testid, and text**. It is ideal when you need traditional selectors and detailed control over element location.

> **Tip:** If you prefer semantic locators based on accessibility (getByRole, getByLabel, etc.), use the [Smart Locators](smart-locators.md) node.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `web-flow` |
| **Category** | Web |
| **Color** | Blue (#3b82f6) |
| **Input** | `in` |
| **Output** | `out` |

---

## General Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **Session Mode** | `new` / `reuse` | `new` | New session or reuse an existing one |
| **Session ID** | `string` | — | Session ID to reuse (when `reuse`) |
| **Headless** | `boolean` | `true` | Run without a visible graphical interface |
| **Storage Strategy** | `inMemory` / `persisted` | `inMemory` | Persist cookies/localStorage between executions |
| **Storage Key** | `string` | — | Identifier for the persisted storage |
| **Browser** | `chromium` / `firefox` / `webkit` | `chromium` | Browser to use for the execution |
| **Device** | `desktop` / `mobile` | `desktop` | Emulation mode |
| **Viewport** | preset or `custom` | `1920x1080` | Window size (Desktop mode) |
| **Width / Height** | `number` | — | Custom dimensions (when `custom`) |
| **Device Model** | string | `iPhone 14` | Device to emulate (Mobile mode) |

### Browser

| Value | Browser |
|-------|---------|
| `chromium` | Chrome / Chromium (default) |
| `firefox` | Firefox |
| `webkit` | Safari / WebKit |

> Firefox and WebKit are downloaded automatically on first use if not already installed.

### Device — Desktop

Available viewport presets:

| Preset | Resolution |
|--------|------------|
| Full HD (default) | 1920 × 1080 |
| 2K | 2560 × 1440 |
| 4K | 3840 × 2160 |
| Laptop | 1440 × 900 |
| HD | 1366 × 768 |
| WXGA | 1280 × 720 |
| XGA | 1024 × 768 |
| Custom | free width and height |

### Device — Mobile

Uses real Playwright device profiles (viewport, user agent, pixel scale ratio, touch). Available models:

**iOS:** iPhone SE, iPhone 12, iPhone 13, iPhone 14, iPhone 14 Pro, iPhone 15, iPhone 15 Pro, iPad Mini, iPad (gen 9), iPad Pro 11

**Android:** Pixel 5, Pixel 7, Galaxy S8, Galaxy S9+, Galaxy Tab S4

> When using Firefox with Mobile mode, the `isMobile` and `hasTouch` options are automatically stripped (not supported by Firefox in Playwright). Viewport and user agent from the device profile are still applied.

### Session Mode

- **New Session (`new`)**: Opens a new browser for each execution. Ideal for isolated tests.
- **Reuse Session (`reuse`)**: Uses a browser session already opened by another Web Flow/Smart Locators node. Useful for splitting long tests into multiple nodes while keeping the same browser and cookies.

### Storage Strategy

- **In Memory (`inMemory`)**: Cookies and localStorage are discarded at the end of the execution.
- **Persisted (`persisted`)**: Cookies and localStorage are saved with a key and can be reused in future executions. Ideal for maintaining login sessions between executions.

---

## Steps (Web Steps)

The Web Flow node executes a sequence of configurable **steps**. Each step represents an action in the browser.

### Available Actions

| Action | Color | Description |
|--------|-------|-------------|
| [navigate](#navigate) | Blue | Navigate to a URL |
| [click](#click) | Yellow | Click on an element |
| [type](#type) | Green | Type text into a field |
| [wait](#wait) | Purple | Wait for a condition or time |
| [assert](#assert) | Red | Verify a condition on the page |
| [extract](#extract) | Cyan | Extract data from an element |
| [hover](#hover) | Cyan | Hover the mouse over an element |
| [scroll](#scroll) | Yellow | Scroll the page |
| [refresh](#refresh) | Orange | Reload the page |
| [select](#select) | Purple | Select an option from a dropdown |
| [press-key](#press-key) | Green | Press a key |
| [frame](#frame) | Gray | Switch between frames/iframes |

---

## Selector Strategies

Web Flow uses **selector strategies** to locate elements on the page. Each step can have multiple strategies ordered by priority — if the first one does not find the element, the next one is tried.

| Strategy | Description | Example |
|----------|-------------|---------|
| `css` | CSS selector | `#login-button`, `.submit-btn`, `[data-testid="login"]` |
| `dataTestId` | data-testid attribute | `login-button` |
| `role` | ARIA role | `button` |
| `text` | Visible text | `Sign In` |
| `xpath` | XPath expression | `//button[@type="submit"]` |

**Recommended priority:**
1. `dataTestId` — Most stable, does not change with layout
2. `css` with ID — Usually unique on the page
3. `role` — Accessibility-based
4. `text` — May change with translations
5. `xpath` — Last resort, most fragile

---

## Action Details

### navigate

Navigates to a specific URL.

| Field | Type | Description |
|-------|------|-------------|
| **URL** | `string` | Destination URL (supports `{{ }}` expressions) |

**Example:**
```
URL: https://mysite.com/login
URL: {{ variables.BASE_URL }}/dashboard
```

---

### click

Clicks on a page element.

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Selector strategies |
| **Retries** | `number` | Number of attempts (default: 3) |

The click tries to locate the element using each strategy in order. If it fails, it waits and tries again (up to the number of retries).

---

### type

Types text into an input field.

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Selector strategies |
| **Text** | `string` | Text to type (supports `{{ }}`) |
| **Clear First** | `boolean` | Clears the field before typing |

> **Note:** The `type` action uses Playwright's `fill()`, which replaces the field value instantly. To simulate character-by-character typing, use the [Smart Locators](smart-locators.md) node with the `type` action (pressSequentially).

---

### wait

Waits for a condition before proceeding.

| Field | Type | Description |
|-------|------|-------------|
| **Mode** | `string` | Wait type |
| **Timeout (ms)** | `number` | Maximum wait time |
| **Selectors** | `array` | Selectors (mode `selectorVisible`) |

**Modes:**

| Mode | Description |
|------|-------------|
| `networkIdle` | Waits for all network requests to finish |
| `selectorVisible` | Waits for an element to become visible |
| `timeout` | Waits for a fixed amount of time (ms) |

---

### assert

Verifies a condition on the page. If the condition fails, the step is marked as failed.

| Field | Type | Description |
|-------|------|-------------|
| **Name** | `string` | Assertion identifier |
| **Mode** | `string` | Verification type |
| **Selectors** | `array` | Element selectors |
| **Expected Text** | `string` | Expected value |
| **Case Sensitive** | `boolean` | Case-sensitive comparison |
| **Continue on Failure** | `boolean` | Do not fail the entire node |

**Modes:**

| Mode | Description |
|------|-------------|
| `elementExists` | Checks if the element exists on the page |
| `textContains` | Checks if the element's text contains the expected value |
| `textEquals` | Checks if the element's text is exactly the expected value |

Assertion results are available in the outputs:
```
{{ steps["web-flow"].outputs.asserts.assertionName }}  →  true or false
```

---

### extract

Extracts data from a page element.

| Field | Type | Description |
|-------|------|-------------|
| **Name** | `string` | Extraction name (key in the output) |
| **Selectors** | `array` | Element selectors |
| **Attribute** | `string` | What to extract |

**Attributes:**

| Attribute | Description |
|-----------|-------------|
| `text` | Visible text of the element |
| `innerHTML` | Inner HTML |
| `href` | URL of links |
| `src` | URL of images |
| `value` | Form field value |

Extracted data is available in the outputs:
```
{{ steps["web-flow"].outputs.extracts.extractionName }}
```

---

### hover

Hovers the mouse over an element.

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Element selectors |
| **Retries** | `number` | Number of attempts |
| **Wait After (ms)** | `number` | Wait time after hover |

---

### scroll

Scrolls the page or to an element.

| Field | Type | Description |
|-------|------|-------------|
| **Mode** | `string` | Scroll type |
| **Pixels** | `number` | Number of pixels (mode `by`) |
| **Wait After (ms)** | `number` | Wait time after scrolling |
| **Selectors** | `array` | Selectors (mode `toSelector`) |

**Modes:**

| Mode | Description |
|------|-------------|
| `toSelector` | Scrolls until the element is visible |
| `by` | Scrolls a fixed number of pixels |
| `toBottom` | Scrolls to the bottom of the page |

---

### refresh

Reloads the current page.

| Field | Type | Description |
|-------|------|-------------|
| **Wait Until** | `string` | Load event |
| **Wait After (ms)** | `number` | Additional time after reloading |

**Events:**

| Event | Description |
|-------|-------------|
| `load` | Page fully loaded |
| `domcontentloaded` | DOM ready (faster) |
| `networkidle` | No pending network requests |

---

### select

Selects an option from a `<select>` element (dropdown).

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Dropdown selectors |
| **Select By** | `string` | Selection criterion |
| **Value** | `string` | Value matching the criterion |

**Criteria:**

| Criterion | Description |
|-----------|-------------|
| `value` | By the option's `value` attribute |
| `label` | By the option's visible text |
| `index` | By the option's index (position) |

---

### press-key

Presses a keyboard key.

| Field | Type | Description |
|-------|------|-------------|
| **Key** | `string` | Key name (e.g.: `Enter`, `Tab`, `Escape`) |
| **Selectors** | `array` | Focused element selectors (optional) |
| **Wait After (ms)** | `number` | Wait time after pressing |

---

### frame

Switches context to an iframe.

| Field | Type | Description |
|-------|------|-------------|
| **Mode** | `string` | Frame selection method |
| **Selectors** | `array` | Selectors (mode `selector`) |
| **Frame Name** | `string` | Name (mode `name`) |
| **Timeout (ms)** | `number` | Maximum wait time |

**Modes:**

| Mode | Description |
|------|-------------|
| `selector` | Locates the iframe by CSS selector |
| `name` | Locates the iframe by its name attribute |
| `main` | Returns to the main context (outside the iframe) |

---

## Evidence (Screenshots)

Each step can have individual evidence configuration:

| Field | Type | Description |
|-------|------|-------------|
| **Capture Screenshot** | `boolean` | Enable capture |
| **Mode** | `before` / `after` / `both` | When to capture |
| **Name Template** | `string` | Custom file name |
| **Wait for Load** | `string` | Event before capturing |
| **Delay (ms)** | `number` | Additional time before capture |

---

## Accessibility Testing

Web Flow supports automatic accessibility scanning with built-in **axe-core**. When enabled, the node audits the page after each step that captures a screenshot, identifying WCAG/ARIA violations and visually marking the affected elements.

### Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **Accessibility Scan** | `boolean` | `false` | Enables axe-core scanning |
| **Fail When Severity >=** | `none` / `minor` / `moderate` / `serious` / `critical` | `serious` | Minimum severity level that fails the node |

### How It Works

After each step that takes a screenshot, axe-core is injected into the page and scans the document for violations. Violations are drawn over the screenshot as colored boxes around the affected elements, plus a summary panel in the corner:

| Severity | Color | Description |
|----------|-------|-------------|
| **Critical** | 🔴 Red | Blocks access for users with disabilities |
| **Serious** | 🟠 Orange | Causes significant difficulty |
| **Moderate** | 🟡 Yellow | Causes some difficulty |
| **Minor** | 🔵 Blue | Recommended improvement |

At the end of execution, the following are generated automatically:

- **Per-step screenshots** — overlay with colored boxes + panel `Critical N / Serious N / Moderate N / Minor N` + top 2 rules
- **Severity chart** — `{id}-accessibility-severity-chart.png` — distribution by level
- **Rules chart** — `{id}-accessibility-rule-chart.png` — top 8 most violated rules
- **JSON report** — `{id}-accessibility-report.json` — full data for all violations

### Pass/Fail Criteria

The node fails if there is any violation with severity **equal to or above** the configured threshold. Use `none` to never fail (collect metrics without blocking the flow).

### Example: Login audit

```
Node configuration:
  Accessibility Scan: true
  Fail When Severity >= : serious

Steps:
1. navigate → https://mysite.com/login    (no screenshot = no scan)
2. type → #email → "{{ variables.EMAIL }}" → screenshot after → scan executed
3. click → button[type="submit"]           → screenshot after → scan executed
4. wait → networkIdle
5. assert → loginOk → hasURL → /dashboard → screenshot after → scan executed
```

If any step has a `serious` or `critical` violation, the node fails with:
> *"Accessibility findings at or above "serious" were detected."*

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `sessionId` | `string` | Browser session ID |
| `extracts` | `object` | Extracted data (key → value) |
| `asserts` | `object` | Assertion results (key → boolean) |
| `accessibilityPassed` | `boolean` | Whether it passed the severity criteria (when enabled) |
| `accessibilityViolationCount` | `number` | Total violation instances found |
| `accessibility` | `object` | Full accessibility report |
| `accessibility.threshold` | `string` | Configured threshold |
| `accessibility.scanCount` | `number` | Number of checkpoints scanned |
| `accessibility.counts` | `object` | `{ minor, moderate, serious, critical }` — aggregated totals |
| `accessibility.steps` | `array` | Per-checkpoint details (url, counts, topRules) |
| `accessibility.rules` | `array` | Top 10 violated rules across the entire flow |

---

## Complete Example

### Logging into a website

Steps configured in the node:

1. **navigate** → `https://mysite.com/login`
2. **type** → Selector: `#email`, Text: `{{ variables.USER_EMAIL }}`
3. **type** → Selector: `#password`, Text: `{{ variables.USER_PASSWORD }}`
4. **click** → Selector: `button[type="submit"]`
5. **wait** → Mode: `networkIdle`
6. **assert** → Name: `loginOk`, Mode: `textContains`, Selector: `.welcome`, Text: `Welcome`
7. **extract** → Name: `userName`, Selector: `.user-name`, Attribute: `text`

**Result:** After execution, the outputs will be:
```json
{
  "sessionId": "abc-123",
  "extracts": { "userName": "João Silva" },
  "asserts": { "loginOk": true }
}
```

---

## Tips

- Use **data-testid** as the primary selector strategy — it is more stable
- Configure **screenshots** on critical steps to make debugging easier
- Use **session reuse** when multiple Web Flow/Smart Locators nodes need to share the same browser
- For forms with autocomplete or input masks, consider using the [Smart Locators](smart-locators.md) node with the `type` action (character-by-character typing)
- **headless: false** mode is useful during development to visualize what the test is doing
