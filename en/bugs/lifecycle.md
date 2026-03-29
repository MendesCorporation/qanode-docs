# Bug Lifecycle

This page describes the complete journey of a bug — from opening to closure — including how assignment, queues, claim, and transitioning work in practice.

---

## 1. Opening a bug

### From a failed execution

This is the most common way to open a bug. When an execution ends with failure:

1. Go to the execution detail
2. Click **Open Bug**
3. Fill in the fields required by the workflow (defined in the initial status)
4. Click **Save**

The bug is created with a direct link to the execution, the flow, and, when applicable, the step that caused the failure. This link is preserved even if the flow is edited later — the bug stores a snapshot of the original scenario state.

> **Permission required:** `bug.create`

### Manually

To open a bug without a link to an execution:

1. Go to **Bugs** in the sidebar
2. Click **New Bug**
3. Fill in the required fields
4. Click **Save**

---

## 2. Bug identification

Each bug automatically receives:

- **Bug Key** — unique reference code in the format `BUG-N` (e.g.: `BUG-42`), useful for mentioning in comments, emails, or external systems
- **Creation date** and **reporter** — who opened it and when

---

## 3. Assignment — Queues and Assignees

After being opened, a bug needs to be directed to someone to work on it. This can happen in two ways:

### Queue

A queue represents a team. When a bug is in a queue:
- Any member of that queue with `bug.assign` can **claim** and take ownership of the bug
- The bug is visible to the entire queue team

Queues can be applied automatically when the workflow defines a **default queue** for a status — when transitioning to that status, the bug is automatically routed to the configured queue.
Queues can be applied automatically when the workflow defines a **default queue** for a status — in the absence of a manual routing choice, the bug is routed to the configured queue.

### Direct assignee

When a bug is assigned directly to a user:
- Only that user can transition in the normal flow
- Other users, including users from the same queue, cannot claim it in the normal flow while there is a direct assignee

### No assignment

A bug with no queue and no assignee can be claimed by any user with `bug.assign`.

---

## 4. Claim — Taking ownership

**Claim** is the act of putting the bug in your name to work on it. Normal transitioning requires the bug to be claimed by the user who will transition it.

To claim:

1. Open the bug detail
2. Click **Claim**

### When can I claim?

| Bug situation | Who can claim |
|---------------|---------------|
| In queue, no assignee | Any member of that queue with `bug.assign` |
| No queue and no assignee | Any user with `bug.assign` |
| With direct assignee | Nobody in the normal flow; only someone with `bug.transition_any` can act through administrative bypass |
| In final status | Nobody — final status blocks claiming |

### What claiming does

- Sets your user as the direct assignee of the bug
- Keeps the associated queue (the bug remains linked to the team)

### What claiming does not do

Claiming does not remove the associated queue. However, if there is a direct assignee, that ownership becomes the effective one in the normal flow. In practice, other members of the same queue can no longer claim or transition the bug while it is assigned to someone.

---

## 5. Transitioning — Changing the status

Transitioning means moving the bug from one status to another following the transitions defined in the workflow.

### How to transition

1. Open the bug detail
2. The available transition buttons appear based on the transitions from the current status
3. Click the desired transition (e.g.: "Start Investigation", "Fix", "Close")
4. Fill in the required transition fields (if any)
5. Add an optional note
6. Confirm

### Transitioning rules

**To transition in the normal flow (`bug.transition`):**
- The bug must be claimed by your user
- If the bug is only in a queue (no direct assignee), you must claim it first
- If the bug belongs to another assignee, you cannot transition it

**To transition with administrative bypass (`bug.transition_any`):**
- You can transition to any status, regardless of who the assignee is
- No prior claim needed
- Useful for process managers and administrators

### What happens after transitioning

- The bug status is updated
- If the transition does not manually define a new assignee or queue, and the target status has a **default queue**, the bug is routed to it
- If the target status is **final**, the closing date is recorded
- If the bug returns from a final status to an active status, the closing date is removed
- The transition generates an event in the bug's **history** with the note provided

---

## 6. Practical situations

### Bug stuck in QA queue, no assignee

1. A QA user with `bug.assign` claims → bug is now in their name
2. They investigate and click "Fix" → bug moves to the next status
3. If the next status has a Dev default queue, the bug is automatically routed to the Dev queue

### Bug with a direct assignee

1. Only that assignee can transition in the normal flow
2. Another user with `bug.transition` cannot claim or transition while there is already a direct assignee
3. A manager with `bug.transition_any` can advance the bug without restriction

### Closed bug being reopened

1. A user with `bug.transition_any` transitions back to an active status (e.g.: "Reopened")
2. The closing date is automatically removed
3. The history records the reopening with who did it and when

---

## 7. Bug list

The bug list screen shows all bugs in the instance with filters by:

- **Status** — filter by one or more workflow statuses
- **Severity** — critical, high, medium, low
- **Priority** — bug urgency
- **Queue** — responsible team
- **Assignee** — assigned user

Each row in the list shows the Bug Key, title, status, severity, priority, assignee, and creation date. Click any bug to open its detail.

---

## 8. Bug detail

The bug detail screen brings everything together in one place:

| Section | What it shows |
|---------|---------------|
| **Header** | Bug Key, title, current status, reporter, and date |
| **Information** | Severity, priority, assignee, queue, project, and origin execution |
| **Custom fields** | Additional fields defined by the organization |
| **Transitions** | Buttons to transition to the next status |
| **Sandbox** | Button to open investigation in an isolated environment |
| **Comments** | Discussion about the bug |
| **Attachments** | Files sent to the bug |
| **History** | Complete trail of all changes and transitions |
