# Working with Nodes

This guide details how to add, connect, configure, and manage nodes in the flow editor.

---

## Adding Nodes

### Dragging from the Palette

The primary way to add nodes is by dragging from the **node palette** (left side) to the **canvas**:

1. Locate the desired category in the palette
2. Click and drag the node onto the canvas
3. Release it at the desired position

[Dragging a node from the palette](../../assets/images/flow-editor.mp4)
*Image: Node being dragged from the node palette to the canvas*

### Pasting JSON (Chrome Recorder)

You can also paste nodes copied from the **Chrome Recorder** (extension):

1. In the Chrome Recorder, click **Copy JSON**
2. In the flow editor, press **Ctrl+V**
3. The Web Flow node will be added with all recorded steps

---

## Connecting Nodes

### Creating Connections

To connect two nodes:

1. Hover over the **output handle** (●) of the source node
2. Click and drag to the **input handle** (●) of the target node
3. Release to create the connection

<!-- ![Connecting nodes](../../assets/images/conectar-nos.png) -->
*Image: Line being dragged from an output handle to an input handle*

### Input and Output Handles

| Type | Position | Description |
|------|----------|-------------|
| **Input** (in) | Top of the node | Receives data from previous nodes |
| **Output** (out) | Bottom of the node | Sends data to subsequent nodes |

Some nodes have **multiple output handles**:

- **If** → `true` and `false`
- **Switch** → One handle per case + `default`
- **Loop** → `loop` (loop body) and `done` (loop exit)

### Removing Connections

To remove a connection:
1. Click on the connection line to select it
2. Press **Delete** or **Backspace**

---

## Configuring Nodes

### Properties Panel

When you click on a node, the properties panel opens on the right. Each node type has specific fields, but all share:

| Field | Description |
|-------|-------------|
| **Label** | Node name (displayed on the canvas) |
| **Continue on Failure** | If enabled, the flow continues even if this node fails |

### Using Expressions in Fields

Most fields accept **expressions** with the `{{ }}` syntax:

```
{{ steps["Node Name"].outputs.property }}
{{ variables.myVariable }}
```

This allows nodes to use data produced by previous nodes. For example:

- **Navigation URL**: `{{ variables.BASE_URL }}/login`
- **Text to fill**: `{{ steps["Node Name"].outputs.result.email }}`
- **SQL**: `SELECT * FROM users WHERE email = '{{ steps.extract.outputs.extracts.email }}'`

> For more details, see [Expressions](../expressions/expressions.md).

---

## Nodes with Multiple Steps

Some nodes support **multiple internal steps**, making them more powerful:

### Web Flow and Smart Locators

These nodes allow you to add several web automation steps within a single node:

1. In the properties panel, click **+ Add Step**
2. Select the action type (navigate, click, fill, assert, etc.)
3. Configure the step parameters
4. Repeat to add more steps

Steps are executed **sequentially**, in the order they appear in the list.

You can:
- **Reorder** steps by dragging
- **Expand/Collapse** steps to see details
- **Remove** steps by clicking the trash icon
- **Configure evidence** (screenshots) individually per step

### SSH Command

The SSH node also supports multiple steps (commands):

1. Each step is a command to be executed
2. Steps are executed sequentially over the same SSH connection
3. Optionally, wait for specific text in the output (match string)

---

## Flow Control Nodes

### Creating Conditional Branches

To create a conditional branch:

1. Add an **If** node to the canvas
2. Configure the condition in the properties panel
3. Connect the **true** output to the path that should be executed when the condition is true
4. Connect the **false** output to the alternative path

**Example:**

```
[HTTP Request] → [If status === 200]
                       │ true → [Log "Success"]
                       │ false → [Log "Error"] → [Stop and Fail]
```

### Simple Condition vs. Visual Builder

The **If** node offers two configuration modes:

**Simple Mode:**
```javascript
{{ steps["http-request"].outputs.status === 200 }}
```

**Visual Builder Mode:**
- Field: `steps["http-request"].outputs.status`
- Operator: `===`
- Value: `200`

The visual builder mode is more user-friendly and does not require knowledge of JavaScript.

---

## Nodes with Visual Query Builder

Database nodes (PostgreSQL, MySQL, etc.) offer a **visual query builder** in addition to the direct SQL option:

### Available Presets

| Preset | Description |
|--------|-------------|
| **Custom SQL** | Write SQL freely |
| **SELECT** | Visual SELECT builder |
| **EXISTS** | Checks if a record exists |
| **COUNT** | Counts records |
| **ASSERT** | Verifies if a value matches |
| **INSERT** | Visual INSERT builder |
| **UPDATE** | Visual UPDATE builder |
| **DELETE** | Visual DELETE builder |

The query builder allows you to select tables, columns, WHERE conditions, ORDER BY, and LIMIT without writing SQL.

---

## Tips and Best Practices

### Name Your Nodes

Give descriptive names to your nodes by changing the **label**. This makes the flow easier to read and makes expressions clearer:

```
{{ steps.login.outputs.json.token }}
```
is more readable than:
```
{{ steps["HTTP Request 2"].outputs.json.token }}
```

### Use Logical Groups

Organize related nodes close to each other on the canvas. For example, group login nodes in one area and verification nodes in another.

### Configure Evidence

For web tests, enable screenshots on critical steps. This makes debugging easier and generates evidence in reports.

### Use Continue on Failure with Care

The **Continue on Failure** toggle is useful for verification (assert) nodes where you want to record all failures instead of stopping at the first one. Use it sparingly — in general, failures should interrupt the flow.

---

## Next Steps

- [Execution and Debugging](execution-debugging.md) — How to run and diagnose issues
- [Node Reference](../nodes/flow-control/if.md) — Details of each node type
