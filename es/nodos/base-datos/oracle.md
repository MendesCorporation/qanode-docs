# Nodo Oracle

El nodo **Oracle** permite ejecutar consultas y operaciones en una base de datos Oracle Database.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `oracle-query` |
| **Categoría** | Base de Datos |
| **Color** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Conexión

### Usando Credenciales Guardadas (Recomendado)

Seleccione una credencial de tipo **Oracle**.

### Configuración Manual

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Connection String** | `string` | URI completa (TNS o Easy Connect) |
| **Host** | `string` | Dirección del servidor |
| **Puerto** | `number` | Puerto (predeterminado: 1521) |
| **Service Name** | `string` | Nombre del servicio Oracle |
| **Usuario** | `string` | Usuario |
| **Contraseña** | `string` | Contraseña |

**Connection String (Easy Connect):**
```
host:1521/service_name
```

---

## Modos de Consulta

Soporta los mismos presets que los demás nodos de base de datos:

- **Custom SQL** — SQL libre con parámetros
- **SELECT** — Generador de consultas visual
- **EXISTS** — Verificar existencia
- **COUNT** — Contar registros
- **ASSERT** — Verificar valor
- **INSERT** — Insertar registros
- **UPDATE** — Actualizar registros
- **DELETE** — Eliminar registros

**Nota:** En Oracle, los marcadores usan `:1`, `:2` (bind variables):

```sql
SELECT * FROM users WHERE email = :1 AND active = :2
```

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `rows` | `array` | Registros retornados |
| `rowCount` | `number` | Número de registros afectados/retornados |

---

## Diferencias de Oracle

- **Service Name** en lugar del nombre de la base de datos
- Las bind variables usan `:1`, `:2` en lugar de `$1` o `?`
- Sintaxis SQL de Oracle (ej: `ROWNUM` en lugar de `LIMIT`)
- El generador de consultas visual genera SQL compatible automáticamente
