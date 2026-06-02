# Variables

**Variables** are reusable values available in test scenarios. They allow you to centralize configurations such as URLs, test credentials, flags, and shared data.

QANode has two main scopes:

- **Global** — available to any scenario.
- **Project** — available to scenarios linked to a specific project.

---

## Managing Variables

1. In the sidebar, click **Variables**
2. The list displays registered global variables

[Variable list](../../assets/images/variavel.mp4)
*Image: Variables screen showing a list with name, type, value, and last updated*

---

## Global and Project Variables

### Global Variables

Global variables should be used for values shared across multiple projects, such as:

- common base URL;
- execution flags;
- reusable test data;
- general settings.

They are created from the sidebar **Variables** menu.

### Project Variables

Project variables live inside the **Variables** tab on the project page. Use this scope when the value belongs to a specific project, such as:

- that system's staging URL;
- a test user for that project;
- identifiers or test data owned by that team;
- settings that should not appear in the global list.

When a scenario belongs to a project, project variables are available with the same syntax as global variables:

```
{{ variables.VARIABLE_NAME }}
```

If a global variable and a project variable have the same name, avoid relying on that overlap. Prefer clear names for each scope.

---

## Creating a Variable

1. For a global variable, open **Variables** from the sidebar. For a project variable, open the project and go to the **Variables** tab
2. Click **+ New Variable**
3. Fill in:

| Field | Required | Description |
|-------|----------|-------------|
| **Name** | Yes | Variable identifier (no spaces) |
| **Type** | Yes | Value type |
| **Value** | Yes | Variable value |
| **Secret** | No | If enabled, the value will be encrypted |

4. Click **Save**

---

## Variable Types

| Type | Description | Example |
|------|-------------|---------|
| **String** | Text | `https://api.exemplo.com` |
| **Number** | Number | `30000` |
| **Boolean** | True/False | `true` |
| **JSON** | Object or array | `{ "timeout": 5000, "retries": 3 }` |

---

## Secret Variables

Variables marked as **secret** are encrypted in the database. They are ideal for:

- Test passwords
- API tokens
- Access keys

Characteristics:
- The value is **masked** in the interface (displayed as `••••••`)
- The value is **encrypted** in the database
- The actual value is **accessible** during flow executions

> **Note:** For connection credentials (database, SSH, API), prefer using [Credentials](../credentials/credentials.md) instead of secret variables.

---

## Using Variables in Flows

Access variables using the expression syntax:

```
{{ variables.VARIABLE_NAME }}
```

In the flow editor, the **Variables** button in the properties panel opens an organized list where you can drag or click values.

Variables are separated by scope:

| Group | Initial state | Description |
|-------|---------------|-------------|
| **Flow** | Open | Local variables and outputs from previous nodes in the current flow |
| **Project** | Closed | Variables registered in the scenario's project |
| **Global** | Closed | Global variables for the installation |

This separation avoids mixing global, project, and flow data when many variables are registered.

### Previous Node Outputs

Each executed node appears as a group with:

- node name;
- node type;
- execution status;
- number of available fields.

Objects and arrays start collapsed to keep the list readable. When a field is expanded, its children appear below with a short visual name. For example, inside `fileRef`, children appear as `name`, `mimeType`, and `sizeBytes`, but the copied expression remains complete.

### Files in the Variables Panel

When an output is a file (`fileRef`), the panel shows the file as a draggable item. When expanded, the main details are available:

| Field | Use |
|-------|-----|
| `name` | File name |
| `mimeType` | MIME type |
| `sizeBytes` | Size in bytes |

Internal fields such as `source` and `path` do not appear in the main list to avoid confusing common usage, but they remain accessible by expression when needed.

If an input field does not accept files, or does not accept that file type, the panel marks the field in red and shows an unsupported file message.

### Usage Examples

**API URL:**
```
URL: {{ variables.API_URL }}/api/users
```

**Timeout:**
```
Timeout: {{ variables.DEFAULT_TIMEOUT }}
```

**Test credentials:**
```
Email: {{ variables.TEST_EMAIL }}
Password: {{ variables.TEST_PASSWORD }}
```

**JSON configuration:**
```
Retries: {{ variables.CONFIG.retries }}
Timeout: {{ variables.CONFIG.timeout }}
```

---

## Runtime Override

The **Set Variable** node can override variables during execution:

```
[Set Variable: API_URL = https://staging.exemplo.com]
    │
    ▼
[HTTP Request: GET {{ variables.API_URL }}/api/health]
```

The override is temporary — it only affects the current execution. The original value in the database is not changed.

### Pipeline Override — Enterprise

In QANode Enterprise CI/CD integration, a variable can also be overridden directly by the CLI or API for a specific execution.

Example:

```bash
npx @qanode/cli run suite \
  --suite-id SUITE_ID \
  --var BASE_URL=https://preview.app \
  --wait
```

This override:

- applies only to the current execution
- does not change the persisted variable value
- is useful for temporary build, staging, and preview environments

> For details, see [Per-execution Overrides](../ci-cd/overrides.md).

---

## Best Practices

### Naming

Use **UPPER_SNAKE_CASE** for global and project variables:

| Good | Bad |
|------|-----|
| `API_URL` | `apiUrl` |
| `TEST_EMAIL` | `email` |
| `DEFAULT_TIMEOUT` | `timeout` |
| `MAX_RETRIES` | `retries` |

### Environment Organization

Create variables with an environment prefix to make switching easier:

```
PROD_API_URL = https://api.exemplo.com
STAGING_API_URL = https://staging.exemplo.com
DEV_API_URL = http://localhost:3000
```

### When to Use Variables vs. Credentials

| Scenario | Use |
|----------|-----|
| API base URL | Variable |
| Authentication token | HTTP Credential |
| Database connection data | Database Credential |
| Generic test password | Secret Variable |
| Configuration flag | Variable |
| SSH connection data | SSH Credential |

---

## Tips

- **Centralize URLs** in variables to make environment switching easier
- **Use secret variables** for sensitive data
- **Prefer credentials** for structured connection data
- Use **global** variables for values shared across the whole instance
- Use **project** variables for project-specific data
- The **JSON** type is useful for grouping related configurations
