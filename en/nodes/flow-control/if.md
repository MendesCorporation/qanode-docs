# If Node

The **If** node allows you to create conditional branches in the flow. It evaluates an expression and directs execution to either the **true** or **false** path.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `if` |
| **Category** | Flow Control |
| **Color** | Yellow (#f59e0b) |
| **Input** | `in` |
| **Outputs** | `true`, `false` |

---

## Configuration

### Simple Mode

In simple mode, you write a JavaScript expression that will be evaluated as true or false:

**Field: Condition**
```javascript
{{ steps["http-request"].outputs.status === 200 }}
```

The expression has access to the execution context:
- `steps` — Outputs of previous nodes
- `variables` — Global variables

**Examples:**

```javascript
// Check HTTP status
{{ steps["http-request"].outputs.status === 200}}

// Check if text contains a value
{{ steps.extract.outputs.extracts.title.includes("Bem-vindo") }}

// Check numeric value
{{ steps.query.outputs.rowCount > 0 }}

// Check boolean
{{ variables.featureEnabled === true }}

// Combination with AND/OR
{{ steps.api.outputs.status  === 200 && steps.api.outputs.json.active  === true }}
```

### Visual Builder Mode

The visual mode allows you to build conditions without writing code:

| Field | Description | Example |
|-------|-------------|---------|
| **Field** | Expression to evaluate | `steps["http-request"].outputs.status` |
| **Operator** | Comparison operation | `===` |
| **Value** | Expected value | `200` |
| **Logic** | Operator between conditions | `AND` / `OR` |

**Available operators:**

| Operator | Description |
|----------|-------------|
| `===` | Equal (strict) |
| `!==` | Not equal (strict) |
| `==` | Equal (with conversion) |
| `!=` | Not equal (with conversion) |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal |
| `<=` | Less than or equal |
| `includes` | Contains (for strings) |
| `startsWith` | Starts with |
| `endsWith` | Ends with |

You can add multiple conditions combined with **AND** (all must be true) or **OR** (at least one must be true).

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `_branch` | `string` | The path taken: `"true"` or `"false"` |

---

## Behavior

1. The expression is evaluated using the data from the current context
2. If the result is **truthy** (true, number > 0, non-empty string), the `true` output is activated
3. If the result is **falsy** (false, 0, null, undefined, empty string), the `false` output is activated
4. Nodes connected to the **inactive** path will be **skipped**

---

## Practical Example

### Check API response

```
[HTTP Request: GET /api/user/1]
    │
    ▼
[If: status === 200]
    │ true → [Log: "Usuário encontrado: {{ steps["http-request"].outputs.json.name }}"]
    │ false → [Log: "Usuário não encontrado"] → [Stop and Fail]
```

### Data validation

```
[PostgreSQL: SELECT count(*) FROM orders WHERE user_id = '123']
    │
    ▼
[If: rowCount > 0]
    │ true → [Log: "Pedidos encontrados"]
    │ false → [Log: "Nenhum pedido"]
```

---

## Tips

- Use the **visual builder mode** whenever possible — it is more readable and less error-prone
- Name the node descriptively: "If login successful", "If record exists"
- Remember that `{{ }}` expressions are evaluated before the comparison
- For complex conditions, consider using a **Custom JavaScript** node beforehand and evaluating the result in the If node
