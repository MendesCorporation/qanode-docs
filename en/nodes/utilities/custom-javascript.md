# Custom JavaScript Node

The **Custom JavaScript** node allows you to execute custom JavaScript code during the flow execution. Ideal for data transformations, complex calculations, and logic not covered by native nodes.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `custom-js` |
| **Category** | Utilities |
| **Color** | ⚪ Gray (#6b7280) |
| **Input** | `in` |
| **Output** | `out` |

---

## Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Code** | `string` | JavaScript code to be executed |

The code editor uses the **Monaco Editor** (same as VS Code) with:
- Syntax highlighting
- Autocomplete for `steps`, `variables`, `return`, `console.log`
- Syntax validation

---

## Execution Context

The code has access to:

| Variable | Type | Description |
|----------|------|-------------|
| `steps` | `object` | Outputs from all previous nodes |
| `variables` | `object` | Global and runtime variables |

### Accessing data from previous nodes

```javascript
// Output from an HTTP Request
const userData = steps["http-request"].outputs.json;
const userName = userData.name;

// Output from a PostgreSQL query
const rows = steps["postgres-query"].outputs.rows;
const firstUser = rows[0];

// Output from web extraction
const title = steps["web-flow"].outputs.extracts.pageTitle;
```

### Accessing variables

```javascript
const apiUrl = variables.API_URL;
const environment = variables.ENVIRONMENT;
```

---

## Returning Values

The code **must return** a value using `return`. The returned value is available in the outputs:

```javascript
// Return a simple value
return "Hello World";

// Return an object
return {
  fullName: steps["Query"].outputs.rows[0].first_name + " " + steps["Query"].outputs.rows[0].last_name,
  isAdmin: steps["Query"].outputs.rows[0].role === "admin"
};

// Return an array
return steps["Query"].outputs.rows.map(row => row.email);
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `result` | `any` | Value returned by the code |

### Accessing the result

```
{{ steps["custom-js"].outputs.result }}
{{ steps["custom-js"].outputs.result.fullName }}
{{ steps["custom-js"].outputs.result[0] }}
```

---

## Practical Examples

### Transform API data

```javascript
const items = steps["http-request"].outputs.json.items;

// Filter and transform
const activeItems = items
  .filter(item => item.active)
  .map(item => ({
    id: item.id,
    name: item.name.toUpperCase(),
    price: (item.price * 1.1).toFixed(2)
  }));

return {
  items: activeItems,
  count: activeItems.length,
  totalPrice: activeItems.reduce((sum, item) => sum + parseFloat(item.price), 0)
};
```

### Generate test data

```javascript
const timestamp = Date.now();
return {
  email: `test-${timestamp}@exemplo.com`,
  username: `user_${timestamp}`,
  password: `Pass${timestamp}!`
};
```

### Complex validation

```javascript
const response = steps["API Check"].outputs.json;
const dbRecord = steps["DB Query"].outputs.rows[0];

const validations = [];

if (response.name !== dbRecord.name) {
  validations.push(`Nome diverge: API="${response.name}" DB="${dbRecord.name}"`);
}

if (response.email !== dbRecord.email) {
  validations.push(`Email diverge: API="${response.email}" DB="${dbRecord.email}"`);
}

return {
  valid: validations.length === 0,
  errors: validations,
  errorCount: validations.length
};
```

### Format a report

```javascript
const runs = steps["Query Runs"].outputs.rows;

const summary = {
  total: runs.length,
  passed: runs.filter(r => r.status === 'success').length,
  failed: runs.filter(r => r.status === 'failed').length,
  passRate: 0
};

summary.passRate = ((summary.passed / summary.total) * 100).toFixed(1) + '%';

return summary;
```

---

## Limitations

- The code runs in a **sandbox environment** — with no access to:
  - File system (fs)
  - Node.js modules (require/import)
  - Network (fetch, http)
  - Environment variables (use `variables` instead of `process.env`)
- For network operations, use the **HTTP Request** node
- For database operations, use the **Database** nodes
- For file operations, use the **SSH Command** node

---

## Tips

- Use descriptive names for the node: "Transform Data", "Generate Credentials", "Validate Response"
- **Always return** a value — without `return`, the output will be `undefined`
- For debugging, use `console.log()` — messages appear in the node logs
- The Monaco Editor offers autocomplete — type `steps.` to see available nodes
- Use it to **transform data** between nodes, not for complex business logic
