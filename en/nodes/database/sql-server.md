# SQL Server Node

The **SQL Server** node runs queries and operations against Microsoft SQL Server databases. It provides a **visual query builder** for common operations and also supports **Custom SQL** when you need full control.

---

## Overview

[Overview](../../assets/images/nos-sql-server-visao-geral.mp4)

| Property | Value |
|----------|-------|
| **Type** | `sqlserver-query` |
| **Category** | Database |
| **Color** | 🟢 Green (#22c55e) |
| **Input** | `in` |
| **Output** | `out` |

---

## Connection

### Using Saved Credentials (Recommended)

Select a **SQL Server** credential in the **Credential** field. The credential centralizes connection data and avoids exposing the password directly in the node.

### Manual Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Host** | `string` | SQL Server address |
| **Port** | `number` | TCP port (default: 1433) |
| **Database** | `string` | Database name |
| **User** | `string` | Connection user |
| **Password** | `string` | Connection password |
| **Encrypt** | `boolean` | Uses encrypted connection |
| **Trust Server Certificate** | `boolean` | Accepts the server certificate without full validation |

In corporate environments, `Encrypt` is often required. In internal environments with self-signed or private certificates, `Trust Server Certificate` may be necessary.

---

## Query Modes

SQL Server supports the same visual presets as the other relational database nodes:

- **Custom SQL** — free SQL with `{{ }}` expressions;
- **SELECT** — visual query builder;
- **EXISTS** — checks whether a record exists;
- **COUNT** — counts records;
- **ASSERT** — validates a value in the database;
- **INSERT** — inserts records;
- **UPDATE** — updates records;
- **DELETE** — deletes records.

The query builder automatically generates SQL compatible with SQL Server.

---

## Custom SQL

Use **Custom SQL** when you need to write the query manually.

| Field | Type | Description |
|-------|------|-------------|
| **SQL** | `string` | SQL query, with support for `{{ }}` expressions |

### Using Expressions in SQL

In the current SQL Server node panel, a custom query is written directly in the **SQL** field. To use dynamic values, write `{{ }}` expressions inside the SQL itself.

Complete example:

```sql
SELECT *
FROM dbo.Users
WHERE email = '{{ variables.TEST_EMAIL }}'
  AND active = 1;
```

If `variables.TEST_EMAIL` is `john@example.com`, the executed SQL is equivalent to:

```sql
SELECT *
FROM dbo.Users
WHERE email = 'john@example.com'
  AND active = 1;
```

For values from previous steps, use the same syntax:

```sql
SELECT *
FROM dbo.Users
WHERE id = {{ steps.api.outputs.json.id }};
```

For text, dates, and values that SQL should treat as strings, add quotes:

```sql
SELECT *
FROM dbo.Orders
WHERE customer_email = '{{ steps.api.outputs.json.email }}'
  AND created_at >= '{{ variables.START_DATE }}';
```

For numbers and flags, usually do not add quotes:

```sql
SELECT *
FROM dbo.Users
WHERE id = {{ variables.USER_ID }}
  AND active = {{ variables.ACTIVE_FLAG }};
```

> **Careful:** because expressions are applied directly inside SQL, use controlled values and review quotes for text fields. Avoid injecting free-form end-user input without validation.

---

## Visual SELECT

In the **SELECT** preset, you can build queries without writing SQL manually.

| Field | Description |
|-------|-------------|
| **Table** | Source table |
| **Columns** | Returned columns; empty returns all columns |
| **Where** | Filter conditions |
| **Order By** | Sorting |
| **Limit** | Maximum number of records |

In SQL Server, the limit is generated as `TOP (...)`, not as `LIMIT`.

Generated example:

```sql
SELECT TOP (10) [id], [email], [active]
FROM [dbo].[Users]
WHERE [active] = 1;
```

---

## SQL Server Differences

| Topic | Behavior |
|-------|----------|
| **Default port** | `1433` |
| **Row limit** | `TOP (...)` |
| **Identifiers** | Columns and tables may be generated with brackets, such as `[Users]` |
| **Booleans** | Many SQL Server databases use `1` and `0` in `bit` columns |
| **TLS/certificate** | May require `Encrypt` and `Trust Server Certificate`, depending on the environment |

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `rows` | `array` | Records returned by the query |
| `rowCount` | `number` | Number of rows returned in the first recordset |
| `recordsAffected` | `array` | Rows affected by write commands, when reported by the driver |

### Accessing Outputs

```
{{ steps["sqlserver-query"].outputs.rows }}
{{ steps["sqlserver-query"].outputs.rows[0].email }}
{{ steps["sqlserver-query"].outputs.rowCount }}
{{ steps["sqlserver-query"].outputs.recordsAffected }}
```

---

## Examples

### Validate a Record Created by API

```
[HTTP Request: POST /users]
    │
    ▼
[SQL Server: SELECT * FROM dbo.Users WHERE email = '{{ steps["http-request"].outputs.json.email }}']
    │
    ▼
[If: {{ steps["sqlserver-query"].outputs.rowCount > 0 }}]
```

### Prepare Test Data

```sql
INSERT INTO dbo.TestData ([key], [value])
VALUES ('test_order', '{{ variables.ORDER_ID }}');
```

### Clean Up Test Data

```sql
DELETE FROM dbo.TestData
WHERE [key] = 'test_order';
```

---

## Tips

- Use saved credentials whenever possible.
- Test the connection before running the scenario.
- Add quotes around expressions that represent text or dates.
- In environments with internal certificates, review `Encrypt` and `Trust Server Certificate`.
- For simple queries, prefer the visual query builder; for joins, procedures, or advanced SQL, use Custom SQL.
