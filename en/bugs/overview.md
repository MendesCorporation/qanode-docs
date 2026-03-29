# Bug Tracking Module — Overview

> **Available in:** QANode Enterprise

The Bug Tracking module turns a failed execution into a controlled tracking process. Instead of a failed run simply sitting in history, you can open a formal bug, assign it to a person or team, follow its investigation, and close it with a complete audit trail.

---

## What the module offers

- **Bug opening** from a failed execution or manually
- **Configurable workflow** — your organization defines the statuses and transitions of the process
- **Queues and assignment** — bugs can be routed to specific teams or individuals
- **Claim** — a user takes ownership of the bug before acting on it
- **Sandbox investigation** — a disposable copy of the original flow to investigate without affecting official runs or KPIs
- **Custom fields** — additional fields defined by your organization
- **Comments, attachments, and full history** — every interaction with the bug is recorded

---

## How to access

The Bug Tracking module appears in the sidebar with the **bug** icon (🐛). Clicking it shows the list of all bugs in the instance with filters by status, severity, priority, queue, and assignee.

To open a bug directly from a failed execution, go to the execution detail and click **Open Bug**.

---

## Permissions

Access to each feature of the module is controlled by permissions assigned to the user's role. The available permissions are:

| Permission | What it allows |
|-----------|----------------|
| `bug.view` | View bugs, comment, and manage their own attachments |
| `bug.create` | Open new bugs |
| `bug.edit` | Edit custom fields of the bug |
| `bug.assign` | Assign bugs to people/queues and claim |
| `bug.transition` | Transition through the workflow's normal transitions |
| `bug.transition_any` | Administrative bypass — transition to any status without ownership restriction |
| `bug.run_sandbox` | Create and discard investigation sandboxes |
| `bug.delete_attachment_any` | Delete attachments uploaded by anyone |
| `bug.configure_workflow` | Configure the workflow, statuses, transitions, and custom fields |

Permissions are configured in **Settings → Roles**.

---

## Core concepts

### Workflow

The workflow defines which statuses exist, which transitions are possible between them, and which fields appear at each stage. Every instance has exactly one active workflow. See [Workflow Builder](./workflow-builder.md).

### Status

Each bug is always in one status. The initial status is defined in the workflow and is where the bug starts. Final statuses indicate closure — when a bug reaches a final status, the closing date is automatically recorded.

### Transition

A transition is the path between two statuses. Each transition can define required fields that must be filled when transitioning.

### Queue

A queue represents a team or group responsible for a set of bugs. When a bug is **only in a queue** (with no direct assignee), any member of that queue with the appropriate permission can act on it. If there is a direct assignee, that user's ownership prevails in the normal flow.

### Claim

Claim is the act of taking ownership of the bug. After claiming, the bug belongs to you. Normal transitioning requires the bug to be claimed by the user who will transition it.

### Sandbox

The sandbox is a disposable copy of the original flow used to investigate the failure. Changes in the sandbox do not affect the official flow and do not count in KPIs and reports. See [Investigation Sandbox](./sandbox.md).

---

## Navigation

| Page | What you will learn |
|------|---------------------|
| [Workflow Builder](./workflow-builder.md) | How to configure statuses, transitions, fields, and custom fields |
| [Bug Lifecycle](./lifecycle.md) | How to open, assign, claim, and transition bugs |
| [Investigation Sandbox](./sandbox.md) | How to investigate failures without affecting official runs |
| [Comments, Attachments & History](./comments-attachments.md) | How to collaborate and track the history of a bug |
