# Switch Node

The **Switch** node allows you to create multiple branches based on the value of an expression. It is useful when there are more than two possible paths (where If would not be sufficient).

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `switch` |
| **Category** | Flow Control |
| **Color** | Yellow (#f59e0b) |
| **Input** | `in` |
| **Outputs** | `case-0`, `case-1`, ..., `default` (dynamic) |

---

## Configuration

### Simple Mode

In simple mode, you define an expression and values for each case:

**Field: Expression**
```javascript
{{ steps["http-request"].outputs.status }}
```

**Cases:**

| Case | Value | Label (Optional) |
|------|-------|------------------|
| Case 0 | `200` | Success |
| Case 1 | `404` | Not Found |
| Case 2 | `500` | Server Error |
| Default | — | Others |

The expression is evaluated and compared with each case value. If no case matches, the `default` path is activated.

### Visual Builder Mode

The visual mode allows you to use different operators for each case:

| Field | Description |
|-------|-------------|
| **Comparison Field** | Expression to evaluate |
| **Operator** | Comparison operation per case |
| **Value** | Expected value |

**Available operators:**

| Operator | Description |
|----------|-------------|
| `===` | Equal (strict) |
| `!==` | Not equal |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal |
| `<=` | Less than or equal |
| `includes` | Contains |
| `startsWith` | Starts with |
| `endsWith` | Ends with |

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `_branch` | `string` | The name of the activated case |

---

## Behavior

1. The expression is evaluated
2. The result is compared with each case in order
3. The **first case** that matches will have its path activated
4. If no case matches, the `default` path is activated
5. Nodes on the inactive paths will be **skipped**

---

## Practical Example

### Routing by HTTP status

```
[HTTP Request]
    │
    ▼
[Switch: {{ steps["http-request"].outputs.status }}]
    │ case-0 (200) → [Log: "Sucesso"] → [Extrair dados]
    │ case-1 (401) → [Log: "Não autorizado"] → [Refazer login]
    │ case-2 (404) → [Log: "Não encontrado"]
    │ default → [Log: "Erro inesperado"] → [Stop and Fail]
```

### Routing by user type

```
[PostgreSQL: SELECT role FROM users WHERE id = '123']
    │
    ▼
[Switch: {{ steps.query.outputs.rows[0].role }}]
    │ case-0 ("admin") → [Admin Flow]
    │ case-1 ("user") → [User Flow]
    │ default → [Visitor Flow]
```

---

## Tips

- Use **descriptive labels** on cases to make the flow easier to read
- The `default` case should always be connected to handle unexpected values
- For only two paths, prefer the [If](if.md) node
- Cases are evaluated in order — place the most common ones first
