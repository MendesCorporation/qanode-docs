# Set Variable Node

The **Set Variable** node allows you to define or update variables at runtime. Variables defined this way are available to all subsequent nodes in the flow.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `set-variable` |
| **Category** | Utilities |
| **Color** | ⚪ Gray (#6b7280) |
| **Input** | `in` |
| **Output** | `out` |

---

## Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Key** | `string` | Variable name |
| **Value** | `any` | Value to be assigned (supports `{{ }}`) |

---

## Usage

### Define a simple variable

```
Key: token
Value: {{ steps.login.outputs.json.token }}
```

### Define a variable with a calculation

```
Key: totalPrice
Value: {{ steps.cart.outputs.json.subtotal * 1.1 }}
```

### Access the variable in subsequent nodes

After defining it, the variable is accessible via:
```
{{ variables.token }}
{{ variables.totalPrice }}
```

---

## Examples

### Save an authentication token

```
[HTTP Request: POST /login] → [Set Variable: token = {{ steps.login.outputs.json.token }}]
    │
    ▼
[HTTP Request: GET /profile → Header: Authorization = Bearer {{ variables.token }}]
```

### Prepare a dynamic URL

```
[Set Variable: baseUrl = https://{{ variables.ENVIRONMENT }}.exemplo.com]
    │
    ▼
[HTTP Request: GET {{ variables.baseUrl }}/api/users]
```

---

## Tips

- Use **descriptive names** for variables: `authToken` is better than `t`
- Variables defined with Set Variable **overwrite** global variables with the same name during execution
- The value supports **any expression** `{{ }}`, including calculations and access to outputs
