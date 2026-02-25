# MongoDB Node

The **MongoDB** node allows you to perform operations on a MongoDB database, including queries, insertions, updates, deletions, and aggregation pipelines.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `mongodb-query` |
| **Category** | Database |
| **Color** | 🟢 Green (#22c55e) |
| **Input** | `in` |
| **Output** | `out` |

---

## Connection

### Using Saved Credentials (Recommended)

Select a credential of type **MongoDB**.

### Manual Configuration

| Field | Type | Description |
|-------|------|-------------|
| **URI** | `string` | MongoDB connection string |
| **Database** | `string` | Database name |
| **Collection** | `string` | Collection name |

**Example URIs:**
```
mongodb://user:password@host:27017/database?authSource=admin
mongodb+srv://user:password@cluster.example.mongodb.net/database
```

---

## Operations

### find

Finds multiple documents.

| Field | Type | Description |
|-------|------|-------------|
| **Filter** | `JSON` | Search criteria |
| **Limit** | `number` | Maximum number of documents (default: 20) |
| **Options** | `JSON` | Projection, sort, etc. |

**Example:**
```json
// Filter
{ "active": true, "age": { "$gte": 18 } }

// Options
{ "sort": { "name": 1 }, "projection": { "name": 1, "email": 1 } }
```

---

### findOne

Finds a single document.

| Field | Type | Description |
|-------|------|-------------|
| **Filter** | `JSON` | Search criteria |

**Example:**
```json
{ "_id": "abc123" }
{ "email": "john@example.com" }
```

---

### insertOne

Inserts a new document.

| Field | Type | Description |
|-------|------|-------------|
| **Document** | `JSON` | Document to be inserted |

**Example:**
```json
{
  "name": "John",
  "email": "john@example.com",
  "createdAt": "{{ new Date().toISOString() }}"
}
```

---

### updateOne

Updates an existing document.

| Field | Type | Description |
|-------|------|-------------|
| **Filter** | `JSON` | Criteria to find the document |
| **Update** | `JSON` | Update operations |

**Example:**
```json
// Filter
{ "_id": "abc123" }

// Update
{ "$set": { "active": false, "updatedAt": "2024-01-01" } }
```

---

### deleteOne

Removes a document.

| Field | Type | Description |
|-------|------|-------------|
| **Filter** | `JSON` | Criteria to find the document |

**Example:**
```json
{ "email": "remove@example.com" }
```

---

### aggregate

Executes an aggregation pipeline.

| Field | Type | Description |
|-------|------|-------------|
| **Pipeline** | `JSON` | Array of stages |
| **Limit** | `number` | Maximum number of results |

**Example:**
```json
[
  { "$match": { "status": "active" } },
  { "$group": { "_id": "$department", "count": { "$sum": 1 } } },
  { "$sort": { "count": -1 } }
]
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `documents` | `array` | Returned documents (find, aggregate) |
| `document` | `any` | Single document (findOne) |
| `result` | `object` | Operation result (insert, update, delete) |

### Accessing Outputs

```
// List of documents
{{ steps.mongo.outputs.documents }}

// First document
{{ steps.mongo.outputs.documents[0].name }}

// Single document
{{ steps.mongo.outputs.document.email }}

// Insert result
{{ steps.mongo.outputs.result.insertedId }}

// Update result
{{ steps.mongo.outputs.result.modifiedCount }}
```

---

## Practical Examples

### Verify data after API call

```
[HTTP Request: POST /api/products]
    │
    ▼
[MongoDB: findOne → { "sku": "{{ steps.api.outputs.json.sku }}" }]
    │
    ▼
[If: {{ steps.mongo.outputs.document }} !== null]
    │ true → [Log: "Product created: {{ steps.mongo.outputs.document.name }}"]
```

### Prepare test data

```
[MongoDB: insertOne → { "name": "Test User", "role": "tester" }]
    │
    ▼
[HTTP Request: GET /api/users/{{ steps.insert.outputs.result.insertedId }}]
```

### Report with aggregation

```
[MongoDB: aggregate → [
    { "$match": { "date": { "$gte": "2024-01-01" } } },
    { "$group": { "_id": "$status", "count": { "$sum": 1 } } }
]]
    │
    ▼
[Log: "Results: {{ JSON.stringify(steps.aggregate.outputs.documents) }}"]
```

---

## Tips

- The **Filter** field accepts all MongoDB operators (`$eq`, `$ne`, `$gt`, `$in`, `$regex`, etc.)
- Use **`{{ }}` expressions** in JSON values to inject dynamic data
- The **default limit** of 20 documents can be changed to avoid returning excessive data
- For complex operations, use **aggregation pipelines**
- **Test the connection** on the credentials screen before using it in the flow
