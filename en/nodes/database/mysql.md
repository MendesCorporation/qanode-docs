# MySQL Node

The **MySQL** node allows you to execute queries and operations on a MySQL database. The interface and features are identical to the [PostgreSQL](postgresql.md) node.

---

## Overview

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

- **Custom SQL** — Free SQL with parameters
- **SELECT** — Visual query builder
- **EXISTS** — Check existence
- **COUNT** — Count records
- **ASSERT** — Check value
- **INSERT** — Insert records
- **UPDATE** — Update records
- **DELETE** — Remove records

> For full details on each mode, refer to the [PostgreSQL](postgresql.md) node documentation.

**Note:** In MySQL, parameter placeholders use `?` instead of `$1`, `$2`:

```sql
SELECT * FROM users WHERE email = ? AND active = ?
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `rows` | `array` | Returned records |
| `rowCount` | `number` | Number of affected/returned records |

---

## Tips

- Remember the placeholder difference: MySQL uses `?`, PostgreSQL uses `$1`, `$2`
- The visual query builder automatically generates correct SQL for MySQL
- For more details on each query mode, see the [PostgreSQL](postgresql.md) documentation
