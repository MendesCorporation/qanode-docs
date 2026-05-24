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

En modo **Custom SQL**, escriba la consulta directamente en el campo **SQL** y use expresiones `{{ }}` para valores dinámicos.

```sql
SELECT *
FROM users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = 1;
```

Para textos y fechas, coloque comillas alrededor de la expresión. Para números y flags, normalmente use la expresión sin comillas.

> MariaDB es un fork de MySQL y usa sintaxis SQL compatible. Para detalles completos de cada preset, consulte la documentación de [PostgreSQL](postgresql.md).

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `rows` | `array` | Registros retornados |
| `rowCount` | `number` | Número de registros afectados/retornados |
