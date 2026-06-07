# Chrome Extension — QANode Recorder

The **QANode Recorder** is a Google Chrome extension that records interactions with web pages and converts them into test steps compatible with QANode's **Smart Web Flow**, **Web Flow**, or **Smart Locators** nodes.

---

## Installation

Install it from the Chrome Web Store:

[QANode Recorder on the Chrome Web Store](https://chromewebstore.google.com/detail/qanode-recorder/gmdklmpafacfkjmljgnoafcjjcdjiajo)

After installing:

1. Pin the QANode Recorder icon in Chrome.
2. Open the website you want to record.
3. Click the extension icon.
4. Choose the recording mode.
5. Click **Start**.

For internal development environments, the extension can also be installed manually through `chrome://extensions` using **Load unpacked**, when instructed by the QANode team.

---

## Recorder Mode

Before recording, choose the **recorder mode** in the extension popup:

| Mode | Strategy | When to Use |
|------|----------|-------------|
| **Smart Web Flow** (recommended) | Semantics + CSS/XPath + identity + context | New web flows, enterprise apps, menus, modals, grids, iframes, drag/drop, and dynamic pages |
| **Web Flow** | CSS / XPath | Legacy flows or sites with very stable `data-testid`, `id`, and CSS selectors |
| **Smart Locators** | Semantic (`getByRole`, `getByLabel`, etc.) | Legacy semantic flows and simpler accessible pages |
| **XPath Inspector** | XPath candidates | Manual selector diagnostics; does not generate a flow node |

Each recording mode creates the matching node on the canvas when the JSON is pasted.

> You cannot switch recording mode while there are recorded steps. Switching mode clears the current recording.

See also:

- [Smart Web Flow](../nodes/web/smart-web-flow.md)
- [Web Flow](../nodes/web/web-flow.md)
- [Smart Locators](../nodes/web/smart-locators.md)

---

## Recording Modes

The extension operates in interaction modes while recording:

### Smart Web Capture: Touch Mode and Inspect Mode

When recorder mode is **Smart Web Flow**, the extension lets you choose how steps are captured:

| Mode | How It Works | When to Use |
|------|--------------|-------------|
| **Touch mode** | You use the site normally and the extension records clicks, fills, scrolls, navigation, drag/drop, and special shortcuts. | Best for recording fast flows that follow the user's manual path. |
| **Inspect mode** | The extension opens draggable helper panels on top of the page. You click an element to inspect it, review suggested actions, and choose the step to record. | Best for building or correcting steps with more precision, especially on dynamic pages, lists, cards, menus, iframes, and similar elements. |

In **Inspect mode**, the page shows two panels:

- **Controls**: shows recorded steps, lets you pause, finish and copy the JSON, delete steps, copy an individual step, and reorder the list.
- **Selected element / Actions**: shows the selected element, the actions available for that target, and captured data such as target identity, strategies, match count, confidence, and ancestors.

When you click an element in Inspect mode, the extension does not automatically assume you want a click step. It first selects the target and shows actions such as **Click**, **Fill**, **Select**, **Upload file**, **Hover**, **Press key**, **Wait visible**, **Assert**, **Extract**, **Extract list**, **Extract table**, or **Drag**, depending on the detected element type.

This helps avoid accidental steps and makes it clearer which locators will be sent to QANode. The strategies shown in the panel are the strategies the extension can send in the step. You can uncheck strategies or ancestors you do not want to use before recording the action.

The panel also shows the match count when possible. Strategies without a useful match are ignored to reduce noise. If a strategy has many matches, prefer a more specific one or review the target before recording the action.

For guided actions:

- **Extract list**: select the repeated item; the extension highlights similar items and then lets you choose the fields to extract inside the selected item.
- **Drag**: select the source item; the extension keeps the source highlighted and asks for the drop target.
- **Press key**: select the target and enter the desired key.
- **Fill**: select the field and enter the value to fill.
- **Select option**: for native selects, the extension shows the available options in a clickable list.
- **Upload file**: for file inputs, the extension records the upload step and asks for the file required to continue recording.

### REC — Normal Recording

Records interactions automatically:

| Interaction | Generated Action |
|-------------|------------------|
| **Click** on element | `click` with target information |
| **Type** in field | `fill`/`type` with text and target |
| **Select** option | `selectOption`/`select` with selected value |
| **Choose file** in upload input | `uploadFile` with the input target |
| **Scroll** page | `scroll` with position |
| **Drag and drop** element | `drag` with source, target, and/or coordinates |
| **Navigate** to URL | `navigate` with URL |
| **Reload** page | `refresh` |

In **Smart Web Flow** mode, recorded actions may also carry target identity, target reference, alternative strategies, iframe context, expected effects, evidence, and session bootstrap data.

When a click opens a new tab/window, the extension can insert a **Switch Page / Tab** (`switchPage`) step after the click so the next steps continue on the correct page.

When a click triggers a browser download, Smart Web Flow captures the file during execution. There is no need to record a separate download action.

When the page opens a file picker, the extension records the upload step and asks for a file during recording so the user can continue the next site steps. When imported into QANode, the step stays highlighted until the user chooses the `fileRef` that will be used during execution.

### Ctrl+A — Assert Mode

Captures elements to create assertions:

1. Press **Ctrl+A**.
2. Click the element to verify.
3. An `assert` step is created with the element text/value.

### Ctrl+E — Extract Mode

Captures a single value:

1. Press **Ctrl+E**.
2. Click the element to extract.
3. An `extract` step is created.

When the element value is dynamic, the extension avoids relying only on the captured text. It prioritizes unique selectors and, when needed, records the item position among visible matches so QANode extracts the future value instead of only searching for the text seen during recording.

### Ctrl+Shift+E — Extract List Mode

Captures repeated elements such as table rows, cards, or list items:

1. Press **Ctrl+Shift+E**.
2. The indicator changes to **LIST:ITEM** — click the repeated element that represents one list item, such as a table row or card.
3. Cyan overlays highlight all detected items with the same pattern.
4. The indicator changes to **LIST:0** — click the fields you want to extract inside each item, such as name column or price column.
5. Each selected field is highlighted in orange and added to the step. The counter increases with each field.
6. Click the **REC indicator** to finish — an `extractList` step is recorded with all selected fields.

Pressing **Ctrl+A** or **Ctrl+E** during list capture cancels the mode and returns to normal recording.

### Ctrl+Alt+W — Wait Mode

Creates an explicit wait for a page element:

1. Press **Ctrl+Alt+W**.
2. Click the element that must be visible before continuing.
3. A `wait` step is created.

Use this when the page has asynchronous loading, delayed components, animations, or elements that depend on API responses.

### Ctrl+Alt+T — Extract Table Mode

Available in **Smart Web Flow** mode. Captures a table or grid as structured tabular data:

1. Press **Ctrl+Alt+T**.
2. Click the table, grid, or tabular region.
3. The recorder creates an `extractTable` step.
4. In QANode, review the output name and row limit if needed.

For cards, visual lists, or highly custom tables, **Extract List** may be better because you choose the fields manually.

### XPath Inspector

XPath Inspector helps investigate selectors directly on the page. It does not record a test flow.

1. Select **XPath Inspector** in the popup.
2. Click **Activate**.
3. Hover the page to see visible elements highlighted.
4. Click the element to inspect.
5. Review generated XPath candidates and uniqueness.
6. Copy the XPath if needed.

The selected element flashes on the page, helping confirm that the XPath points to the correct target.

---

## Recording a Test

1. Click the extension icon in the toolbar.
2. Select the **recorder mode** (Smart Web Flow, Web Flow, or Smart Locators).
3. In **Smart Web Flow**, choose **Touch mode** or **Inspect mode**.
4. Click **Start** to begin recording.
5. In Touch mode, the indicator turns red to show active recording.
6. Navigate and interact with the site normally, or, in Inspect mode, select elements and choose actions in the floating panels.
7. Use **Ctrl+A** to add assertions.
8. Use **Ctrl+E** to add single-value extractions.
9. Use **Ctrl+Shift+E** to add list extractions.
10. Use **Ctrl+Alt+W** to add explicit waits.
11. Use **Ctrl+Alt+T** to add table extraction in Smart Web Flow mode.
12. Click the icon again and click **Stop**, or finish from the Inspect mode panel.

---

## Using in QANode

1. In the extension popup, click **Copy JSON**.
2. In QANode, open the flow editor.
3. Press **Ctrl+V** on the canvas.
4. A node is created according to the selected mode:
   - **Smart Web Flow** → `smartWebSteps`
   - **Web Flow** → `webSteps` with CSS/XPath selector strategies
   - **Smart Locators** → `smartSteps` with semantic locators
5. Review and adjust the generated steps when needed.

---

## Smart Web Flow Metadata

In Smart Web Flow mode, the extension records more than a selector. A step may include:

- primary semantic locator;
- alternative CSS/XPath selectors;
- element identity;
- useful text, role, tag, and attributes;
- row, card, or repeated-item context;
- iframe information;
- approximate visual position;
- expected effects after the action;
- evidence settings.

This lets QANode execute the step with more context and avoids relying only on a fragile selector.

When the extension detects that a target is inside a repeated row, card, or list item, it may record a **Target Reference**. QANode uses it to find the correct item first and only then resolve the button, link, field, or control inside it.

Example:

```
Target Reference: INC0010008
Target: "Open" button
```

---

## Drag and Drop

The extension records **drag and drop** operations in Smart Web Flow, Web Flow, and Smart Locators modes.

During recording, QANode Recorder tries to capture:

- the source element before the movement starts;
- the drop target, when identifiable;
- source and target coordinates as fallback;
- gesture metadata to reproduce the movement more accurately.

In **Smart Web Flow**, recording combines dragged item identity, target identity, visual/structural context, and supporting coordinates when needed. This helps with boards, kanban columns, reorderable lists, and drop areas that change during the gesture.

In some sites, especially enterprise applications with highly customized drag/drop, the browser may block the movement performed by the extension itself during recording. In these cases, the extension warns that it could not visually move the item, but it can still record the step for QANode to execute later.

Review the drag/drop step when:

- the target is a canvas or free area without a clear element;
- several elements have the same text;
- the element is icon-only without `aria-label`;
- the page moves or reorders elements during the gesture;
- the destination is an empty column, virtualized list, or area that only appears while dragging.

For enterprise applications, prefer adding `data-testid`, `data-qa`, `aria-label`, or accessible names to buttons and drop areas.

---

## Smart Features

### Deduplication

The extension avoids duplicate steps:

- **Click + Navigation**: when a click triggers navigation, the extra `navigate` is suppressed.
- **Consecutive typing**: repeated fills in the same field are consolidated.
- **Scroll**: small scrolls are ignored.

### Forms

- **Focus out** saves the input value when a field loses focus.
- **Enter** saves the input and records the submit button click.
- **Change** saves dropdown/select values immediately.

### Persistence

Recording survives page reloads:

- steps are saved in `chrome.storage.local`;
- after reload, recording continues from where it stopped;
- a `refresh` step is added automatically;
- when a step is deleted in the popup, the internal page state is also synchronized so the deleted step does not return.

### Browser Session

When possible, the extension captures cookies and localStorage together with the JSON. When pasted into QANode, this session can be imported as persisted storage, avoiding manual login before the first run.

Review imported sessions when authentication expires quickly, depends on MFA/IP/device, or when the scenario must start without cookies.

### Navigation Detection

The extension uses browser APIs to distinguish:

- **Navigation** to a new URL → `navigate`;
- **Reload** of the same URL → `refresh`;
- **Click → navigation** → suppress redundant `navigate`;
- **Click → new tab** → `switchPage` can be added.

### Frames and iFrames

When the action happens inside an accessible iframe, the extension tries to record the frame context before the action. It may use frame selector, name, and URL so execution can return to the same context.

Cross-origin or very dynamic iframes may still require reviewing the `frame` step in QANode.

---

## Expected Effects

In Smart Web Flow mode, some steps include expected effects such as `domStable`, `targetVisible`, `overlayOpened`, or `pageOpened`.

The UI does not expose individual expected-effect editing. The user controls the step-level **Use expected effects** toggle. Keep it enabled when the action must prove that a page changed, menu opened, modal appeared, tab opened, or next target became visible. Disable it for volatile screens or when a manual `wait`/`assert` is more appropriate.

---

## Export Format

The copied JSON is compatible with the selected node type:

- **Smart Web Flow**: `type: "smart-web-flow"` and `smartWebSteps`;
- **Web Flow**: `type: "web-flow"` and `webSteps`;
- **Smart Locators**: `type: "smart-locators"` and `smartSteps`.

Smart Web Flow JSON may include extra identity, context, expected-effect, and evidence fields. These fields help execution. In the UI, users usually only enable or disable expected-effect usage for the step.

---

## Recommended Workflow

```
1. Choose the recorder mode, preferably Smart Web Flow for new flows
2. Record the interaction with the extension
3. Paste into QANode (Ctrl+V)
4. Review generated steps
5. Review expected effects and target reference when present
6. Add waits where needed
7. Add assertions for validations
8. Adjust selectors/locators if necessary
9. Configure screenshots for critical steps
10. Save and execute
```

---

## Limitations

- **Chrome only** — does not work in other browsers.
- **Not every wait is automatic** — review waits in slow pages, grids, and API-driven loading.
- **Strict CSP sites** may block content script injection.
- **Cross-origin iframes** may not be accessible.

---

## Tips

- Use **Smart Web Flow** for new flows, **Smart Locators** for semantic legacy flows, and **Web Flow** for traditional CSS/XPath selectors.
- Record simple actions and refine in QANode.
- Use **Ctrl+A** for verification points.
- Use **Ctrl+E** for single-value extraction.
- Use **Ctrl+Shift+E** for repeated elements such as tables and card lists.
- Use **Ctrl+Alt+W** when an element must appear before the next step.
- Use **Ctrl+Alt+T** for tables and grids in Smart Web Flow.
- Review **Target Reference** in rows, cards, and lists.
- Prefer `data-testid`, `data-qa`, `aria-label`, and accessible names when you can influence the application.
