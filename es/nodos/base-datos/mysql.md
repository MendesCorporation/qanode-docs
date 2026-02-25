# Nodo MySQL

El nodo **MySQL** permite ejecutar consultas y operaciones en una base de datos MySQL. La interfaz y funcionalidades son idénticas al nodo [PostgreSQL](postgresql.md).

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `mysql-query` |
| **Categoría** | Base de Datos |
| **Color** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Conexión

### Usando Credenciales Guardadas (Recomendado)

Seleccione una credencial de tipo **MySQL** en el campo **Credencial**.

### Configuración Manual

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Connection String** | `string` | URI completa |
| **Host** | `string` | Dirección del servidor |
| **Puerto** | `number` | Puerto (predeterminado: 3306) |
| **Base de Datos** | `string` | Nombre de la base de datos |
| **Usuario** | `string` | Usuario |
| **Contraseña** | `string` | Contraseña |
| **SSL** | `boolean` | Usar conexión SSL |

---

## Modos de Consulta

MySQL soporta los mismos presets que PostgreSQL:

- **Custom SQL** — SQL libre con parámetros
- **SELECT** — Generador de consultas visual
- **EXISTS** — Verificar existencia
- **COUNT** — Contar registros
- **ASSERT** — Verificar valor
- **INSERT** — Insertar registros
- **UPDATE** — Actualizar registros
- **DELETE** — Eliminar registros

> Para detalles completos de cada modo, consulte la documentación del nodo [PostgreSQL](postgresql.md).

**Nota:** En MySQL, los marcadores de parámetros usan `?` en lugar de `$1`, `$2`:

```sql
SELECT * FROM users WHERE email = ? AND active = ?
```

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `rows` | `array` | Registros retornados |
| `rowCount` | `number` | Número de registros afectados/retornados |

---

## Consejos

- Recuerde la diferencia de marcadores: MySQL usa `?`, PostgreSQL usa `$1`, `$2`
- El generador de consultas visual genera SQL correcto para MySQL automáticamente
- Para más detalles sobre cada modo de consulta, vea la documentación de [PostgreSQL](postgresql.md)
