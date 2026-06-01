# MariaDB Node

The **MariaDB** node allows you to execute queries and operations on a MariaDB database. The interface and features are identical to the [MySQL](mysql.md) node.

---

## Overview

[Overview](../../assets/images/nos-maria-db-visao-geral.mp4)

| Property | Value |
|----------|-------|
| **Type** | `mariadb-query` |
| **Category** | Database |
| **Color** | 🟢 Green (#22c55e) |
| **Input** | `in` |
| **Output** | `out` |

---

## Connection

| Field | Type | Description |
|-------|------|-------------|
| **Credential** | `string` | Credential of type **MariaDB** |
| **Host** | `string` | Server address |
| **Port** | `number` | Port (default: 3306) |
| **Database** | `string` | Database name |
| **User** | `string` | User |
| **Password** | `string` | Password |
| **SSL** | `boolean` | Use SSL connection |

---

## Query Modes

Supports the same presets: Custom SQL, SELECT, EXISTS, COUNT, ASSERT, INSERT, UPDATE, DELETE.

In **Custom SQL** mode, write the query directly in the **SQL** field and use `{{ }}` expressions for dynamic values.

```sql
SELECT *
FROM users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = 1;
```

For text and dates, wrap the expression in quotes. For numbers and flags, normally use the expression without quotes.

> MariaDB is a fork of MySQL and uses compatible SQL syntax. For full details on each preset, refer to the [PostgreSQL](postgresql.md) documentation.

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `rows` | `array` | Returned records |
| `rowCount` | `number` | Number of affected/returned records |
