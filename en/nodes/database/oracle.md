# Oracle Node

The **Oracle** node allows you to execute queries and operations on an Oracle Database.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `oracle-query` |
| **Category** | Database |
| **Color** | 🟢 Green (#22c55e) |
| **Input** | `in` |
| **Output** | `out` |

---

## Connection

### Using Saved Credentials (Recommended)

Select a credential of type **Oracle**.

### Manual Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Connection String** | `string` | Full URI (TNS or Easy Connect) |
| **Host** | `string` | Server address |
| **Port** | `number` | Port (default: 1521) |
| **Service Name** | `string` | Oracle service name |
| **User** | `string` | User |
| **Password** | `string` | Password |

**Connection String (Easy Connect):**
```
host:1521/service_name
```

---

## Query Modes

Supports the same presets as the other database nodes:

- **Custom SQL** — Free SQL with parameters
- **SELECT** — Visual query builder
- **EXISTS** — Check existence
- **COUNT** — Count records
- **ASSERT** — Check value
- **INSERT** — Insert records
- **UPDATE** — Update records
- **DELETE** — Remove records

**Note:** In Oracle, placeholders use `:1`, `:2` (bind variables):

```sql
SELECT * FROM users WHERE email = :1 AND active = :2
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `rows` | `array` | Returned records |
| `rowCount` | `number` | Number of affected/returned records |

---

## Oracle Differences

- **Service Name** instead of database name
- Bind variables use `:1`, `:2` instead of `$1` or `?`
- Oracle SQL syntax (e.g., `ROWNUM` instead of `LIMIT`)
- The visual query builder automatically generates compatible SQL
