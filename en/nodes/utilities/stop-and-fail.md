# Stop and Fail Node

The **Stop and Fail** node immediately halts the flow execution and marks the result as **failed**. It is a terminal node — it has no outputs.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `stop-and-fail` |
| **Category** | Utilities |
| **Color** | ⚪ Gray (#6b7280) |
| **Input** | `in` |
| **Outputs** | None (terminal node) |

---

## Configuration

This node has no configuration fields. When reached, it simply stops execution.

---

## Usage

Stop and Fail is typically used at the end of conditional paths that represent error scenarios:

```
[If: {{ steps.api.outputs.status }} === 200]
    │ true → [Continuar testes...]
    │ false → [Log: "API retornou erro"] → [Stop and Fail]
```

```
[Switch: {{ steps.query.outputs.rowCount }}]
    │ case-0 (0) → [Log: "Nenhum dado encontrado"] → [Stop and Fail]
    │ default → [Processar dados...]
```

---

## Tips

- Combine with a **Log** node beforehand to record the reason for the failure
- Use after critical validations that must prevent the flow from continuing
- The entire execution status will be marked as **failed**
