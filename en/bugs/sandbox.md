# Investigation Sandbox

> **Permission required:** `bug.run_sandbox`

The sandbox is an isolated investigation environment linked to a bug. It allows you to run the original flow that caused the failure, observe the behavior, and make adjustments — without affecting official data, contaminating reports, or interfering with real project runs.

---

## What the sandbox is

When a bug is opened from a failed execution, QANode preserves a **snapshot of the original flow** at the moment the bug was created. The sandbox always starts from this snapshot, not from the current flow.

This means that:

- Even if someone edits or deletes the original flow afterward, the sandbox remains based on the exact state that caused the problem
- The investigation always starts from the scenario that reproduces the failure, not from a newer version that may have changed

---

## Opening the sandbox

1. Go to the bug detail
2. Click **Open Sandbox**
3. QANode creates a copy of the original flow marked as sandbox
4. You are redirected to the editor with this flow

If an active sandbox already exists for the bug, the system reopens the existing one instead of creating a new one.

---

## What you can do in the sandbox

Inside the sandbox you have access to the full flow editor and direct shortcuts to the bug:

- **Run** the flow to reproduce the failure
- **Inspect** nodes and see step-by-step results
- **Modify** the flow to test fix hypotheses
- **Re-run** as many times as needed
- **Add a comment to the bug** without leaving the sandbox (💬 button in the toolbar)
- **Add an attachment to the bug** without leaving the sandbox (📎 button in the toolbar)

### Direct comment on the bug

Click the **comment** button (balloon icon) in the toolbar. A modal opens for you to type and save the comment directly on the bug linked to the sandbox — no need to go back to the bug screen.

Use it to record findings while still investigating: what you reproduced, what you tested, what you found.

### Direct attachment on the bug

Click the **paperclip** button in the toolbar. A modal opens with a drop zone — paste a screenshot with Ctrl+V or click to select a file. The file is sent directly to the linked bug. Useful for attaching screenshots, logs, or evidence collected during the investigation, without leaving the sandbox.

All runs made inside the sandbox:
- Are marked as `sandbox` internally
- **Do not appear in official reports and dashboards**
- **Do not count in project KPIs**
- **Do not affect the original flow**

---

## Who can open the sandbox

Sandbox access follows the same ownership rules as the bug:

| Bug situation | Who can open sandbox |
|---------------|----------------------|
| In queue, no assignee | Any member of that queue with `bug.run_sandbox` |
| Assigned directly to a user | Only that user in the normal flow |
| In another team's queue | Users outside the queue cannot in the normal flow |
| In final status | Nobody — sandbox is blocked at final status |
| User with `bug.transition_any` | Can bypass ownership, but still needs `bug.run_sandbox` |

---

## Discarding the sandbox

When the investigation is done:

1. Go to the bug detail
2. Click **Discard Sandbox**
3. Confirm the action

Discarding:
- Removes the sandbox flow
- Removes all sandbox runs linked to it
- Records the event in the bug history

Discarding is permanent — a removed sandbox cannot be recovered. If you need to investigate again, a new sandbox can be created at any time (while the bug is not in a final status).

---

## Difference between sandbox and official runs

| | Official run | Sandbox run |
|-|--------------|-------------|
| Appears in reports | ✅ Yes | ❌ No |
| Counts in project KPIs | ✅ Yes | ❌ No |
| Based on current flow | ✅ Yes | ❌ Uses original snapshot |
| Can be freely modified | ❌ No | ✅ Yes |

---

## Best practices

- **Don't discard the sandbox before documenting** — if you found the root cause, add a comment on the bug with your conclusions before discarding
- **Use the sandbox to reproduce before fixing** — confirming you can reproduce the failure in the sandbox is the first step of a good investigation
- **One sandbox at a time** — the system reuses the existing sandbox. If you need a clean state, discard the current one and open a new one
