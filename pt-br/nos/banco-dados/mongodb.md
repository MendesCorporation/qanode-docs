# Nó MongoDB

O nó **MongoDB** permite executar operações no banco de dados MongoDB, incluindo consultas, inserções, atualizações, exclusões e aggregation pipelines.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `mongodb-query` |
| **Categoria** | Banco de Dados |
| **Cor** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Conexão

### Usando Credenciais Salvas (Recomendado)

Selecione uma credencial do tipo **MongoDB**.

### Configuração Manual

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **URI** | `string` | Connection string MongoDB |
| **Banco de Dados** | `string` | Nome do banco |
| **Coleção** | `string` | Nome da coleção |

**URI de exemplo:**
```
mongodb://usuario:senha@host:27017/banco?authSource=admin
mongodb+srv://usuario:senha@cluster.exemplo.mongodb.net/banco
```

---

## Operações

### find

Busca múltiplos documentos.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Filtro** | `JSON` | Critérios de busca |
| **Limite** | `number` | Máximo de documentos (padrão: 20) |
| **Opções** | `JSON` | Projeção, sort, etc. |

**Exemplo:**
```json
// Filtro
{ "active": true, "age": { "$gte": 18 } }

// Opções
{ "sort": { "name": 1 }, "projection": { "name": 1, "email": 1 } }
```

---

### findOne

Busca um único documento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Filtro** | `JSON` | Critérios de busca |

**Exemplo:**
```json
{ "_id": "abc123" }
{ "email": "joao@exemplo.com" }
```

---

### insertOne

Insere um novo documento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Documento** | `JSON` | Documento a ser inserido |

**Exemplo:**
```json
{
  "name": "João",
  "email": "joao@exemplo.com",
  "createdAt": "{{ new Date().toISOString() }}"
}
```

---

### updateOne

Atualiza um documento existente.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Filtro** | `JSON` | Critérios para encontrar o documento |
| **Update** | `JSON` | Operações de atualização |

**Exemplo:**
```json
// Filtro
{ "_id": "abc123" }

// Update
{ "$set": { "active": false, "updatedAt": "2024-01-01" } }
```

---

### deleteOne

Remove um documento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Filtro** | `JSON` | Critérios para encontrar o documento |

**Exemplo:**
```json
{ "email": "remover@exemplo.com" }
```

---

### aggregate

Executa um aggregation pipeline.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Pipeline** | `JSON` | Array de estágios |
| **Limite** | `number` | Máximo de resultados |

**Exemplo:**
```json
[
  { "$match": { "status": "active" } },
  { "$group": { "_id": "$department", "count": { "$sum": 1 } } },
  { "$sort": { "count": -1 } }
]
```

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `documents` | `array` | Documentos retornados (find, aggregate) |
| `document` | `any` | Documento único (findOne) |
| `result` | `object` | Resultado da operação (insert, update, delete) |

### Acessando os Outputs

```
// Lista de documentos
{{ steps.mongo.outputs.documents }}

// Primeiro documento
{{ steps.mongo.outputs.documents[0].name }}

// Documento único
{{ steps.mongo.outputs.document.email }}

// Resultado de insert
{{ steps.mongo.outputs.result.insertedId }}

// Resultado de update
{{ steps.mongo.outputs.result.modifiedCount }}
```

---

## Exemplos Práticos

### Verificar dados após API

```
[HTTP Request: POST /api/products]
    │
    ▼
[MongoDB: findOne → { "sku": "{{ steps.api.outputs.json.sku }}" }]
    │
    ▼
[If: {{ steps.mongo.outputs.document }} !== null]
    │ true → [Log: "Produto criado: {{ steps.mongo.outputs.document.name }}"]
```

### Preparar dados de teste

```
[MongoDB: insertOne → { "name": "Test User", "role": "tester" }]
    │
    ▼
[HTTP Request: GET /api/users/{{ steps.insert.outputs.result.insertedId }}]
```

### Relatório com aggregation

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

## Dicas

- O campo **Filtro** aceita todos os operadores MongoDB (`$eq`, `$ne`, `$gt`, `$in`, `$regex`, etc.)
- Use **expressões `{{ }}`** nos valores do JSON para injetar dados dinâmicos
- O **limite padrão** de 20 documentos pode ser alterado para evitar retornar dados excessivos
- Para operações complexas, use **aggregation pipelines**
- **Teste a conexão** na tela de credenciais antes de usar no fluxo
