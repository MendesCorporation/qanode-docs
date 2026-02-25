# Nodo MongoDB

El nodo **MongoDB** permite realizar operaciones en una base de datos MongoDB, incluyendo consultas, inserciones, actualizaciones, eliminaciones y aggregation pipelines.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `mongodb-query` |
| **Categoría** | Base de Datos |
| **Color** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Conexión

### Usando Credenciales Guardadas (Recomendado)

Seleccione una credencial de tipo **MongoDB**.

### Configuración Manual

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **URI** | `string` | Cadena de conexión MongoDB |
| **Base de Datos** | `string` | Nombre de la base de datos |
| **Colección** | `string` | Nombre de la colección |

**URIs de ejemplo:**
```
mongodb://usuario:contraseña@host:27017/basedatos?authSource=admin
mongodb+srv://usuario:contraseña@cluster.ejemplo.mongodb.net/basedatos
```

---

## Operaciones

### find

Busca múltiples documentos.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Filtro** | `JSON` | Criterios de búsqueda |
| **Límite** | `number` | Máximo de documentos (predeterminado: 20) |
| **Opciones** | `JSON` | Proyección, sort, etc. |

**Ejemplo:**
```json
// Filtro
{ "active": true, "age": { "$gte": 18 } }

// Opciones
{ "sort": { "name": 1 }, "projection": { "name": 1, "email": 1 } }
```

---

### findOne

Busca un único documento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Filtro** | `JSON` | Criterios de búsqueda |

**Ejemplo:**
```json
{ "_id": "abc123" }
{ "email": "juan@ejemplo.com" }
```

---

### insertOne

Inserta un nuevo documento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Documento** | `JSON` | Documento a ser insertado |

**Ejemplo:**
```json
{
  "name": "Juan",
  "email": "juan@ejemplo.com",
  "createdAt": "{{ new Date().toISOString() }}"
}
```

---

### updateOne

Actualiza un documento existente.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Filtro** | `JSON` | Criterios para encontrar el documento |
| **Update** | `JSON` | Operaciones de actualización |

**Ejemplo:**
```json
// Filtro
{ "_id": "abc123" }

// Update
{ "$set": { "active": false, "updatedAt": "2024-01-01" } }
```

---

### deleteOne

Elimina un documento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Filtro** | `JSON` | Criterios para encontrar el documento |

**Ejemplo:**
```json
{ "email": "eliminar@ejemplo.com" }
```

---

### aggregate

Ejecuta un aggregation pipeline.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Pipeline** | `JSON` | Array de etapas |
| **Límite** | `number` | Máximo de resultados |

**Ejemplo:**
```json
[
  { "$match": { "status": "active" } },
  { "$group": { "_id": "$department", "count": { "$sum": 1 } } },
  { "$sort": { "count": -1 } }
]
```

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `documents` | `array` | Documentos retornados (find, aggregate) |
| `document` | `any` | Documento único (findOne) |
| `result` | `object` | Resultado de la operación (insert, update, delete) |

### Accediendo a los Outputs

```
// Lista de documentos
{{ steps.mongo.outputs.documents }}

// Primer documento
{{ steps.mongo.outputs.documents[0].name }}

// Documento único
{{ steps.mongo.outputs.document.email }}

// Resultado de insert
{{ steps.mongo.outputs.result.insertedId }}

// Resultado de update
{{ steps.mongo.outputs.result.modifiedCount }}
```

---

## Ejemplos Prácticos

### Verificar datos después de una llamada a la API

```
[HTTP Request: POST /api/products]
    │
    ▼
[MongoDB: findOne → { "sku": "{{ steps.api.outputs.json.sku }}" }]
    │
    ▼
[If: {{ steps.mongo.outputs.document }} !== null]
    │ true → [Log: "Producto creado: {{ steps.mongo.outputs.document.name }}"]
```

### Preparar datos de prueba

```
[MongoDB: insertOne → { "name": "Test User", "role": "tester" }]
    │
    ▼
[HTTP Request: GET /api/users/{{ steps.insert.outputs.result.insertedId }}]
```

### Reporte con aggregation

```
[MongoDB: aggregate → [
    { "$match": { "date": { "$gte": "2024-01-01" } } },
    { "$group": { "_id": "$status", "count": { "$sum": 1 } } }
]]
    │
    ▼
[Log: "Resultados: {{ JSON.stringify(steps.aggregate.outputs.documents) }}"]
```

---

## Consejos

- El campo **Filtro** acepta todos los operadores MongoDB (`$eq`, `$ne`, `$gt`, `$in`, `$regex`, etc.)
- Use **expresiones `{{ }}`** en los valores del JSON para inyectar datos dinámicos
- El **límite predeterminado** de 20 documentos puede modificarse para evitar retornar datos excesivos
- Para operaciones complejas, use **aggregation pipelines**
- **Pruebe la conexión** en la pantalla de credenciales antes de usarla en el flujo
