# Chrome Extension — QANode Recorder

The **QANode Recorder** is a Google Chrome extension that **records your interactions** with web pages and automatically converts them into test steps compatible with QANode's **Web Flow** or **Smart Locators** nodes.

---

## Installation

1. Open Chrome and go to `chrome://extensions`
2. Enable **Developer mode** (toggle in the upper right corner)
3. Click **Load unpacked**
4. Select the `extension/` folder from QANode
5. The extension will appear in the toolbar

---

## Recorder Mode

Before starting the recording, choose the **recorder mode** in the extension popup:

| Mode | Selectors | When to Use |
|------|-----------|-------------|
| **Web Flow** (default) | CSS / XPath | Sites with `data-testid`, `id`, or stable CSS selectors |
| **Smart Locators** | Semantic (getByRole, getByLabel, etc.) | Sites with good accessibility practices |

Each mode generates a node of the same name on the canvas when pasting the JSON.

> **Note:** You cannot switch modes during recording. If there are recorded steps, switching modes will clear them.

For more details about the differences between the two nodes, see:
- [Web Flow](../nodes/web/web-flow.md)
- [Smart Locators](../nodes/web/smart-locators.md)

---

## Recording Modes

The extension operates in three interaction modes during recording:

### REC — Normal Recording

Automatically records your interactions:

| Interaction | Generated Action |
|-------------|-----------------|
| **Click** on element | `click` with element selectors |
| **Type** in field | `type` with text and selectors |
| **Select** option | `select` with selected value |
| **Scroll** the page | `scroll` with position |
| **Navigate** to URL | `navigate` with URL |
| **Reload** page | `refresh` |

### CTRL+A — Assert Mode

Captures elements to create **assertions** (verifications):

1. Press **Ctrl+A** to activate assert mode
2. Click on the element to verify
3. An `assert` step is created with the element's text/value

### CTRL+E — Extract Mode

Captures elements for **data extraction**:

1. Press **Ctrl+E** to activate extract mode
2. Click on the element to extract
3. An `extract` step is created capturing the text/value

---

## How to Use

### Recording a Test

1. Click the extension icon in the toolbar
2. Select the **recorder mode** (Web Flow or Smart Locators)
3. Click **Start** to begin recording
4. The indicator turns red indicating active recording
5. Browse and interact with the site normally
6. Use **Ctrl+A** to add verifications
7. Use **Ctrl+E** to add extractions
8. Click the icon again and click **Stop** to finish

[Extension popup](../../assets/images/web-recorder.mp4)
*Image: Extension popup showing REC/STOP buttons, list of recorded steps, and Copy JSON button*

### Using in QANode

1. In the extension popup, click **Copy JSON**
2. In QANode, open the flow editor
3. Press **Ctrl+V** on the canvas
4. A node will be created automatically according to the selected mode:
   - **Web Flow** mode → Web Flow node with `selectorStrategies` (CSS/XPath)
   - **Smart Locators** mode → Smart Locators node with `locator` (getByRole, getByLabel, etc.)
5. Review and adjust steps as needed

---

## Selector Strategies

### Web Flow Mode

In Web Flow mode, the extension automatically generates **multiple CSS/XPath selectors** for each element, ordered by priority:

| Priority | Strategy | Example |
|----------|----------|---------|
| 1 | `data-testid` | `[data-testid="login-btn"]` |
| 2 | `id` | `#submit-button` |
| 3 | `data-qa` | `[data-qa="email-input"]` |
| 4 | `name` (inputs) | `input[name="email"]` |
| 5 | `aria-label` | `[aria-label="Enviar"]` |
| 6 | `placeholder` | `input[placeholder="E-mail"]` |
| 7 | `href` (links) | `a[href="/login"]` |
| 8 | Text (buttons/links) | `button:has-text("Entrar")` |
| 9 | XPath (fallback) | `//button[@type="submit"]` |

The most specific and stable selector is always prioritized. This ensures more resilient tests.

### Smart Locators Mode

In Smart Locators mode, the extension converts the captured elements into **semantic Playwright locators**:

| Priority | Method | Example |
|----------|--------|---------|
| 1 | `getByTestId` | Element with `data-testid="submit-btn"` |
| 2 | `getByLabel` | Field with `aria-label="E-mail"` |
| 3 | `getByPlaceholder` | Field with `placeholder="Digite seu nome"` |
| 4 | `getByText` | Element with visible text |
| 5 | `getByRole` | Button, link, heading (fallback) |

Semantic locators are more resilient to layout changes and reflect how users actually find elements on the page.

---

## Smart Features

### Deduplication

The extension prevents step duplication:

- **Click + Navigation**: If a click triggers a navigation, the extra `navigate` step is suppressed
- **Type + Type**: Consecutive typing in the same field is consolidated into a single step
- **Scroll**: Small scrolls (< 200px) are ignored

### Forms

- **Focus out**: Saves the input value when the field loses focus
- **Enter**: Saves the input and records the click on the submit button
- **Change**: Saves immediately for dropdowns and selects

### Persistence

The recording survives **page reloads**:
- Steps are saved in `chrome.storage.local`
- On reload, recording continues from where it left off
- A `refresh` step is automatically added

### Navigation Detection

The extension uses the Performance API to distinguish between:
- **Navigation** (new URL) → generates `navigate` step
- **Refresh** (same URL) → generates `refresh` step
- **Click → Navigation** (clicked link) → suppresses the redundant `navigate`

---

## Export Format

The copied JSON is compatible with the node corresponding to the selected mode.

### Web Flow → Web Flow Node

```json
{
  "type": "web-flow",
  "position": { "x": 100, "y": 100 },
  "data": {
    "label": "Recorded Flow",
    "sessionMode": "new",
    "headless": true,
    "storageStrategy": "inMemory",
    "webSteps": [
      {
        "id": "step-1",
        "action": "navigate",
        "label": "Navigate to page",
        "config": {
          "url": "https://exemplo.com/login"
        },
        "evidence": {
          "takeScreenshot": true,
          "screenshotMode": "after"
        }
      },
      {
        "id": "step-2",
        "action": "type",
        "label": "Type \"user@email.com\"",
        "config": {
          "selectorStrategies": [
            { "type": "css", "value": "#email" },
            { "type": "css", "value": "input[name='email']" }
          ],
          "text": "user@email.com",
          "clearFirst": true
        }
      },
      {
        "id": "step-3",
        "action": "click",
        "label": "Click \"Entrar\"",
        "config": {
          "selectorStrategies": [
            { "type": "css", "value": "[data-testid='login-btn']" },
            { "type": "css", "value": "button[type='submit']" }
          ]
        }
      }
    ]
  }
}
```

### Smart Locators → Smart Locators Node

```json
{
  "type": "smart-locators",
  "position": { "x": 100, "y": 100 },
  "data": {
    "label": "Recorded Smart Locators",
    "sessionMode": "new",
    "headless": true,
    "storageStrategy": "inMemory",
    "smartSteps": [
      {
        "id": "step-1",
        "action": "navigate",
        "label": "Navigate to page",
        "config": {
          "url": "https://exemplo.com/login"
        },
        "evidence": {
          "takeScreenshot": true,
          "screenshotMode": "after"
        }
      },
      {
        "id": "step-2",
        "action": "fill",
        "label": "Fill \"user@email.com\"",
        "config": {
          "locator": {
            "method": "getByLabel",
            "value": "E-mail",
            "exact": true
          },
          "text": "user@email.com",
          "clearFirst": false
        }
      },
      {
        "id": "step-3",
        "action": "click",
        "label": "Click \"Entrar\"",
        "config": {
          "locator": {
            "method": "getByRole",
            "value": "button",
            "name": "Entrar",
            "exact": false
          }
        }
      }
    ]
  }
}
```

> **Key differences:** In Smart Locators mode, the `type` action is converted to `fill`, `select` to `selectOption`, and CSS selectors are replaced by semantic locators (`locator` instead of `selectorStrategies`).

---

## Recommended Workflow

```
1. Choose the recorder mode (Web Flow or Smart Locators)
2. Record the interaction with the extension
3. Paste into QANode (Ctrl+V)
4. Review the generated steps
5. Add waits where necessary (the extension does not record waits)
6. Add asserts for validations (or record with Ctrl+A)
7. Adjust selectors/locators if necessary
8. Configure screenshots for critical steps
9. Save and execute
```

---

## Limitations

- **Chrome only** — does not work in other browsers
- **No automatic waits** — add them manually in QANode
- **No drag & drop** — does not record drag operations
- **Sites with strict CSP** — may block content script injection
- **Cross-origin iFrames** — may not be accessible

---

## Tips

- **Choose the right mode** — use Smart Locators for accessible sites, Web Flow for sites with stable CSS selectors
- **Record simple actions** and refine in QANode
- **Use Ctrl+A** (assert) during recording at verification checkpoints
- **Use Ctrl+E** (extract) to capture data that will be used in subsequent nodes
- **Add waits** in QANode for elements that take time to load
- **Review the selectors** — in Web Flow mode, prefer `data-testid`; in Smart Locators mode, prefer `getByRole` or `getByLabel`
- **Record in segments** — for long flows, record in parts and combine in QANode
