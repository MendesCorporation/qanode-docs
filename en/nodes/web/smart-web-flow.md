# Smart Web Flow Node

The **Smart Web Flow** node is the recommended web automation node in QANode. It combines Chrome extension recording, semantic locators, CSS/XPath selectors, page context, expected-effect checks, evidence, and conservative recovery mechanisms to run scenarios in modern web applications.

It was designed for enterprise systems and dynamic pages such as CRMs, ERPs, ServiceNow, Salesforce, SAP, Jira, internal portals, applications with iframes, menus, grids, kanban boards, rich components, and interfaces that change parts of the DOM between runs.

> **Recommendation:** For new web scenarios, prefer **Smart Web Flow**. [Web Flow](web-flow.md) and [Smart Locators](smart-locators.md) remain available for compatibility and specific use cases.

---

## Overview

[Overview](../../assets/images/nos-smart-web-flow-visao-geral.mp4)

| Property | Value |
|----------|-------|
| **Type** | `smart-web-flow` |
| **Category** | Web |
| **Color** | 🔵 Blue (#3b82f6) |
| **Input** | `in` |
| **Output** | `out` |
| **Recommended recording** | QANode Recorder in Smart Web Flow mode |
| **Element resolution** | Semantic locators + CSS/XPath + target identity + context |
| **Recommended use** | New web flows, dynamic applications, evidence, and assisted recovery |

---

## When to Use

Use **Smart Web Flow** when:

- the flow will be recorded with the Chrome extension;
- the page has menus, dropdowns, modals, iframes, lists, tables, grids, or cards;
- elements may move between executions;
- screenshots by step are important;
- you want automatic checks after important actions;
- you want improvement suggestions after successful runs;
- the same flow needs to run with dynamic data such as ticket number, customer name, card, item, or record.

Consider another node when:

- you already have a simple legacy Web Flow that is stable;
- you need a very specific manual CSS/XPath selector and do not want Smart Web metadata;
- the automation is not web-based, such as API, database, or native mobile.

---

## General Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **Session Mode** | `new` / `reuse` | `new` | Opens a new browser session or reuses an existing one |
| **Session ID** | `string` | — | Name or expression used to reuse a session |
| **Headless** | `boolean` | `true` | Runs the browser without visible UI |
| **Storage Strategy** | `inMemory` / `persisted` | `inMemory` | Controls whether cookies/localStorage are discarded or persisted |
| **Persisted Session ID** | `string` | — | Optional name used to save/reuse cookies and localStorage |
| **Browser** | `chromium` / `firefox` / `webkit` | `chromium` | Browser used during execution |
| **Device** | `desktop` / `mobile` | `desktop` | Desktop viewport or mobile emulation |
| **Viewport** | preset or `custom` | `1920x1080` | Window size in desktop mode |
| **Self Healing** | `boolean` | `true` | Allows conservative recovery when the original target is not found |
| **Accessibility Scan** | `boolean` | `false` | Runs accessibility analysis and stores evidence |
| **Performance Audit** | `boolean` | `false` | Collects navigation, network, and API metrics per page |
| **Capture Browser Console** | `boolean` | `false` | Records console messages, warnings, and page errors during execution |
| **Fail on Console Errors** | `boolean` | `false` | Fails the node when a relevant console error is found |

### Storage and Session

For flows that require login, MFA, or SSO, use persisted storage. This lets QANode reuse cookies and localStorage in future executions without repeating the manual login.

Use `inMemory` for isolated tests and `persisted` for applications that need to keep authentication between executions.

When **Session Mode** is **new** and storage is **persisted**, the persisted session ID is optional:

- if filled, the session is saved with that name;
- if empty, QANode generates an internal key automatically.

Persisted sessions created by the node are available for reuse by other users of the same installation. This keeps shared flows executable even when another person runs the scenario.

Sessions imported by the Chrome extension remain tied to the user who recorded/imported them, because they usually represent personal login, cookies, and permissions.

In **reuse** mode, the **Session ID** field searches named sessions from both scopes. Suggestions show up to 10 sessions and hide automatically generated UUID keys. Sessions unused for 30 days are cleaned automatically.

---

## How Elements Are Found

Each step can carry multiple clues for finding the correct target. Instead of depending on a single selector, Smart Web Flow combines different signals:

| Signal | Example | When it helps |
|--------|---------|---------------|
| **Semantic locator** | `getByRole`, `getByLabel`, `getByText` | Buttons, links, fields, options, and accessible elements |
| **CSS/XPath selectors** | `#email`, `[data-testid="submit"]`, XPath | Pages with stable IDs, attributes, or known structure |
| **Target identity** | text, role, tag, attributes, approximate position | Confirms that the resolved selector still represents the same element |
| **Row/card context** | target reference, parent item, table row | Screens with repeated elements |
| **Control context** | nearby label, expected value, control type | Custom inputs, combos, calendars, and styled checkboxes |
| **Frame/Iframe** | recorded frame context | Applications that render content inside iframes |

The goal is resilience without clicking “anything similar”. When a page is ambiguous, Smart Web Flow tends to fail with diagnostics instead of choosing an unsafe target.

---

## Target Reference

**Target Reference** appears when the step was recorded inside a repeated item, such as:

- a table row;
- a kanban card;
- a list item;
- a search result block;
- a container with many identical buttons.

It tells QANode **which record, row, card, or item should be used as the reference before resolving the target**.

Example:

| Ticket | Customer | Action |
|--------|----------|--------|
| INC0010001 | Apptrix | Open |
| INC0010002 | QANode | Open |

If the target is the **Open** button in row `INC0010002`, the button locator alone is not enough. The target reference can be:

```
INC0010002
```

QANode first finds the row/card containing that text and only then resolves the element inside that context.

Target reference supports expressions:

```
{{ variables.ticketNumber }}
{{ steps["Create ticket"].outputs.extracts.number }}
```

Best practices:

- use a short unique text inside the item, such as number, code, name, or title;
- avoid using the full text of a large card if a smaller part identifies the item;
- for dynamic data, use `{{ }}` instead of leaving the recorded value fixed;
- when possible, ask the development team to expose `data-testid`, `data-qa`, labels, or accessible names.

---

## Strategy Order

In a normal run, Smart Web Flow tries the recorded strategies and metadata in a conservative order:

1. learned preferred strategy, when it was validated by previous successful runs;
2. semantic locators, when applicable;
3. registered CSS/XPath selectors;
4. row/card/control context when the target is repeated or custom;
5. conservative recovery when **Self Healing** is enabled;
6. failure with diagnostics when confidence is not enough.

Users usually do not need to configure this order manually. Review whether the recorded step represents the correct intent: action, target, target reference, data, evidence, and expected effect.

---

## Expected Effects

Some recorded steps have **Expected Effects** generated automatically by the extension or by the engine. They describe what QANode observed as the likely result of an action.

| Effect | What it validates |
|--------|-------------------|
| `domStable` | The page stayed stable for a short period |
| `networkIdle` | Network became idle within the configured limit |
| `urlContains` | URL contains an expected fragment |
| `titleContains` | Page title contains an expected fragment |
| `elementVisible` | A specific selector became visible |
| `targetVisible` | The next expected target became visible |
| `overlayOpened` | A menu, dropdown, modal, or overlay opened |
| `pageOpened` | A new tab/window was opened |

### How to Interpret

In the UI, users do **not** configure each expected effect individually. The available control is the **Use expected effects** toggle, which enables or disables automatic effect validation for that step.

Keep **Use expected effects** enabled when the action must prove that the page changed:

- a click opens a menu;
- a button opens a modal;
- an action navigates to another page;
- an action opens a new tab;
- a hover reveals a submenu;
- a click must make the next field visible.

Disable it for that step when:

- the page is very dynamic and the effect points to volatile content;
- the effect depends on data that always changes;
- the same container opens, but the next real target depends on another step;
- the step already has a more appropriate explicit wait.

For manual control, use `wait` and `assert`. They are better for waits and validations defined directly by the user.

Expected effects recorded by Smart Web Flow start conservatively. When QANode observes the same scenario passing repeatedly with the same target confirmed, it can make that hint stronger. This avoids letting a timing artifact or temporary DOM change break scenarios too early.

---

## Self Healing

When **Self Healing** is enabled, Smart Web Flow can recover a target if the recorded strategies no longer find the element.

It may help when:

- text changed slightly;
- a button moved;
- an ID disappeared, but the element kept the same role;
- a link changed text but kept similar context and destination;
- a field was rendered by another visual component.

It avoids recovery when:

- many candidates are too similar;
- the action may be destructive and the target is unclear;
- the context changed in an incompatible way;
- the best candidate is not clearly better than the second candidate.

When recovery is used, the run shows it as **Self-healing applied**. The user can review and apply it to the flow when it makes sense.

If the recovered selector is already present in the step, QANode suppresses unnecessary “apply optimization” suggestions to avoid asking users to apply something they effectively already have.

---

## Learned Preferred Strategy

After successful runs, Smart Web Flow can learn which strategy resolved the step reliably. It stores success information only when the run passed. Failed runs do not promote or overwrite the preferred strategy.

This helps with resilience: a strategy that is good but sometimes slower is not replaced just because another attempt happened to run after timing changed. When the preferred strategy is tried in a future run, QANode can poll it quickly for its learned average window, then continue with the normal strategy list if it does not resolve.

---

## Actions

| Action | Description |
|--------|-------------|
| `navigate` | Navigate to a URL |
| `switchPage` | Switch to another page/tab/window |
| `click` | Click an element |
| `dblclick` | Double-click an element |
| `fill` | Fill a field instantly |
| `type` | Type text character by character |
| `check` | Check a checkbox |
| `uncheck` | Uncheck a checkbox |
| `selectOption` | Select an option in a native select or compatible control |
| `hover` | Move the mouse over an element |
| `focus` | Focus an element |
| `press` | Press a keyboard key |
| `drag` | Drag and drop an element |
| `wait` | Wait for time, selector, URL, or condition |
| `scroll` | Scroll the page or a scrollable region |
| `refresh` | Reload the current page |
| `clock` | Control browser time for date/time-sensitive scenarios |
| `frame` | Switch or scope execution to an iframe |
| `assert` | Validate text, visibility, URL, title, or value |
| `extract` | Extract a single value |
| `extractList` | Extract repeated items such as rows or cards |
| `extractTable` | Extract a table/grid as structured rows |
| `uploadFile` | Upload a file to a file input |
| `dialog` | Handle browser dialogs |
| `setSlider` | Set a slider/range control value |
| `evaluate` | Run controlled JavaScript in the page |
| `screenshot` | Capture evidence |

---

## Important Actions

### navigate

Navigates to a URL.

| Field | Description |
|-------|-------------|
| **URL** | Target URL, with support for `{{ }}` expressions |
| **Wait Until** | Loading event to wait for |
| **Timeout** | Maximum navigation time |

### switchPage

Switches the active page when a click or action opens another tab/window.

| Mode | Description |
|------|-------------|
| `last` | Uses the last opened tab |
| `next` | Uses the next tab relative to the current one |
| `previous` | Uses the previous tab |
| `match` | Finds a tab by URL and/or title fragment |

The Chrome extension can insert this step automatically when it detects a link that opens a new tab.

### fill and type

Use `fill` for most fields. Use `type` when the application depends on real keyboard events, input masks, autocomplete, or rich text behavior.

For custom editors, Smart Web Flow can target editable regions such as `contenteditable`, textboxes rendered by rich editors, or fields where the extension captured the real typing target.

### selectOption

Selects an option in a native `<select>` or compatible dropdown.

| Field | Description |
|-------|-------------|
| **Select by** | `value`, `label`, or `index` |
| **Value / Label / Index** | Desired option |

For custom dropdowns that are not native `<select>` elements, the extension usually records a sequence of `click`, `press`, or `wait` steps instead of `selectOption`.

In the extension Inspect mode, native selects show their available options in a clickable list. This lets the user choose visually instead of typing the option value manually.

### drag

Drag/drop stores source and target information. When possible, the recorder captures a target container rather than a volatile sibling card, which improves behavior in kanban boards and reorderable lists. If the target is a free area, canvas, or a drop zone that only exists while dragging, review the step manually.

If the extension warns that it could not move the item during recording, still review the step in QANode: on highly customized sites, execution may reproduce the drag/drop even when the extension cannot visually move the card.

### extract

Extracts a single value from the page.

| Field | Description |
|-------|-------------|
| **Name** | Output name |
| **Attribute** | `text`, `inputValue`, `href`, `src`, or a specific attribute |

When the step comes from the extension, Smart Web Flow treats extraction as a dynamic value. The text seen during recording is used as a sample/evidence, but the target should be found by a stable selector, context, or the correct position among equivalent elements.

This helps in screens where the value can change on each run, such as counters, statuses, balances, totals, and indicators.

When the extraction is inside a repeated row, card, or list item, use **Target Reference** to extract only from the correct record.

The extracted value is available at:

```
{{ steps["Node Name"].outputs.extracts.name }}
```

### uploadFile

Uploads a file to an `<input type="file">`.

| Field | Description |
|-------|-------------|
| **File** | `fileRef` to upload |
| **Locator / Strategies** | Target file input |
| **Timeout** | Maximum time to locate the input |

The step is recorded automatically by the extension when the user chooses a file in an upload input. When importing the flow into QANode, if the original file is not attached to the flow yet, the step is highlighted until the user chooses the `fileRef` that will be used during execution.

Use `uploadFile` for scenarios such as submitting documents, attaching receipts, uploading images, PDFs, CSVs, spreadsheets, or testing web forms with upload.

---

## Downloads

Smart Web Flow does not have a separate download action. When a **click** step triggers a browser download, QANode captures the file automatically and exposes it as a `fileRef`.

Downloaded files are available in the `downloads` output and, when there is only one relevant download, can also appear as `fileRef` in the variables panel.

Example:

```
{{ steps["smart-web-flow"].outputs.downloads.report }}
{{ steps["smart-web-flow"].outputs.fileRef }}
```

Use the downloaded `fileRef` in other nodes, such as **Extract File**, **HTTP Request**, **SSH Command**, **Mobile Flow**, or **Custom JavaScript**.

### clock

The `clock` action controls browser time for scenarios that depend on current date/time. It is useful for date pickers, expiration flows, SLA rules, time-based banners, or tests that must be deterministic.

---

## Evidence

Each step can capture screenshots:

| Mode | Behavior |
|------|----------|
| `before` | Captures before the action |
| `after` | Captures after the action |
| `both` | Captures before and after |
| `none` | No screenshot for the step |

Use screenshots for critical points: login, navigation, modal opening, form submission, drag/drop, important assertions, and failures that need visual diagnosis.

---

## Browser Console

When console capture is enabled, Smart Web Flow records messages emitted by the page during execution, such as `console.log`, warnings, JavaScript errors, and browser-reported failures.

These entries appear in the run details and can be downloaded as evidence. Use this feature to investigate screens that look correct visually but still generate frontend errors or browser-side integration failures.

Use **Fail on Console Errors** when a console error should make the test fail. For legacy or noisy systems, first enable capture only to understand the volume of messages before turning console errors into automatic failures.

---

## Outputs

Typical outputs include:

| Output | Description |
|--------|-------------|
| `success` | Whether the node completed successfully |
| `steps` | Step-by-step execution details |
| `extracts` | Values captured by extract actions |
| `tables` | Data captured by extractTable actions |
| `screenshots` | Evidence files/URLs |
| `sessionId` | Browser session ID, useful for reuse |
| `currentUrl` | URL at the end of the node |
| `downloads` | Files downloaded during clicks, as `fileRef` |
| `fileRef` | Shortcut to a downloaded file when applicable |

Access example:

```
{{ steps["smart-web-flow"].outputs.extracts.customerName }}
{{ steps["smart-web-flow"].outputs.tables.orders.rows[0].status }}
{{ steps["smart-web-flow"].outputs.sessionId }}
```

---

## Recommended Workflow

1. Record with QANode Recorder in **Smart Web Flow** mode.
2. Choose the capture mode:
   - **Touch mode** to record by interacting normally with the page;
   - **Inspect mode** to select elements, review suggested actions, and record steps with more control.
3. Execute the flow manually on the site or use the Inspect mode panels to build the actions.
4. Use recording shortcuts when needed: `Ctrl+A` for assert, `Ctrl+E` for extract, `Ctrl+Shift+E` for extract list, `Ctrl+Alt+W` for wait, and `Ctrl+Alt+T` for extract table.
5. Stop the recording or finish it from the Inspect mode panel.
6. Paste the JSON into the QANode canvas.
7. Review the generated actions.
8. Confirm the target reference in repeated rows/cards.
9. Keep expected effects enabled on steps where the screen must change.
10. Add explicit `wait` or `assert` for business-critical validations.
11. Use variables for dynamic data.
12. Execute once and inspect evidence.
13. Apply only the suggestions that match the real user intent.

---

## Best Practices

- Prefer a short, stable target reference for repeated items.
- Use variables in the target reference when the target changes by test data.
- Keep expected effects enabled for menus, modals, page changes, and new tabs.
- Disable expected effects only for volatile steps or when you already added a better explicit wait.
- Use `fill` for normal inputs and `type` for masks, autocomplete, and rich text.
- Use `switchPage` after actions that open new tabs.
- For enterprise applications, ask development teams to expose `data-testid`, `data-qa`, `aria-label`, and accessible names.
- Review drag/drop steps in kanban, virtual lists, and dynamic boards.

---

## Limitations

- Cross-origin iframes may restrict recording or execution.
- Canvas-only controls can require coordinates or custom logic.
- Highly dynamic virtualized lists may need target reference and explicit waits.
- Self Healing is conservative and may fail when the UI is ambiguous.
- Expected effects are generated automatically; the user can enable/disable them, but not configure each internal expected effect from the UI.
