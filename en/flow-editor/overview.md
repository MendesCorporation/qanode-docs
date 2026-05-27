# Flow Editor — Overview 

The **Flow Editor** is the heart of QANode. It is where you create, configure, and run your test scenarios visually.

---

## Editor Interface

The editor is divided into four main areas:

[Flow editor overview](../../assets/images/interface-editor-fluxos.mp4)

*Image: Flow editor showing the four areas: top bar, node palette, canvas, and properties panel*

### 1. Top Bar

The top bar contains:

| Element | Description |
|---------|-------------|
| **Back Button** | Returns to the project (shows a warning if there are unsaved changes) |
| **Flow Name** | Editable field with the scenario name |
| **Save Button** | Saves the current flow (Ctrl+S) |
| **Run Button** | Executes the flow and displays the results |
| **Options Menu** | Export, import, duplicate, and other options |

### 2. Node Palette (Left Side)

The palette displays all available nodes organized by category:

- **Flow Control** — If, Switch, Loop, Merge
- **Web** — Smart Web Flow, Web Flow, Smart Locators
- **API** — HTTP Request
- **Database** — PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, MongoDB
- **Infrastructure** — SSH Command
- **Utilities** — Set Variable, Log, Wait, Stop and Fail, Custom JavaScript
- **Custom Nodes** — Nodes from registered providers (if any)

To add a node, **drag it** from the palette to the canvas.

### 3. Canvas (Central Area)

The canvas is the workspace where you position and connect nodes. Features:

| Action | How To |
|--------|--------|
| **Move the canvas** | Click and drag on the background (or use the mouse scroll) |
| **Zoom** | Mouse scroll or trackpad pinch |
| **Select a node** | Click on the node |
| **Move a node** | Drag the node |
| **Select multiple** | Hold Shift and click, or drag a selection box |
| **Delete a node** | Select and press Delete or Backspace |
| **Connect nodes** | Drag from an output handle (●) to an input handle (●) |
| **Remove connection** | Click on the connection and press Delete |
| **Paste flow** | Ctrl+V with copied flow JSON (e.g., from the Chrome Recorder) |
| **Add annotation** | Right-click the canvas or press Shift+A |
| **Copy/paste node or annotation** | Select the item and use Ctrl+C / Ctrl+V |

### 4. Properties Panel (Right Side)

When you select a node, the properties panel appears on the right with:

- **Run**: Executes the node in isolated mode
- **Variables**: Opens the properties of previous nodes, local variables, and global variables
- **Configuration**: Fields specific to the node type
- **Outputs**: The node's output schema (after execution, shows the actual data)
- **Continue on Failure**: Toggle to continue the flow even if the node fails

---

## Typical Workflow

1. **Create the scenario** from a project
2. **Drag nodes** from the palette to the canvas
3. **Connect nodes** by dragging between handles
4. **Configure each node** using the properties panel
5. **Save** the flow (Ctrl+S)
6. **Run** and check the results

[Typical Workflow](../../assets/images/fluxo-trabalho-editor-fluxos.mp4)

---

## Execution Order

QANode executes nodes in **topological order**, determined by the connections:

```
[A] → [B] → [D]
         ↘
[C] ──────→ [E]
```

In this example:
- `A` executes first
- `B` and `C` can execute next (after `A`)
- `D` executes after `B`
- `E` executes after `B` and `C`

> **Note:** If a node has multiple inputs, it only executes when all previous nodes have completed.

---

## Visual Indicators

After a run, nodes display status indicators:

| Indicator | Meaning |
|-----------|---------|
| 🟢 Green border | Node executed successfully |
| 🔴 Red border | Node failed |
| ⚪ Gray border | Node was skipped (branch not activated) |
| 🔵 Blue border | Node is running |

[Visual Indicators](../../assets/images/indicadores-visuais-editor-fluxos.mp4)

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+S` | Save flow |
| `Delete` / `Backspace` | Delete selected node or connection |
| `Ctrl+C` | Copy node |
| `Ctrl+V` | Paste node, annotation, or JSON from Chrome Recorder |
| `Ctrl+Z` | Undo |
| `Ctrl+Shift+Z` | Redo |
| `Shift+A` | Create a canvas annotation |

### Canvas Annotations

**Annotations** help document parts of the flow directly on the canvas. Use them to explain preconditions, business rules, logical groups, pending work, or notes for the team.

Annotations support:

- free text;
- simple Markdown-style formatting;
- colors;
- resizing;
- dragging and positioning on the canvas;
- copy, paste, duplicate, and delete.

You can create an annotation from the canvas context menu or with `Shift+A`.

[Canvas Annotations](../../assets/images/anotacoes-editor-fluxos.mp4)

---

## Unsaved Changes

The editor automatically detects when there are unsaved changes. When you try to leave the page (via the back button, navigation, or closing the tab), a warning will be displayed:

> **"Leave without saving?"**
> You have unsaved changes. If you leave, your changes will be lost.

This protects against accidental loss of work. The warning only appears when there are real changes — not when opening a flow without modifying it.

---

## Next Steps

- [Working with Nodes](working-with-nodes.md) — Details on adding, connecting, and configuring nodes
- [Execution and Debugging](execution-debugging.md) — How to run flows and diagnose failures
