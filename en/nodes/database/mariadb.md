# MariaDB Node

The **MariaDB** node allows you to execute queries and operations on a MariaDB database. The interface and features are identical to the [MySQL](mysql.md) node.

---

## Overview

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

> MariaDB is a fork of MySQL and uses the same SQL syntax. For full details, refer to the [PostgreSQL](postgresql.md) documentation.

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `rows` | `array` | Returned records |
| `rowCount` | `number` | Number of affected/returned records |
