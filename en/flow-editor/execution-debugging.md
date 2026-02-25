# Execution and Debugging

Learn how to run your flows, interpret results, and diagnose failures.

---

## Running a Flow

### Full Execution

To execute the entire flow:

1. Make sure the flow is saved
2. Click the **Run** button (▶️) in the top bar
3. Wait for completion — nodes will display real-time status indicators

[Flow running](../../assets/images/executando.mp4)
*Image: Canvas showing nodes with status indicators during execution*

### Execution Status

| Status | Indicator | Description |
|--------|-----------|-------------|
| **Running** | 🔵 Blue/Pulsing | The node is being processed |
| **Success** | 🟢 Green | The node executed successfully |
| **Failure** | 🔴 Red | The node encountered an error |
| **Skipped** | ⚪ Gray | The node was not executed (inactive branch) |

---

## Analyzing Results

After execution, click on any node to see its results in the properties panel:

### Results Tab

| Section | Description |
|---------|-------------|
| **Status** | Success or failure, with duration |
| **Logs** | Messages recorded during node execution |
| **Outputs** | Data produced by the node (navigable JSON) |
| **Error** | Detailed error message (when applicable) |
| **Screenshots** | Screen captures (web nodes with evidence enabled) |

### Outputs

The outputs of each node are accessible to subsequent nodes via expressions. For example, after running an **HTTP Request**, the outputs will be:

```json
{
  "status": 200,
  "body": "...",
  "json": {
    "id": 1,
    "name": "João",
    "email": "joao@exemplo.com"
  }
}
```

You can then access this data in subsequent nodes:
```
{{ steps["http-request"].outputs.json.name }}  →  "João"
{{ steps["http-request"].outputs.status }}     →  200
```

### Screenshots (Evidence)

For web nodes (Web Flow and Smart Locators), captured screenshots appear as clickable thumbnails. Click to view at full size.

---

## Debugging Failures

### Identifying the Problem Node

1. Look for nodes with a **red** border (🔴) on the canvas
2. Click on the failing node
3. Check the **error message** and the **logs**

### Common Error Messages

#### Web Flow / Smart Locators

| Error | Cause | Solution |
|-------|-------|----------|
| `Timeout waiting for selector` | Element not found on the page | Check the selector/locator; increase the timeout |
| `Element not visible` | Element exists but is not visible | Add a `wait` or `scroll` step before it |
| `Navigation timeout` | Page did not load in time | Check the URL and connectivity |
| `Element is not attached to DOM` | Element removed before the action | Add a `wait` for stability |

#### HTTP Request

| Error | Cause | Solution |
|-------|-------|----------|
| `ECONNREFUSED` | Server is not accessible | Check the URL and whether the server is running |
| `401 Unauthorized` | Invalid authentication | Check token/credentials |
| `ETIMEOUT` | Request exceeded the time limit | Increase the timeout or check the server |

#### Database

| Error | Cause | Solution |
|-------|-------|----------|
| `Connection refused` | Database not accessible | Check host, port, and firewall |
| `Authentication failed` | Invalid credentials | Check username and password |
| `Relation does not exist` | Table not found | Check the table name and database |

#### SSH

| Error | Cause | Solution |
|-------|-------|----------|
| `Authentication failed` | Invalid SSH credentials | Check username, password, or private key |
| `Connection timeout` | Host not accessible | Check host, port, and network |

### Using Logs for Diagnosis

The **logs** of each node provide details about each executed step. For web nodes, logs show:

```
Navigated to: https://exemplo.com/login
Filled: "usuario@exemplo.com" on [getByLabel("E-mail")]
Clicked: [getByRole("button", { name: "Entrar" })]
Assert passed: textContains "Bem-vindo" — true
```

If a step failed, the log will show exactly which step and why:

```
Navigated to: https://exemplo.com/login
Filled: "usuario@exemplo.com" on [getByLabel("E-mail")]
ERROR: Click failed after 3 attempts: [getByRole("button", { name: "Login" })] — element not found
```

---

## Debugging Tips

### 1. Disable Headless Mode

For web tests, disable **headless mode** on the node to see the browser in action:
- On the Web Flow or Smart Locators node, uncheck **Headless**
- Run the flow and observe the browser

### 2. Add Wait Steps

If elements appear with a delay, add **wait** steps before interacting:
- `wait` with `visible` mode waits for the element to appear
- `wait` with `networkIdle` mode waits for all network requests to finish

### 3. Capture Screenshots for Diagnosis

Enable screenshots in **before** mode to see the page state before each action. This helps identify whether the page was in the expected state.

### 4. Check Expressions

If a node fails with an unexpected value, add a **Log** node before it to inspect the values:

```
Token value: {{ steps.login.outputs.json.token }}
Status: {{ steps["http-request"].outputs.status }}
```

### 5. Use the "Continue on Failure" Toggle

To diagnose multiple failures at once, enable **Continue on Failure** on verification nodes. This allows the flow to continue and you can see all failures in a single run.

---

## Execution Report

After each run, QANode automatically generates a **PDF report** with:

- Execution summary (status, duration, date)
- Details of each step (status, logs, outputs, duration)
- Captured screenshots
- Errors encountered

The report is available in the project's execution list and can be downloaded or sent by email.

---

## Next Steps

- [Node Reference](../nodes/flow-control/if.md) — Complete details of each node type
- [Expressions](../expressions/expressions.md) — Master the dynamic data system
