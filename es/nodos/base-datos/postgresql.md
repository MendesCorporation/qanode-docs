# Nodo PostgreSQL

El nodo **PostgreSQL** permite ejecutar consultas y operaciones en una base de datos PostgreSQL. Ofrece tanto un **generador de consultas visual** como la opción de escribir **SQL personalizado**.

---

## Descripción General

[Descripción General](../../assets/images/nos-postgres-visao-geral.mp4)

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `postgres-query` |
| **Categoría** | Base de Datos |
| **Color** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Conexión

### Usando Credenciales Guardadas (Recomendado)

Seleccione una credencial de tipo **PostgreSQL** en el campo **Credencial**. La credencial contiene toda la información de conexión.

### Configuración Manual

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Connection String** | `string` | URI completa (alternativa) |
| **Host** | `string` | Dirección del servidor |
| **Puerto** | `number` | Puerto (predeterminado: 5432) |
| **Base de Datos** | `string` | Nombre de la base de datos |
| **Usuario** | `string` | Usuario de conexión |
| **Contraseña** | `string` | Contraseña de conexión |
| **SSL** | `boolean` | Usar conexión SSL |

**Connection String:**
```
postgresql://usuario:contraseña@host:5432/basedatos?sslmode=require
```

---

## Modos de Consulta (Presets)

### Custom SQL

Escriba SQL libremente en el campo **SQL**. Para usar datos dinámicos, coloque expresiones `{{ }}` directamente en la query.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **SQL** | `string` | Consulta SQL (soporta `{{ }}`) |

**Ejemplo con variable:**
```sql
SELECT *
FROM users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = true;
```

Si `variables.EMAIL_TESTE` es `juan@ejemplo.com`, el SQL ejecutado será equivalente a:

```sql
SELECT *
FROM users
WHERE email = 'juan@ejemplo.com'
  AND active = true;
```

**Ejemplo con output de otro nodo:**

```sql
SELECT *
FROM users
WHERE id = {{ steps.api.outputs.body.id }};
```

Para textos, fechas y valores tratados como string, coloque comillas alrededor de la expresión. Para números y booleanos, normalmente use la expresión sin comillas.

> **Cuidado:** como la expresión se aplica directamente al SQL, use valores controlados y revise comillas en campos de texto. Evite usar contenido libre digitado por usuario final sin validación.

---

### SELECT (Generador Visual)

Construya consultas SELECT sin escribir SQL:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Tabla** | `string` | Tabla de origen |
| **Columnas** | `array` | Columnas a seleccionar (vacío = todas) |
| **Condiciones WHERE** | `array` | Filtros |
| **ORDER BY** | `object` | Columna y dirección de ordenamiento |
| **LIMIT** | `number` | Límite de registros |

**Condiciones WHERE:**

| Campo | Operador | Valor |
|-------|----------|-------|
| `email` | `=` | `juan@ejemplo.com` |
| `age` | `>` | `18` |
| `name` | `LIKE` | `%García%` |

---

### EXISTS

Verifica si existe al menos un registro que coincida con los criterios.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Tabla** | `string` | Tabla |
| **Condiciones WHERE** | `array` | Criterios de búsqueda |

Retorna `rowCount: 1` si existe, `rowCount: 0` si no existe.

---

### COUNT

Cuenta los registros que coinciden con los criterios.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Tabla** | `string` | Tabla |
| **Condiciones WHERE** | `array` | Criterios de conteo |

---

### ASSERT

Verifica si un valor en la base de datos coincide con el esperado. Ideal para validaciones de datos.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Tabla** | `string` | Tabla |
| **Columna** | `string` | Columna a verificar |
| **Operador** | `string` | Operación de comparación |
| **Valor** | `string` | Valor esperado |
| **Condiciones WHERE** | `array` | Filtros para localizar el registro |

---

### INSERT

Inserta nuevos registros.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Tabla** | `string` | Tabla de destino |
| **Valores** | `array` | Pares columna-valor |

**Ejemplo:**

| Columna | Valor |
|---------|-------|
| `name` | `Maria` |
| `email` | `maria@ejemplo.com` |
| `active` | `true` |

---

### UPDATE

Actualiza registros existentes.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Tabla** | `string` | Tabla |
| **Sets** | `array` | Pares columna-valor a actualizar |
| **Condiciones WHERE** | `array` | Filtros |

---

### DELETE

Elimina registros.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Tabla** | `string` | Tabla |
| **Condiciones WHERE** | `array` | Filtros (obligatorio por seguridad) |

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `rows` | `array` | Array de registros retornados |
| `rowCount` | `number` | Número de registros afectados/retornados |

### Accediendo a los Outputs

```
// Todos los registros
{{ steps.query.outputs.rows }}

// Primer registro
{{ steps.query.outputs.rows[0] }}

// Campo específico del primer registro
{{ steps.query.outputs.rows[0].name }}  →  "Juan"
{{ steps.query.outputs.rows[0].email }}  →  "juan@ejemplo.com"

// Conteo
{{ steps.query.outputs.rowCount }}  →  5
```

---

## Ejemplos Prácticos

### Validar datos después de la creación vía API

```
[HTTP Request: POST /api/users → body: { "name": "Juan" }]
    │
    ▼
[PostgreSQL: SELECT * FROM users WHERE name = 'Juan']
    │
    ▼
[If: {{ steps.query.outputs.rowCount }} > 0]
    │ true → [Log: "Usuario creado exitosamente en la base de datos"]
    │ false → [Stop and Fail]
```

### Preparar datos para pruebas

```
[PostgreSQL: INSERT INTO test_data (key, value) VALUES ('token', 'abc123')]
    │
    ▼
[HTTP Request: GET /api/verify?token={{ steps.insert.outputs.rows[0].token }}]
```

### Limpiar datos después de las pruebas

```
[... pruebas ...]
    │
    ▼
[PostgreSQL: DELETE FROM test_data WHERE created_by = 'automated-test']
```

---

## Consejos

- **Use credenciales guardadas** para gestionar conexiones de forma centralizada
- En **Custom SQL**, escriba la query en el campo **SQL** y use `{{ }}` para datos dinámicos
- El **generador de consultas visual** es ideal para operaciones simples y es menos propenso a errores de sintaxis
- **Use expresiones** en los valores: `{{ steps.api.outputs.body.id }}` para usar datos de nodos anteriores
- **Pruebe la conexión** antes de ejecutar el flujo usando el botón de prueba en la pantalla de credenciales
