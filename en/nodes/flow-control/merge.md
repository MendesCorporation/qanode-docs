# Merge Node

The **Merge** node allows you to join multiple execution paths into a single point. It is useful for reconnecting paths that were split by an If or Switch node.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `merge` |
| **Category** | Flow Control |
| **Color** | Yellow (#f59e0b) |
| **Inputs** | `in1`, `in2` |
| **Output** | `out` |

---

## Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Mode** | `string` | Merge strategy |

---

## Behavior

The Merge node waits for at least one of the input paths to be activated before proceeding. When a conditional path (If/Switch) activates only one branch, the Merge recognizes that the other branch was skipped and does not wait indefinitely.

---

## Practical Example

### Reconnecting after If

```
                   ┌── true ──→ [Process A] ──┐
[If: condition] ───┤                           ├──→ [Merge] → [Log: "Continuando"]
                   └── false ─→ [Process B] ──┘
```

In this example:
1. The If node evaluates the condition
2. Only one of the paths (A or B) will be executed
3. The Merge receives the result from the activated path
4. The flow continues normally after the Merge

### Reconnecting after Switch

```
[Switch] ──case-0──→ [Action 1] ──┐
         ──case-1──→ [Action 2] ──┼──→ [Merge] → [Next]
         ──default─→ [Action 3] ──┘
```

---

## Tips

- Use Merge whenever you need to reconnect split paths
- Merge is especially useful for avoiding duplication of nodes that should execute regardless of the path taken
- Name the Merge descriptively: "Merge after validation", "Join paths"
