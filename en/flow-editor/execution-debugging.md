# Execution and Debugging

Learn how to run your flows, interpret results, and diagnose failures.

---

## Running a Flow 

### Full Execution

To execute the entire flow:

1. Make sure the flow is saved
2. Click the **Run** button (▶️) in the top bar
3. Wait for completion — nodes will display real-time status indicators

[Flow running](../../assets/images/execucao-completa-fluxos.mp4)

*Image: Canvas showing nodes with status indicators during execution*

### Execution Status

| Status | Indicator | Description |
|--------|-----------|-------------|
| **Running** | 🔵 Blue/Pulsing | The node is being processed |
| **Success** | 🟢 Green | The node executed successfully |
| **Failure** | 🔴 Red | The node encountered an error |
| **Cancelled** | ⚪ Gray | The execution was manually interrupted |
| **Skipped** | ⚪ Gray | The node was not executed (inactive branch) |

---

## Cancelling Executions

Running executions can be cancelled by users with the `run.cancel` permission.

Where to cancel:

- **Scenario editor**: during execution, the run button changes to a stop button.
- **Execution list**: use the action menu for the running execution.
- **Suite execution**: cancelling the parent execution also cancels pending or running scenarios in that suite.

When cancelled, the final status is **Cancelled**. Evidence and logs already generated up to that point remain available for review.

Cancellation happens at safe execution points. Web, Smart Web, mobile, API, database, SSH, load, and custom JavaScript nodes check for cancellation while processing. For browser or mobile session nodes, QANode also tries to close opened resources to avoid stuck sessions.

> A very long external action may finish its current segment before recognizing cancellation. In those cases, QANode cancels as soon as execution returns to a safe point.

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

[Analyzing Results](../../assets/images/analise-resultados-editor-fluxos.mp4)

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

For web nodes (Smart Web Flow, Web Flow, and Smart Locators), captured screenshots appear as clickable thumbnails. Click to view at full size.

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
- On the Smart Web Flow, Web Flow, or Smart Locators node, uncheck **Headless**
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

## Opening a Bug from a Failed Execution — Enterprise

When an execution ends with **failure**, the **Open Bug** button becomes available in the execution detail. Clicking it opens a modal to fill in the bug information — title, description, severity, priority, and other fields defined by the workflow.

The created bug is automatically linked to the execution, the scenario, and the step that caused the failure.

> **Permission required:** `bug.create`

[Opening a Bug](../../assets/images/abrir-defeito-editor-fluxos.mp4)

### Related bugs

The execution detail also displays the **Related Bugs** section with the list of all bugs linked to that execution — bug key, title, status, and severity — with a direct link to each bug.

---

## Execution Report

After each run, QANode automatically generates a **PDF report** with:

- Execution summary (status, duration, date)
- Details of each step (status, logs, outputs, duration)
- Captured screenshots
- Errors encountered

The report is available in the project's execution list and can be downloaded or sent by email.

[Execution Report](../../assets/images/relatorio-editor-fluxos.mp4)

---

## Next Steps

- [Node Reference](../nodes/flow-control/if.md) — Complete details of each node type
- [Expressions](../expressions/expressions.md) — Master the dynamic data system
