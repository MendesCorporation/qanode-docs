# Workflow Builder

> **Permission required:** `bug.configure_workflow`

The Workflow Builder is the screen where your organization defines how bugs will flow — which statuses exist, which transitions are possible between them, which fields appear at each stage, and which custom fields the module will have.

Access it at **Settings → Bug Workflow**.

---

## Screen overview

The screen is divided into two main areas:

- **Statuses** — list of all workflow statuses with their fields and configurations
- **Transitions** — list of all possible transitions between statuses

All changes must be saved with the **Save Workflow** button to take effect.

---

## Managing Statuses

### Create a status

1. Click **Add Status**
2. Define:
   - **Name** — name displayed in the interface (e.g.: "In Triage", "Under Investigation")
   - **Initial Status** — check if this is the status where every bug starts when opened (only one can be initial)
   - **Final Status** — check if this status represents closure of the bug (more than one is allowed)
   - **Default Queue** — if set, it is used as the destination queue when no manual routing choice is sent
   - **Allow manual assignment** — if enabled, the user can freely choose the direct assignee when transitioning to this status

### Edit a status

Click on the status name or the edit icon next to it. The same creation settings are available for editing.

### Remove a status

Click the delete icon next to the status.

> **Warning:** A status that still has open bugs associated with it cannot be removed. Move or close those bugs before removing the status.

### Reorder statuses

Drag statuses by their side handle to reorganize the order in which they appear in the interface.

---

## Status fields

The initial status defines the fields that appear when opening a bug. Transition forms are defined on transitions.

The available native fields are:

| Field | Description |
|-------|-------------|
| Title | Bug title |
| Description | Detailed description |
| Severity | Impact level (critical, high, medium, low) |
| Priority | Resolution urgency |
| Resolution | Closure outcome (e.g.: "Fixed", "Not Reproducible") |
| Assignee | User assigned to the bug |
| Queue | Team responsible for the bug |

In addition to native fields, all **active custom fields** are also available.

### Visible vs. Required

For each field you can define:

- **Visible** — the field appears in the interface but does not need to be filled in
- **Required** — the field must be filled in for the operation to be completed

> **Note:** A required field always appears in the interface, even if it is not marked as visible.

---

## Managing Transitions

A transition defines the path from one status to another. Without a transition created, it is not possible to move a bug between two statuses in the normal flow.

### Create a transition

1. Click **Add Transition**
2. Define:
   - **Name** — label displayed on the transition button (e.g.: "Start Investigation", "Fix", "Reject")
   - **Source Status** — the bug's current status
   - **Target Status** — the status the bug moves to when transitioning

### Transition fields

Just like statuses, each transition can have its own visible and required fields. These fields appear in the transition modal when the user clicks the transition button.

Use transition fields to capture relevant information at the moment of the change — for example, requiring **Resolution** when closing, or a **Queue** when handing the bug to another team.

### Remove a transition

Click the delete icon next to the transition. Removing a transition does not affect existing bugs — it only prevents new bugs from using that path.

---

## Default workflow

When no active workflow exists in the instance, QANode automatically creates a default workflow with the following statuses:

| Status | Type |
|--------|------|
| Open | Initial |
| In Progress | — |
| Fixed | — |
| Rejected | — |
| Reopened | — |
| Closed | Final |

You can use this workflow as a starting point and adjust it to match your organization's process.

> **Important:** The default workflow is only created when no active workflow exists. If you already have a workflow configured, the default will never overwrite yours.

---

## Custom Fields

Custom fields allow your organization to add specific information to the bug process — such as an external ticket number, affected system, build version, and more.

Access them in the **Workflow Builder** screen itself, in the **Custom Fields** section.

### Create a custom field

1. Click **Add Field**
2. Define:
   - **Name** — internal identifier for the field (no spaces, e.g.: `external_ticket`)
   - **Label** — name displayed in the interface (e.g.: "Jira Ticket")
   - **Type** — value type (text, number, date, selection)
   - **Active** — whether the field is available for use

### Using custom fields in the workflow

After creating a custom field and making it active, it becomes available to be added as visible or required in any status or transition of the workflow.

### Deactivate vs. delete

- **Deactivate** — the field stops appearing in the interface but already saved data is preserved
- **Delete** — the field is permanently removed

> **Warning:** A field that is in use by an active workflow cannot be deactivated or deleted. Remove the field from the workflow before deactivating it.

---

## Best practices

- **Start simple** — a workflow with 4 or 5 statuses and direct transitions is easier to operate than one with many branches
- **Use a default queue when you want to direct the step to a specific team** — this helps suggest the right queue when the transition does not send another queue manually
- **Require resolution only at closure** — fields like "Resolution" make more sense as required in the closing transition
- **Avoid orphaned statuses** — every status (except final ones) should have at least one outgoing transition, otherwise bugs may get stuck
