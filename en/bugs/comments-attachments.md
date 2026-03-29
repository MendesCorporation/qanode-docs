# Comments, Attachments & History

> **Permission required:** `bug.view`

All collaboration and all activity on a bug are recorded in its detail screen. This page explains how to use comments, attachments, and history to track and document the lifecycle of a bug.

---

## Comments

Comments are the discussion space for the bug. Use them to:

- Record findings during investigation
- Communicate blockers to the team
- Document tested hypotheses
- Coordinate handoffs between teams

### How to comment

1. Go to the bug detail
2. Scroll to the **Comments** section
3. Type your message in the text field
4. Click **Comment**

The comment is saved with your name, profile picture, and date/time. In addition to appearing in the comments section, it also generates an event in the bug's **history**.

---

## Attachments

Attachments allow you to include relevant files directly in the bug — screenshots, logs, reproduction videos, error reports, and more.

### How to attach a file

1. Go to the bug detail
2. Scroll to the **Attachments** section
3. Click **Add Attachment** or drag the file to the designated area
4. The file is uploaded and available to everyone with access to the bug

### Deleting attachments

Attachment deletion follows an authorship rule:

| Who can delete | Condition |
|----------------|-----------|
| The user who uploaded | Can delete any file they uploaded |
| Admin / Super Admin with `bug.delete_attachment_any` | Can delete any attachment from anyone |
| Other users | Cannot delete files uploaded by others |

To delete:
1. Find the attachment in the **Attachments** section
2. Click the delete icon next to the file
3. Confirm the action

---

## History

The history is the complete auditable trail of the bug. Each recorded event shows who did it, what they did, and when.

### What generates an event in the history

| Action | Generated event |
|--------|----------------|
| Bug opened | "Bug created by [user]" |
| Status transition | "Status changed from [X] to [Y] by [user]" + transition note |
| Claim | "Claimed by [user]" |
| Assignee change | "Assignee changed to [user]" |
| Queue change | "Queue changed to [queue]" |
| Comment added | "Comment by [user]" |
| Sandbox created | "Investigation sandbox created" |
| Sandbox discarded | "Investigation sandbox discarded" |
| Attachment added | Generates a history event with the file name; the file also appears in the **Attachments** section |
| Attachment removed | Generates a history event with the removed file name; the removal also affects the **Attachments** section |

### How to access the history

The history is in the **History** section in the bug detail, in chronological order. You can see the entire timeline of the bug from creation to the current moment.

---

## Transition note

When transitioning a bug (changing its status), you can add a **transition note**. This note:

- Appears in the history alongside the status change event
- Is optional in transitions
- Helps document the reason for the change — for example, when rejecting a bug, explain why it was not reproducible

---

## Best practices

- **Comment before transitioning** — if the transition will change bug ownership to another team, record the investigation context in a comment before passing it on
- **Use the transition note for decisions** — when closing, fixing, or rejecting, the transition note is the right place to record what was done or found
- **Attach reproduction evidence** — sandbox screenshots and logs are temporary; if they are relevant for the permanent record, download and attach them directly to the bug
- **History is immutable** — comments and history events cannot be edited or deleted, ensuring complete traceability
