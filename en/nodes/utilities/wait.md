# Wait Node

The **Wait** node pauses the flow execution for a specified amount of time or until a condition is met.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `wait` |
| **Category** | Utilities |
| **Color** | ⚪ Gray (#6b7280) |
| **Input** | `in` |
| **Output** | `out` |

---

## Modes

### Duration

Waits for a fixed amount of time in milliseconds.

| Field | Type | Description |
|-------|------|-------------|
| **Duration (ms)** | `number` | Wait time in milliseconds |

**Example:** Wait 5 seconds
```
Mode: duration
Duration: 5000
```

### Until Condition (Until)

Waits until a condition becomes true, checking periodically.

| Field | Type | Description |
|-------|------|-------------|
| **Condition** | `string` | Expression to be evaluated |
| **Interval (ms)** | `number` | Interval between checks (default: 1000) |
| **Timeout (ms)** | `number` | Maximum wait time (default: 30000) |

**Example:** Wait until a variable is defined
```
Mode: until
Condition: {{ variables.processComplete === true }}
Interval: 2000
Timeout: 60000
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `waited` | `number` | Actual wait time (ms) |
| `conditionMet` | `boolean` | Whether the condition was met (until mode) |

---

## Practical Examples

### Wait between requests

```
[HTTP Request: POST /start-job]
    │
    ▼
[Wait: 10000ms]
    │
    ▼
[HTTP Request: GET /job-status]
```

### Polling until completion

```
[HTTP Request: POST /process]
    │
    ▼
[Wait Until: {{ steps.checkStatus.outputs.json.status }} === "done"]
    │ Interval: 5000ms, Timeout: 120000ms
    ▼
[If: conditionMet]
    │ true → [Log: "Processamento concluído"]
    │ false → [Log: "Timeout!"] → [Stop and Fail]
```

---

## Tips

- Use **duration** for simple waits (e.g., waiting for propagation)
- Use **until condition** for polling (checking status periodically)
- Always set a **timeout** in "until" mode to avoid infinite waits
