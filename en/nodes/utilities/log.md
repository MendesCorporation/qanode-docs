# Log Node

The **Log** node records a message in the execution log. Useful for debugging, tracking progress, and documenting values during execution.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `log` |
| **Category** | Utilities |
| **Color** | ⚪ Gray (#6b7280) |
| **Input** | `in` |
| **Output** | `out` |

---

## Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Message** | `string` | Text to be recorded (supports `{{ }}`) |

---

## Examples

### Simple log

```
Message: Test started successfully
```

### Log with dynamic data

```
Message: User found: {{ steps.query.outputs.rows[0].name }} ({{ steps.query.outputs.rows[0].email }})
```

### Log for debugging

```
Message: API Status: {{ steps["http-request"].outputs.status }}, Body: {{ JSON.stringify(steps["http-request"].outputs.json) }}
```

---

## Tips

- Use logs strategically to **document values** at critical points in the flow
- Messages appear in the **execution results** and in the **PDF report**
- Combine with **If/Switch** for conditional logs:
  ```
  [If: status === 200]
      │ true → [Log: "Sucesso"]
      │ false → [Log: "Falha: status {{ steps.api.outputs.status }}"]
  ```
