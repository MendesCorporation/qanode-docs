# Administration

The administration section allows managing users, roles, permissions, general settings, SMTP, alarms, webhooks, audit logging, and licensing.

Access it through the **gear** icon (⚙️) in the side menu → **Settings**.

---

## Users

### Inviting Users

1. Go to **Settings** → **Users**
2. Click **Invite User**
3. Fill in:
   - **Email**: New user's email address
   - **Role**: Assigned role (defines permissions)
4. An invitation email will be sent with an activation link

> **Requirement:** SMTP must be configured for sending invitations.

### Bulk Import

To register multiple users at once:

1. Go to **Settings** → **Users**
2. Click the **import** icon (next to the invite button)
3. Download the **Excel template** by clicking **Download Template**
4. Fill in the spreadsheet with user data:

| Column | Description |
|--------|-------------|
| **name** | User's full name |
| **email** | User's email address |
| **role** | Role name (e.g.: Admin, Architect, Tester) |

5. Upload the Excel file (.xlsx or .xls) by dragging it to the designated area or clicking to select
6. The system will process each row and send invitation emails

After processing, a summary is displayed showing:
- Number of users successfully imported
- List of failures with the row number, email, and reason for the error (e.g.: email already exists, role not found, missing required fields)

> **Note:** The import respects the license's user limit. If the file contains more users than the plan allows, the system will notify you and offer the option to import only up to the available limit.

### Managing Users

In the user list you can:

| Action | Description |
|--------|-------------|
| **Activate/Deactivate** | Enables or disables user access |
| **Change Role** | Changes the user's role (and permissions) |
| **Delete** | Permanently removes the user |

### User Status

| Status | Description |
|--------|-------------|
| **Active** | User with normal access |
| **Inactive** | Access blocked |
| **Pending** | Invitation sent, awaiting activation |

---

## Roles and Permissions

### System Roles

QANode includes predefined roles that cannot be removed:

| Role | Description |
|------|-------------|
| **Super Admin** | Full access to all features |
| **Admin** | Administrative access |
| **Architect** | Project administration access |
| **Tester** | Scenario and execution access |

### Custom Roles

Create roles with specific permissions:

1. Go to **Settings** → **Roles and Permissions**
2. Click **+ New Role**
3. Define the name, table view, and select the permissions

### Table View

When creating or editing a role, the **Table View** field defines which database tables are accessible in the **query builder** of custom dashboards. This controls what the user can query when creating widgets with SQL queries or exploring data in dashboards.

| Profile | Accessible Tables |
|---------|-------------------|
| **admin** | All tables: Projects, Flows, Suites, Runs, Run Steps, Variables, Credentials, Users, Roles, Bugs, Audit Log |
| **architect** | Projects, Flows, Suites, Runs, Run Steps, Variables |
| **tester** | Projects, Flows, Suites, Runs, Run Steps |
| **none** | No tables — query builder disabled |

> **Note:** Sensitive columns (passwords, tokens, encrypted data) are automatically hidden, regardless of the view profile.

#### Default profiles for system roles

| System Role | Table View |
|-------------|------------|
| **Super Admin** | admin |
| **Admin** | admin |
| **Architect** | architect |
| **Tester** | tester |

### Complete Permissions List

#### Scenarios (Flows)
| Permission | Description |
|------------|-------------|
| `flow.view` | View scenarios — allows seeing the list and details of scenarios |
| `flow.create` | Create scenarios |
| `flow.edit.own` | Edit own scenarios (created by the user) |
| `flow.edit.all` | Edit any scenario (including those of other users) |
| `flow.delete.own` | Delete own scenarios |
| `flow.delete.all` | Delete any scenario |
| `flow.run` | Execute scenarios |

#### Projects
| Permission | Description |
|------------|-------------|
| `project.view` | View projects — allows seeing the list and details of projects |
| `project.create` | Create projects |
| `project.edit.own` | Edit own projects (created by the user) |
| `project.edit.all` | Edit any project |
| `project.archive` | Archive projects |
| `project.delete` | Permanently delete projects |

#### Suites
| Permission | Description |
|------------|-------------|
| `suite.view` | View suites — allows seeing the list and details of suites |
| `suite.create` | Create suites |
| `suite.edit` | Edit suites |
| `suite.delete` | Delete suites |
| `suite.run` | Execute suites |

#### Executions (Runs)
| Permission | Description |
|------------|-------------|
| `run.view` | View own executions — allows seeing the execution history of the user |
| `run.view.all` | View all executions — allows seeing executions from all users |
| `run.cancel` | Cancel ongoing executions |
| `run.delete` | Delete execution records |

#### Variables
| Permission | Description |
|------------|-------------|
| `variable.view` | View variables — allows seeing the variable list |
| `variable.create` | Create variables |
| `variable.edit` | Edit variables |
| `variable.delete` | Delete variables |
| `variable.view.secret` | View secret values — allows revealing the value of variables marked as secret |

#### Credentials
| Permission | Description |
|------------|-------------|
| `credential.view` | View credentials — allows seeing the list of registered credentials |
| `credential.create` | Create credentials |
| `credential.edit` | Edit credentials |
| `credential.delete` | Delete credentials |

#### Components
| Permission | Description |
|------------|-------------|
| `component.view` | View reusable components |
| `component.create` | Create and edit own components |
| `component.edit` | Edit components created by other users |
| `component.delete.own` | Delete own components |
| `component.delete` | Delete any component |
| `component.publish` | Publish components for use in scenarios |

#### Providers
| Permission | Description |
|------------|-------------|
| `provider.view` | View providers |
| `provider.create` | Register providers |
| `provider.edit` | Edit providers |
| `provider.delete` | Remove providers |

#### Reports
| Permission | Description |
|------------|-------------|
| `report.view` | View reports — allows accessing PDF reports generated in executions |
| `report.export` | Export reports — allows downloading reports as PDF |

#### Dashboards
| Permission | Description |
|------------|-------------|
| `dashboard.view` | View dashboards — allows accessing and viewing existing dashboards |
| `dashboard.create` | Create custom dashboards |
| `dashboard.edit` | Edit dashboards |
| `dashboard.delete` | Delete dashboards |
| `dashboard.share` | Share dashboards — allows making dashboards public or sharing with specific roles |
| `dashboard.sql` | Execute SQL — allows using direct SQL queries in the widget query builder |

#### Webhooks
| Permission | Description |
|------------|-------------|
| `webhook.view` | View webhooks |
| `webhook.create` | Create webhooks |
| `webhook.edit` | Edit webhooks |
| `webhook.delete` | Delete webhooks |

#### Users
| Permission | Description |
|------------|-------------|
| `user.view` | View user list |
| `user.invite` | Invite new users — sends invitation email with activation link |
| `user.edit` | Edit user data (change role, information) |
| `user.delete` | Permanently delete users |
| `user.deactivate` | Deactivate/activate users — blocks or restores access without deleting |

#### Roles
| Permission | Description |
|------------|-------------|
| `role.view` | View roles |
| `role.create` | Create roles |
| `role.edit` | Edit roles |
| `role.delete` | Delete roles |

#### Bugs — Enterprise
| Permission | Description |
|------------|-------------|
| `bug.view` | View bugs — allows seeing the list, commenting, and managing attachments |
| `bug.create` | Open new bugs |
| `bug.edit` | Edit custom fields of the bug |
| `bug.assign` | Assign bugs to people or queues and claim |
| `bug.transition` | Transition bugs through the workflow's normal transitions |
| `bug.transition_any` | Administrative bypass — transition to any status without ownership restriction |
| `bug.run_sandbox` | Create and discard investigation sandboxes |
| `bug.delete_attachment_any` | Delete attachments uploaded by anyone |
| `bug.configure_workflow` | Configure the workflow, statuses, transitions, and custom fields |

#### Settings
| Permission | Description |
|------------|-------------|
| `settings.view` | View settings — basic access to the settings page |
| `settings.smtp` | Configure SMTP — allows changing email settings |
| `settings.integration_token` | Manage your own integration tokens |
| `settings.integration_token_all` | View all tokens, revoke tokens created by other people, and define the global expiration policy |
| `settings.mfa` | Configure MFA — allows managing two-factor authentication |
| `settings.audit` | View audit — access to the audit log |
| `settings.report_template` | Manage report templates — allows creating and editing PDF templates |

---

## General Settings — Super Admin

The **General** tab groups settings that affect the entire instance. It is available to Super Admins in **Settings → General**.

Sections are organized in accordions to make navigation easier.

### Security and MFA

The global MFA configuration is in **Settings → General → Security and MFA**.

Use this section to require two-factor authentication and strengthen access for administrative users.

### Version History — Enterprise

Version history allows QANode to keep snapshots of scenarios and components.

| Setting | Description |
|---------|-------------|
| **Scenario history** | Maximum number of versions retained per scenario |
| **Component history** | Maximum number of versions retained per component |

How it works:

- scenarios create a snapshot when a relevant flow change is saved;
- components create a snapshot when a new version is published;
- older versions can be opened in read-only mode;
- restoring creates a new current version from the selected snapshot;
- when the limit is reduced, exceeding versions are removed automatically.

The recommended default is to keep enough versions for auditing and rollback without accumulating unnecessary history.

### Execution Data Retention

Execution data retention removes old execution artifacts, such as evidence and technical attachments from older runs. It does not remove the execution record itself.

Main fields:

| Field | Description |
|-------|-------------|
| **Enable scheduled cleanup** | Runs cleanup automatically once per day |
| **Successful executions** | Days to keep artifacts from successful executions |
| **Failed executions** | Days to keep artifacts from failed executions |
| **Cancelled executions** | Days to keep artifacts from cancelled executions |
| **Sandbox executions** | Days to keep artifacts from temporary executions |
| **Keep latest per flow** | Preserves the most recent evidence per scenario |

Scheduled cleanup runs once per day after the fixed time defined by the instance. You can also use **Preview cleanup** to estimate the impact before running it.

For safety, cleanup preserves executions linked to bugs or tickets and does not remove bug attachments, ticket evidence, sessions, report templates, or saved browser sessions.

---

## SMTP Configuration

For sending emails (invitations, reports, alarms):

1. Go to **Settings** → **SMTP**
2. Fill in:

| Field | Description |
|-------|-------------|
| **Host** | SMTP server (e.g.: `smtp.gmail.com`) |
| **Port** | Server port |
| **Security** | STARTTLS (587), TLS/SSL (465) or None |
| **User** | Authentication email/username |
| **Password** | Email password |
| **Sender** | Sending address (from) |

3. Click **Save** — the system tests the connection automatically

---

## Integration Tokens

**Integration Tokens** allow pipelines and automations to authenticate without relying on a browser session.

To access them:

1. Go to **Settings** → **Access Tokens**
2. Generate a token
3. Store the value as a pipeline secret
4. Use it with `QANODE_URL` and `QANODE_TOKEN`

Users with `settings.integration_token_all` can also:

- view tokens from the whole instance
- revoke tokens created by other people
- define the global expiration policy

> For the complete guide, see [CI/CD Integration — Overview](../ci-cd/overview.md).

---

## Alarms

Configure automatic notifications for suite failures:

1. Go to **Settings** → **Alarms**
2. Configure:
   - **Monitored suites** — which suites trigger an alarm
   - **Recipients** — who receives the notification
   - **Condition** — when to notify (failure, always, etc.)

> **Requirement:** SMTP must be configured for sending alarms by email.

---

## Webhooks

Webhooks allow automatically sending HTTP POST notifications to external services when execution events occur. Each user manages their own webhooks independently.

### Configuring Webhooks

1. Go to **Settings** → **Webhooks**
2. Click **+ New Webhook**
3. Fill in:

| Field | Description |
|-------|-------------|
| **Name** | Webhook identifier name |
| **URL** | HTTP/HTTPS address that will receive the POST |
| **Secret** | (Optional) HMAC key for payload signing |
| **Events** | Which events trigger the webhook |
| **Scope** | Filter by specific project, suite, or scenario |

### Available Events

#### Test execution

| Event | Description |
|-------|-------------|
| `run.completed` | Scenario execution completed (success or failure) |
| `run.success` | Scenario execution completed successfully |
| `run.failed` | Scenario execution completed with failure |
| `suite.completed` | Suite execution completed (success or failure) |
| `suite.success` | Suite execution completed successfully |
| `suite.failed` | Suite execution completed with failure |

#### Defects

| Event | Description |
|-------|-------------|
| `defect.opened` | A new defect was created |
| `defect.status_changed` | A defect transitioned to a non-final workflow status |
| `defect.closed` | A defect transitioned to a final workflow status (fixed, rejected, closed, etc.) |

### Scope (Filters)

You can filter webhooks to trigger only in specific contexts:

- **Project** — Only events from a specific project
- **Suite** — Only executions from a specific suite (test execution events only)
- **Scenario** — Only events from a specific scenario/flow

> When selecting a project, suites and scenarios are filtered to show only those belonging to that project. When selecting a suite, the scenario filter is hidden.

> **Note:** Suite scope does not apply to defect events. A webhook scoped to a suite will only fire for test execution events (`run.*`, `suite.*`), never for defect events.

### Webhook Payload

Every delivery is an HTTP `POST` with the following JSON body:

```json
{
  "event": "<event-name>",
  "timestamp": "2026-04-15T10:30:00.000Z",
  "data": { ... }
}
```

#### Test execution payload

```json
{
  "event": "run.completed",
  "timestamp": "2026-04-15T10:30:00.000Z",
  "data": {
    "runId": "uuid",
    "flowId": "uuid",
    "flowName": "Login Flow",
    "suiteId": "uuid",
    "suiteName": "Smoke Suite",
    "projectId": "uuid",
    "projectName": "App Mobile",
    "status": "success",
    "durationMs": 1234,
    "executor": "Username",
    "errorSummary": null
  }
}
```

#### Defect payload (`defect.opened`, `defect.status_changed`, `defect.closed`)

```json
{
  "event": "defect.status_changed",
  "timestamp": "2026-04-15T15:10:00.000Z",
  "data": {
    "defectId": "uuid",
    "defectNumber": 42,
    "title": "Login button does not respond on Safari",
    "description": "When clicking Login on Safari 17...",
    "status": "in_progress",
    "previousStatus": "open",
    "severity": "high",
    "priority": "urgent",
    "resolution": null,
    "isFinal": false,
    "closedAt": null,
    "createdAt": "2026-04-15T14:32:00.000Z",
    "updatedAt": "2026-04-15T15:10:00.000Z",
    "project": { "id": "uuid", "name": "App Mobile" },
    "flow": { "id": "uuid", "name": "Login Flow" },
    "run": { "id": "uuid", "status": "failed" },
    "reporterUser": { "id": "uuid", "name": "Maria", "email": "maria@company.com" },
    "assigneeUser": { "id": "uuid", "name": "João Silva", "email": "joao@company.com" },
    "assigneeRole": null,
    "customFields": { "environment": "production", "browser": "Safari 17" },
    "actionComment": {
      "content": "Starting investigation on staging",
      "user": { "id": "uuid", "name": "João Silva", "email": "joao@company.com" }
    }
  }
}
```

| Field | Description |
|-------|-------------|
| `previousStatus` | Status before this transition; `null` on `defect.opened` |
| `isFinal` | `true` when the current status is a final/closed status |
| `closedAt` | ISO timestamp when the defect was closed; `null` if still open |
| `customFields` | Key-value map of custom fields defined in the defect workflow |
| `actionComment` | Note entered during this action, with the user who performed it; `null` if no note was provided |

### HMAC Signature

If the **Secret** field is filled in, QANode adds the `X-QANode-Signature` header with the HMAC-SHA256 signature of the payload. This allows the receiving service to validate the authenticity of the request.

Example header:
```
X-QANode-Signature: sha256=abc123...
```

### Retries

In case of delivery failure (network error or HTTP code other than 2xx), QANode automatically retries up to **3 times** with increasing intervals:

| Attempt | Interval |
|---------|----------|
| 1st | Immediate |
| 2nd | 1 second |
| 3rd | 5 seconds |

### Delivery Log

For each webhook, you can view the history of the last 50 deliveries:

1. Click the **list** icon next to the webhook
2. View the status of each delivery (OK/Failed), HTTP code, attempts, and errors

### Testing Webhooks

Click the **play** icon next to the webhook to send a test payload. This allows you to verify that the URL is accessible and responding correctly.

### Permissions

The following roles have access to webhooks:

| Role | Access |
|------|--------|
| **Super Admin** | Full |
| **Admin** | Full |
| **Architect** | Full |
| **Tester** | No access |

> Each user views and manages only their own webhooks.

---

## Audit

The audit log records all important actions in the system:

### Tracked Actions

| Action | Description |
|--------|-------------|
| **create** | Resource creation |
| **update** | Resource update |
| **delete** | Resource deletion |
| **execute** | Scenario/suite execution |
| **login** | User login |
| **logout** | User logout |
| **import** | Data import |
| **export** | Data export |
| **test** | Connection test |
| **invite** | User invitation |
| **setup** | Initial configuration |

### Tracked Entities

Projects, Flows, Suites, Runs, Variables, Credentials, Providers, Users, Roles, Settings, System.

### Log Information

| Field | Description |
|-------|-------------|
| **Action** | Type of action performed |
| **Entity** | Type and name of the affected resource |
| **User** | Who performed the action |
| **IP** | Source IP address |
| **Date/Time** | When the action occurred |

### Viewing

1. Go to **Settings** → **Audit**
2. Navigate through the history using pagination
3. Use filters to search for specific actions


---

## Password Change

Any user can change their own password:

1. Go to **Settings** → **Change Password**
2. Enter the current password
3. Enter and confirm the new password
4. Click **Save**

---

## MFA (Two-Factor Authentication)

For greater security, enable two-factor authentication:

1. Go to **Settings** → **MFA**
2. Scan the QR Code with an authenticator app (Google Authenticator, Authy, etc.)
3. Enter the generated code to confirm
4. MFA will be active — codes will be requested at each login

---

## License

Manage the QANode license:

1. Go to **Settings** → **License**
2. View:
   - License **Status** (valid/expired)
   - **Expiration date**
   - **Remaining days**
   - **User limit**
   - **Active users** vs limit

---

## Tips

- **Configure SMTP first** — many features depend on email
- Use **custom roles** for fine-grained access control
- **Monitor the audit log** regularly to detect unexpected actions
- **Enable MFA** for administrator accounts
- **Deactivate users** instead of deleting — preserves audit history
- **Use webhooks** to integrate QANode with Slack, Teams, Discord, or any service that accepts HTTP POST
- **Set the secret** on webhooks to ensure request authenticity
