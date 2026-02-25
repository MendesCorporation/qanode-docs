# Loop Node

The **Loop** node allows you to repeat the execution of a section of the flow multiple times. It supports three repetition modes: by count, by array, and by condition (while).

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `loop` |
| **Category** | Flow Control |
| **Color** | Yellow (#f59e0b) |
| **Input** | `in` |
| **Outputs** | `loop` (loop body), `done` (output after completion) |

---

## Loop Modes

### 1. Count

Repeats a fixed number of times.

| Field | Type | Description |
|-------|------|-------------|
| **Count** | `string/number` | Number of iterations |
| **Start Index** | `number` | Initial index value (default: 0) |

**Example:** Repeat 5 times
```
Count: 5
Start Index: 0
```

At each iteration, the index is incremented (0, 1, 2, 3, 4).

### 2. Array

Iterates over each item in an array.

| Field | Type | Description |
|-------|------|-------------|
| **Array Expression** | `string` | Expression that returns an array |

**Example:** Iterate over query results
```
{{ steps.query.outputs.rows }}
```

At each iteration, the current item and index are available in the loop outputs.

### 3. While

Repeats while a condition is true.

| Field | Type | Description |
|-------|------|-------------|
| **Condition** | `string` | Expression evaluated at each iteration |
| **Maximum Iterations** | `number` | Safety limit (default: 100) |

Like the If node, while mode supports **simple mode** (JavaScript expression) and **visual builder mode**.

**Example:** Repeat while there is a next page
```javascript
{{ steps["http-request"].outputs.json.hasNextPage }} === true
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `_loopItem` | `any` | Item iterated in array mode |
| `_loopIndex` | `any` | Index of the last iteration |
| `_loopCount` | `number` | Total number of iterations |

---

## Behavior

1. The loop evaluates whether it should continue (based on the mode)
2. If yes, it executes all nodes connected to the **loop** output
3. At the end of the loop body, it returns to the Loop node for the next iteration
4. When the loop ends, it executes the nodes connected to the **done** output

### Visual Flow

```
        ┌─────────────────────────────────┐
        │                                 │
        ▼                                 │
[Loop] ──loop──→ [Action 1] → [Action 2] ─┘
   │
   done
   │
   ▼
[Next Node]
```

> **Note:** Nodes in the loop body are executed at each iteration. Use expressions to access the current item/index.

---

## Practical Example

### Iterate over database results

```
[PostgreSQL: SELECT * FROM users WHERE active = true]
    │
    ▼
[Loop (array): {{ steps.query.outputs.rows }}]
    │ loop → [HTTP Request: POST /api/notify]
    │             Body: { "email": "{{ steps.loop.outputs._loopItems[steps.loop.outputs._loopCount].email }}" }
    │
    │ done → [Log: "Todas as notificações enviadas"]
```

### Loop with count

```
[Loop (count): 3 times]
    │ loop → [HTTP Request: GET /api/retry-endpoint]
    │             → [If: status === 200]
    │                   true → (break/success)
    │
    │ done → [Log: "Tentativas esgotadas"]
```

---

## Tips

- Always configure the **maximum iterations** in while mode to avoid infinite loops
- For large arrays, consider the performance impact
- Use **Log** nodes inside the loop to track progress
- In array mode, the array items are accessible via `_loopItems`
