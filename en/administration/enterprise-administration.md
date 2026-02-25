# Administration

The administration section allows managing users, roles, permissions, SMTP settings, alarms, webhooks, audit logging, and licensing.

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
| **admin** | All tables: Projects, Flows, Suites, Runs, Run Steps, Variables, Credentials, Users, Roles, Audit Log |
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

#### Settings
| Permission | Description |
|------------|-------------|
| `settings.view` | View settings — basic access to the settings page |
| `settings.smtp` | Configure SMTP — allows changing email settings |
| `settings.mfa` | Configure MFA — allows managing two-factor authentication |
| `settings.audit` | View audit — access to the audit log |
| `settings.report_template` | Manage report templates — allows creating and editing PDF templates |

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

| Event | Description |
|-------|-------------|
| `run.completed` | Scenario execution completed (success or failure) |
| `run.success` | Scenario execution completed successfully |
| `run.failed` | Scenario execution completed with failure |
| `suite.completed` | Suite execution completed (success or failure) |
| `suite.success` | Suite execution completed successfully |
| `suite.failed` | Suite execution completed with failure |

### Scope (Filters)

You can filter webhooks to trigger only in specific contexts:

- **Project** — Only executions from a specific project
- **Suite** — Only executions from a specific suite
- **Scenario** — Only executions from a specific scenario

> When selecting a project, suites and scenarios are filtered to show only those belonging to that project. When selecting a suite, the scenario filter is hidden.

### Webhook Payload

The POST body sent contains:

```json
{
  "event": "run.completed",
  "timestamp": "2026-02-13T10:30:00.000Z",
  "data": {
    "runId": "uuid",
    "flowId": "uuid",
    "flowName": "Nome do Cenário",
    "suiteId": "uuid",
    "suiteName": "Nome da Suíte",
    "projectId": "uuid",
    "projectName": "Nome do Projeto",
    "status": "success",
    "durationMs": 1234,
    "executor": "Nome do Usuário",
    "errorSummary": null
  }
}
```

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
