# Nodo SQL Server

El nodo **SQL Server** permite ejecutar consultas y operaciones en bases Microsoft SQL Server. Ofrece **query builder visual** para operaciones comunes y también permite escribir **SQL customizado** cuando se necesita control total.

---

## Visión General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `sqlserver-query` |
| **Categoría** | Base de Datos |
| **Color** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Conexión

### Usando Credenciales Guardadas (Recomendado)

Seleccione una credencial de tipo **SQL Server** en el campo **Credencial**. La credencial centraliza los datos de conexión y evita exponer la contraseña directamente en el nodo.

### Configuración Manual

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Host** | `string` | Dirección del servidor SQL Server |
| **Puerto** | `number` | Puerto TCP (predeterminado: 1433) |
| **Base de Datos** | `string` | Nombre de la base |
| **Usuario** | `string` | Usuario de conexión |
| **Contraseña** | `string` | Contraseña de conexión |
| **Encrypt** | `boolean` | Usa conexión cifrada |
| **Trust Server Certificate** | `boolean` | Acepta el certificado del servidor sin validación completa |

En ambientes corporativos, es común usar `Encrypt` habilitado. En ambientes internos con certificado propio o auto-firmado, `Trust Server Certificate` puede ser necesario.

---

## Modos de Consulta

SQL Server soporta los mismos presets visuales que los otros nodos relacionales:

- **Custom SQL** — SQL libre con expresiones `{{ }}`;
- **SELECT** — query builder visual;
- **EXISTS** — verifica existencia de registro;
- **COUNT** — cuenta registros;
- **ASSERT** — valida valor en la base;
- **INSERT** — inserta registros;
- **UPDATE** — actualiza registros;
- **DELETE** — elimina registros.

El query builder genera SQL compatible con SQL Server automáticamente.

---

## Custom SQL

Use **Custom SQL** cuando necesite escribir la consulta manualmente.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **SQL** | `string` | Consulta SQL, con soporte para expresiones `{{ }}` |

### Usando Expresiones en SQL

En el panel actual del nodo SQL Server, la consulta customizada se escribe directamente en el campo **SQL**. Para usar valores dinámicos, escriba expresiones `{{ }}` dentro del propio SQL.

Ejemplo completo:

```sql
SELECT *
FROM dbo.Users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = 1;
```

Si `variables.EMAIL_TESTE` es `juan@ejemplo.com`, el SQL ejecutado será equivalente a:

```sql
SELECT *
FROM dbo.Users
WHERE email = 'juan@ejemplo.com'
  AND active = 1;
```

Para valores de pasos anteriores, use la misma sintaxis:

```sql
SELECT *
FROM dbo.Users
WHERE id = {{ steps.api.outputs.json.id }};
```

Para textos, fechas y valores que deben tratarse como string en SQL, coloque comillas:

```sql
SELECT *
FROM dbo.Orders
WHERE customer_email = '{{ steps.api.outputs.json.email }}'
  AND created_at >= '{{ variables.FECHA_INICIAL }}';
```

Para números y flags, normalmente no use comillas:

```sql
SELECT *
FROM dbo.Users
WHERE id = {{ variables.USER_ID }}
  AND active = {{ variables.ACTIVE_FLAG }};
```

> **Cuidado:** como la expresión se aplica directamente en el SQL, use valores controlados y revise las comillas en campos de texto. Evite usar contenido libre ingresado por usuarios finales sin validación.

---

## SELECT Visual

En el preset **SELECT**, puede montar consultas sin escribir SQL manualmente.

| Campo | Descripción |
|-------|-------------|
| **Tabla** | Tabla de origen |
| **Columnas** | Columnas retornadas; vacío retorna todas |
| **Where** | Condiciones de filtro |
| **Order By** | Ordenación |
| **Limit** | Cantidad máxima de registros |

En SQL Server, el límite se genera como `TOP (...)`, no como `LIMIT`.

Ejemplo generado:

```sql
SELECT TOP (10) [id], [email], [active]
FROM [dbo].[Users]
WHERE [active] = 1;
```

---

## Diferencias de SQL Server

| Tema | Comportamiento |
|------|----------------|
| **Puerto predeterminado** | `1433` |
| **Límite de filas** | `TOP (...)` |
| **Identificadores** | Columnas y tablas pueden generarse con corchetes, como `[Users]` |
| **Booleanos** | En muchas bases SQL Server, valores booleanos se tratan como `1` y `0` en columnas `bit` |
| **TLS/certificado** | Puede exigir `Encrypt` y `Trust Server Certificate`, según el ambiente |

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `rows` | `array` | Registros retornados por la consulta |
| `rowCount` | `number` | Cantidad de filas retornadas en el primer recordset |
| `recordsAffected` | `array` | Filas afectadas por comandos de escritura, cuando lo informa el driver |

### Accediendo a Outputs

```
{{ steps["sqlserver-query"].outputs.rows }}
{{ steps["sqlserver-query"].outputs.rows[0].email }}
{{ steps["sqlserver-query"].outputs.rowCount }}
{{ steps["sqlserver-query"].outputs.recordsAffected }}
```

---

## Ejemplos

### Validar Registro Creado por API

```
[HTTP Request: POST /users]
    │
    ▼
[SQL Server: SELECT * FROM dbo.Users WHERE email = '{{ steps["http-request"].outputs.json.email }}']
    │
    ▼
[If: {{ steps["sqlserver-query"].outputs.rowCount > 0 }}]
```

### Preparar Datos de Prueba

```sql
INSERT INTO dbo.TestData ([key], [value])
VALUES ('pedido_test', '{{ variables.PEDIDO_ID }}');
```

### Limpiar Datos de Prueba

```sql
DELETE FROM dbo.TestData
WHERE [key] = 'pedido_test';
```

---

## Consejos

- Use credenciales guardadas siempre que sea posible.
- Pruebe la conexión antes de ejecutar el escenario.
- Coloque comillas en expresiones que representan texto o fechas.
- En ambientes con certificado interno, revise las opciones `Encrypt` y `Trust Server Certificate`.
- Para consultas simples, prefiera el query builder visual; para joins, procedures o SQL avanzado, use Custom SQL.
