# Variables

**Variables** are global reusable values available in any test scenario. They allow you to centralize configurations such as URLs, test credentials, flags, and shared data.

---

## Managing Variables

1. In the sidebar, click **Variables**
2. The list displays all registered variables

[Variable list](../../assets/images/variavel.mp4)
*Image: Variables screen showing a list with name, type, value, and last updated*

---

## Creating a Variable

1. Click **+ New Variable**
2. Fill in:

| Field | Required | Description |
|-------|----------|-------------|
| **Name** | Yes | Variable identifier (no spaces) |
| **Type** | Yes | Value type |
| **Value** | Yes | Variable value |
| **Secret** | No | If enabled, the value will be encrypted |

3. Click **Save**

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

---

## Best Practices

### Naming

Use **UPPER_SNAKE_CASE** for global variables:

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
- Variables are **global** — accessible in any flow
- The **JSON** type is useful for grouping related configurations
