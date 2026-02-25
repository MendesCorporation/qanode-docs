# Nodo MariaDB

El nodo **MariaDB** permite ejecutar consultas y operaciones en una base de datos MariaDB. La interfaz y funcionalidades son idénticas al nodo [MySQL](mysql.md).

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `mariadb-query` |
| **Categoría** | Base de Datos |
| **Color** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Conexión

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Credencial** | `string` | Credencial de tipo **MariaDB** |
| **Host** | `string` | Dirección del servidor |
| **Puerto** | `number` | Puerto (predeterminado: 3306) |
| **Base de Datos** | `string` | Nombre de la base de datos |
| **Usuario** | `string` | Usuario |
| **Contraseña** | `string` | Contraseña |
| **SSL** | `boolean` | Usar conexión SSL |

---

## Modos de Consulta

Soporta los mismos presets: Custom SQL, SELECT, EXISTS, COUNT, ASSERT, INSERT, UPDATE, DELETE.

> MariaDB es un fork de MySQL y usa la misma sintaxis SQL. Para detalles completos, consulte la documentación de [PostgreSQL](postgresql.md).

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `rows` | `array` | Registros retornados |
| `rowCount` | `number` | Número de registros afectados/retornados |
