# MySQL Node

The **MySQL** node allows you to execute queries and operations on a MySQL database. The interface and features are identical to the [PostgreSQL](postgresql.md) node.

---

## Overview

[Overview](../../assets/images/nos-my-sql-visao-geral.mp4)

| Property | Value |
|----------|-------|
| **Type** | `mysql-query` |
| **Category** | Database |
| **Color** | 🟢 Green (#22c55e) |
| **Input** | `in` |
| **Output** | `out` |

---

## Connection

### Using Saved Credentials (Recommended)

Select a credential of type **MySQL** in the **Credential** field.

### Manual Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Connection String** | `string` | Full URI |
| **Host** | `string` | Server address |
| **Port** | `number` | Port (default: 3306) |
| **Database** | `string` | Database name |
| **User** | `string` | User |
| **Password** | `string` | Password |
| **SSL** | `boolean` | Use SSL connection |

---

## Query Modes

MySQL supports the same presets as PostgreSQL:

- **Custom SQL** — Free SQL with `{{ }}` expressions directly in the query
- **SELECT** — Visual query builder
- **EXISTS** — Check existence
- **COUNT** — Count records
- **ASSERT** — Check value
- **INSERT** — Insert records
- **UPDATE** — Update records
- **DELETE** — Remove records

> For full details on each mode, refer to the [PostgreSQL](postgresql.md) node documentation.

**Custom SQL example with variable:**

```sql
SELECT *
FROM users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = 1;
```

Use quotes for text and dates. For numbers and flags, normally use the expression without quotes:

```sql
SELECT *
FROM orders
WHERE customer_id = {{ variables.CUSTOMER_ID }};
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `rows` | `array` | Returned records |
| `rowCount` | `number` | Number of affected/returned records |

---

## Tips

- In **Custom SQL**, write the query in the **SQL** field and use `{{ }}` for dynamic data
- Review quotes in text and date fields
- The visual query builder automatically generates correct SQL for MySQL
- For more details on each query mode, see the [PostgreSQL](postgresql.md) documentation
