# Smart Locators Node

The **Smart Locators** node allows you to automate interactions with web pages using **Playwright's semantic locators** — based on ARIA roles, labels, placeholders, and other accessibility attributes. This approach is more resilient than CSS selectors, as it reflects how users actually find elements on the page.

> **Tip:** If you prefer traditional CSS selectors and XPath, use the [Web Flow](web-flow.md) node.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `smart-locators` |
| **Category** | Web |
| **Color** | Blue (#3b82f6) |
| **Input** | `in` |
| **Output** | `out` |

---

## General Configuration

Identical to the Web Flow node:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **Session Mode** | `new` / `reuse` | `new` | New session or reuse an existing one |
| **Session ID** | `string` | — | Session ID to reuse |
| **Headless** | `boolean` | `true` | Run without a visible graphical interface |
| **Storage Strategy** | `inMemory` / `persisted` | `inMemory` | Persist cookies/localStorage |
| **Storage Key** | `string` | — | Identifier for the persisted storage |
| **Browser** | `chromium` / `firefox` / `webkit` | `chromium` | Browser to use for the execution |
| **Device** | `desktop` / `mobile` | `desktop` | Emulation mode |
| **Viewport** | preset or `custom` | `1920x1080` | Window size (Desktop mode) |
| **Width / Height** | `number` | — | Custom dimensions (when `custom`) |
| **Device Model** | string | `iPhone 14` | Device to emulate (Mobile mode) |

> The Browser & Device options are identical to the Web Flow node — see the [Web Flow documentation](web-flow.md#browser) for details on viewport presets and available mobile devices.

---

## Locator System

### Location Methods

Instead of CSS selectors, Smart Locators uses Playwright's **semantic methods**:

| Method | Description | Example |
|--------|-------------|---------|
| `getByRole` | Locates by ARIA role | Button "Sign In", Link "Home" |
| `getByText` | Locates by visible text | Text "Welcome" |
| `getByLabel` | Locates by associated label | Field with label "Email" |
| `getByPlaceholder` | Locates by placeholder | Field with placeholder "Enter your name" |
| `getByAltText` | Locates by alt text | Image with alt "Logo" |
| `getByTitle` | Locates by title | Element with title "Close" |
| `getByTestId` | Locates by data-testid | `data-testid="submit-btn"` |

### Locator Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Method** | `string` | Location method |
| **Value** | `string` | Text/value to search for |
| **Name** | `string` | Role name (only for `getByRole`) |
| **Exact** | `boolean` | Exact match (false = contains) |
| **Nth** | `number` | Element index (when there are multiple) |

### Available ARIA Roles

For the `getByRole` method, the following roles are available:

| Interaction Roles | Structure Roles | Feedback Roles |
|-------------------|-----------------|----------------|
| `button` | `heading` | `alert` |
| `checkbox` | `list` | `alertdialog` |
| `combobox` | `listitem` | `dialog` |
| `link` | `navigation` | `progressbar` |
| `listbox` | `region` | `status` |
| `menu` | `row` | `tooltip` |
| `menuitem` | `cell` | — |
| `option` | `grid` | — |
| `radio` | `tab` | — |
| `searchbox` | `tablist` | — |
| `slider` | `tabpanel` | — |
| `spinbutton` | `toolbar` | — |
| `switch` | `tree` | — |
| `textbox` | `treeitem` | — |

**Usage example:**
```
Method: getByRole
Value: button
Name: Sign In
```

Equivalent in Playwright: `page.getByRole('button', { name: 'Sign In' })`

---

## Available Actions

| Action | Color | Description |
|--------|-------|-------------|
| [navigate](#navigate) | Blue | Navigate to URL |
| [click](#click) | Yellow | Click on element |
| [dblclick](#dblclick) | Yellow | Double click |
| [fill](#fill) | Green | Fill field (instant) |
| [type](#type) | Green | Type character by character |
| [check](#check) | Green | Check checkbox/radio |
| [uncheck](#uncheck) | Green | Uncheck checkbox |
| [selectOption](#selectoption) | Purple | Select dropdown option |
| [hover](#hover) | Cyan | Hover mouse over element |
| [focus](#focus) | Light Blue | Focus on element |
| [press](#press) | Green | Press a key |
| [assert](#assert) | Red | Verify condition |
| [extract](#extract) | Cyan | Extract data |
| [wait](#wait) | Purple | Wait for condition |
| [scroll](#scroll) | Yellow | Scroll page |
| [refresh](#refresh) | Orange | Reload page |
| [frame](#frame) | Gray | Switch frame |
| [rightClick](#rightclick) | Yellow | Right-click |
| [upload](#upload) | Blue | Upload files |
| [dialog](#dialog) | Purple | Handle dialogs (alert/confirm) |
| [dragDrop](#dragdrop) | Blue | Drag and drop |
| [evaluate](#evaluate) | White | Execute JavaScript on the page |
| [screenshot](#screenshot) | Blue | Capture screenshot |

---

## Action Details

### navigate

Navigates to a URL.

| Field | Type | Description |
|-------|------|-------------|
| **URL** | `string` | Destination URL (supports `{{ }}`) |

---

### click

Clicks on a semantically located element.

| Field | Type | Description |
|-------|------|-------------|
| **Locator** | `locator` | Locator configuration |
| **Retries** | `number` | Number of attempts (default: 3) |

The click waits for the element to be visible, scrolls to it if necessary, and then clicks. If it fails, it retries.

**Example:**
```
Method: getByRole
Value: button
Name: Submit
Retries: 3
```

---

### dblclick

Double-clicks on an element.

| Field | Type | Description |
|-------|------|-------------|
| **Locator** | `locator` | Locator configuration |
| **Retries** | `number` | Number of attempts |

---

### fill

Fills an input field **instantly** (replaces the value).

| Field | Type | Description |
|-------|------|-------------|
| **Locator** | `locator` | Locator configuration |
| **Text** | `string` | Text to fill (supports `{{ }}`) |
| **Clear First** | `boolean` | Clears the field first |

Uses Playwright's `locator.fill()` — ideal for simple fields.

---

### type

Types text **character by character**, simulating real typing.

| Field | Type | Description |
|-------|------|-------------|
| **Locator** | `locator` | Locator configuration |
| **Text** | `string` | Text to type |
| **Delay (ms)** | `number` | Interval between keys (default: 50ms) |
| **Clear First** | `boolean` | Clears the field first |

Uses Playwright's `locator.pressSequentially()` — ideal for fields with:
- **Autocomplete** (suggestions appear as you type)
- **Input masks** (phone numbers, zip codes)
- **On-keypress validation** (validates each key)

**Example:**
```
Locator: getByPlaceholder → "Phone"
Text: 123.456.789-00
Delay: 100ms
```

---

### check / uncheck

Checks or unchecks a checkbox/radio button.

| Field | Type | Description |
|-------|------|-------------|
| **Locator** | `locator` | Locator configuration |

**Example:**
```
Method: getByLabel
Value: I accept the terms of use
```

---

### selectOption

Selects an option from a dropdown (`<select>`).

| Field | Type | Description |
|-------|------|-------------|
| **Locator** | `locator` | Dropdown configuration |
| **Select By** | `string` | `value`, `label`, or `index` |
| **Value** | `string` | Corresponding value |

---

### hover

Hovers the mouse over an element.

| Field | Type | Description |
|-------|------|-------------|
| **Locator** | `locator` | Locator configuration |
| **Retries** | `number` | Number of attempts |
| **Wait After (ms)** | `number` | Wait time after hover |

---

### focus

Sets focus on an element.

| Field | Type | Description |
|-------|------|-------------|
| **Locator** | `locator` | Locator configuration |

---

### press

Presses a keyboard key in the context of an element.

| Field | Type | Description |
|-------|------|-------------|
| **Locator** | `locator` | Element that will receive the key |
| **Key** | `string` | Key name (`Enter`, `Tab`, `Escape`, etc.) |
| **Wait After (ms)** | `number` | Wait time after pressing |

---

### assert

Verifies conditions on the page. Offers multiple verification modes:

| Field | Type | Description |
|-------|------|-------------|
| **Name** | `string` | Assertion identifier |
| **Locator** | `locator` | Target element (when applicable) |
| **Mode** | `string` | Verification type |
| **Expected Text** | `string` | Expected value |
| **Continue on Failure** | `boolean` | Do not fail the entire node |

**Assert Modes:**

| Mode | Description | Requires Locator |
|------|-------------|-----------------|
| `visible` | Element is visible | Yes |
| `hidden` | Element is hidden | Yes |
| `textContains` | Text contains value | Yes |
| `textEquals` | Text is exactly the value | Yes |
| `hasCount` | Element count | Yes |
| `enabled` | Element is enabled | Yes |
| `disabled` | Element is disabled | Yes |
| `checked` | Checkbox/radio is checked | Yes |
| `hasValue` | Input has specific value | Yes |
| `hasAttribute` | Element has attribute with value | Yes |
| `hasURL` | Page URL contains/equals | No |
| `hasTitle` | Page title contains/equals | No |

**Examples:**

```
// Verify that the "Submit" button is visible
Mode: visible, Locator: getByRole("button", "Submit")

// Verify that message contains "Success"
Mode: textContains, Locator: getByText("Success"), Expected: "Success"

// Verify URL after login
Mode: hasURL, Expected: "/dashboard"

// Verify that field is disabled
Mode: disabled, Locator: getByLabel("Email")
```

---

### extract

Extracts data from an element.

| Field | Type | Description |
|-------|------|-------------|
| **Name** | `string` | Extraction name |
| **Locator** | `locator` | Locator configuration |
| **Attribute** | `string` | What to extract |

**Available attributes:**

| Attribute | Description |
|-----------|-------------|
| `text` | Visible text (`textContent`) |
| `innerText` | Inner text |
| `innerHTML` | Inner HTML |
| `inputValue` | Input field value |
| `href` | URL of links |
| `src` | URL of images |
| `value` | Generic value |

---

### wait

Waits for a condition.

| Mode | Description |
|------|-------------|
| `timeout` | Waits for a fixed time |
| `visible` | Waits for element to become visible |
| `hidden` | Waits for element to disappear |
| `networkIdle` | Waits for network requests to finish |

---

### scroll

Scrolls the page.

| Mode | Description |
|------|-------------|
| `toLocator` | Scrolls to the element |
| `by` | Scrolls a fixed number of pixels |
| `toBottom` | Scrolls to the bottom of the page |

---

### refresh

Reloads the page. Same configuration as Web Flow.

---

### frame

Switches to an iframe. Supports `name` and `main` modes (return to main context).

---

## Evidence (Screenshots per Step)

Identical to Web Flow — each step can have individual screenshot capture configuration.

---

## Accessibility Testing

Smart Locators supports automatic accessibility scanning with built-in **axe-core**. When enabled, the node audits the page after each step that captures a screenshot, identifying WCAG/ARIA violations and visually marking the affected elements.

> Smart Locators already uses semantic locators based on ARIA roles — combining it with accessibility scanning is especially useful for verifying that the elements you interact with pass accessibility criteria.

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

### Example: Verify login form accessibility

```
Node configuration:
  Accessibility Scan: true
  Fail When Severity >= : serious

Steps:
1. navigate → https://mysite.com/login            (no screenshot = no scan)
2. fill → getByLabel("Email") → "..."              → screenshot after → scan executed
3. fill → getByLabel("Password") → "..."           → screenshot after → scan executed
4. click → getByRole("button", { name: "Sign In" }) → screenshot after → scan executed
5. wait → networkIdle
6. assert → "loginOk" → textContains → getByText("Welcome") → screenshot → scan executed
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

## Complete Example: Login with Smart Locators

```
Step 1: navigate → https://mysite.com/login
Step 2: fill → getByLabel("Email") → "{{ variables.EMAIL }}"
Step 3: fill → getByLabel("Password") → "{{ variables.PASSWORD }}"
Step 4: click → getByRole("button", { name: "Sign In" })
Step 5: wait → networkIdle
Step 6: assert → "loginOk" → textContains → getByText("Welcome")
Step 7: extract → "userName" → getByRole("heading") → text
```

---

## When to Use Smart Locators vs Web Flow

| Scenario | Recommendation |
|----------|---------------|
| Site with good accessibility practices | **Smart Locators** |
| Site with data-testid on all elements | **Web Flow** |
| Forms with labels | **Smart Locators** |
| Elements without text or role | **Web Flow** (CSS/XPath) |
| Accessibility testing | **Smart Locators** |
| Legacy / sites without semantics | **Web Flow** |

---

## Tips

- **Prefer `getByRole`** — it is the most resilient and reflects how assistive technologies find elements
- **Use `getByLabel`** for form fields — it is more stable than CSS
- **`getByTestId`** is great for elements without a clear role/label
- Use the **type** action (instead of fill) for fields with masks or autocomplete
- The **Nth** field allows you to select a specific element when multiple ones match the locator
