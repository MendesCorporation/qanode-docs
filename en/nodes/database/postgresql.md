# PostgreSQL Node

The **PostgreSQL** node allows you to execute queries and operations on a PostgreSQL database. It offers both a **visual query builder** and the option to write **custom SQL**.

---

## Overview

[Overview](../../assets/images/nos-postgres-visao-geral.mp4)

| Property | Value |
|----------|-------|
| **Type** | `postgres-query` |
| **Category** | Database |
| **Color** | 🟢 Green (#22c55e) |
| **Input** | `in` |
| **Output** | `out` |

---

## Connection

### Using Saved Credentials (Recommended)

Select a credential of type **PostgreSQL** in the **Credential** field. The credential contains all connection information.

### Manual Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Connection String** | `string` | Full URI (alternative) |
| **Host** | `string` | Server address |
| **Port** | `number` | Port (default: 5432) |
| **Database** | `string` | Database name |
| **User** | `string` | Connection user |
| **Password** | `string` | Connection password |
| **SSL** | `boolean` | Use SSL connection |

**Connection String:**
```
postgresql://user:password@host:5432/database?sslmode=require
```

---

## Query Modes (Presets)

### Custom SQL

Write SQL freely in the **SQL** field. To use dynamic data, place `{{ }}` expressions directly in the query.

| Field | Type | Description |
|-------|------|-------------|
| **SQL** | `string` | SQL query (supports `{{ }}`) |

**Example with variable:**
```sql
SELECT *
FROM users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = true;
```

If `variables.EMAIL_TESTE` is `john@example.com`, the executed SQL is equivalent to:

```sql
SELECT *
FROM users
WHERE email = 'john@example.com'
  AND active = true;
```

**Example with another node output:**

```sql
SELECT *
FROM users
WHERE id = {{ steps.api.outputs.json.id }};
```

For text, dates, and string-like values, wrap the expression in quotes. For numbers and booleans, normally use the expression without quotes.

> **Caution:** because the expression is applied directly to the SQL, use controlled values and review quotes in text fields. Avoid using free-form end-user content without validation.

---

### SELECT (Visual Builder)

Build SELECT queries without writing SQL:

| Field | Type | Description |
|-------|------|-------------|
| **Table** | `string` | Source table |
| **Columns** | `array` | Columns to select (empty = all) |
| **WHERE Conditions** | `array` | Filters |
| **ORDER BY** | `object` | Column and sort direction |
| **LIMIT** | `number` | Record limit |

**WHERE Conditions:**

| Field | Operator | Value |
|-------|----------|-------|
| `email` | `=` | `john@example.com` |
| `age` | `>` | `18` |
| `name` | `LIKE` | `%Smith%` |

---

### EXISTS

Checks whether at least one record matching the criteria exists.

| Field | Type | Description |
|-------|------|-------------|
| **Table** | `string` | Table |
| **WHERE Conditions** | `array` | Search criteria |

Returns `rowCount: 1` if it exists, `rowCount: 0` if it does not exist.

---

### COUNT

Counts records matching the criteria.

| Field | Type | Description |
|-------|------|-------------|
| **Table** | `string` | Table |
| **WHERE Conditions** | `array` | Count criteria |

---

### ASSERT

Checks whether a value in the database matches the expected value. Ideal for data validations.

| Field | Type | Description |
|-------|------|-------------|
| **Table** | `string` | Table |
| **Column** | `string` | Column to check |
| **Operator** | `string` | Comparison operation |
| **Value** | `string` | Expected value |
| **WHERE Conditions** | `array` | Filters to locate the record |

---

### INSERT

Inserts new records.

| Field | Type | Description |
|-------|------|-------------|
| **Table** | `string` | Target table |
| **Values** | `array` | Column-value pairs |

**Example:**

| Column | Value |
|--------|-------|
| `name` | `Maria` |
| `email` | `maria@example.com` |
| `active` | `true` |

---

### UPDATE

Updates existing records.

| Field | Type | Description |
|-------|------|-------------|
| **Table** | `string` | Table |
| **Sets** | `array` | Column-value pairs to update |
| **WHERE Conditions** | `array` | Filters |

---

### DELETE

Removes records.

| Field | Type | Description |
|-------|------|-------------|
| **Table** | `string` | Table |
| **WHERE Conditions** | `array` | Filters (required for safety) |

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `rows` | `array` | Array of returned records |
| `rowCount` | `number` | Number of affected/returned records |

### Accessing Outputs

```
// All records
{{ steps.query.outputs.rows }}

// First record
{{ steps.query.outputs.rows[0] }}

// Specific field from the first record
{{ steps.query.outputs.rows[0].name }}  →  "John"
{{ steps.query.outputs.rows[0].email }}  →  "john@example.com"

// Count
{{ steps.query.outputs.rowCount }}  →  5
```

---

## Practical Examples

### Validate data after creation via API

```
[HTTP Request: POST /api/users → body: { "name": "John" }]
    │
    ▼
[PostgreSQL: SELECT * FROM users WHERE name = 'John']
    │
    ▼
[If: {{ steps.query.outputs.rowCount }} > 0]
    │ true → [Log: "User successfully created in the database"]
    │ false → [Stop and Fail]
```

### Prepare data for testing

```
[PostgreSQL: INSERT INTO test_data (key, value) VALUES ('token', 'abc123')]
    │
    ▼
[HTTP Request: GET /api/verify?token={{ steps.insert.outputs.rows[0].token }}]
```

### Clean up data after testing

```
[... tests ...]
    │
    ▼
[PostgreSQL: DELETE FROM test_data WHERE created_by = 'automated-test']
```

---

## Tips

- **Use saved credentials** to manage connections in a centralized way
- In **Custom SQL**, write the query in the **SQL** field and use `{{ }}` for dynamic data
- The **visual query builder** is ideal for simple operations and is less prone to syntax errors
- **Use expressions** in values: `{{ steps.api.outputs.json.id }}` to use data from previous nodes
- **Test the connection** before running the flow using the test button on the credentials screen
