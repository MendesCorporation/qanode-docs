# Provider API Contract

This page documents the complete HTTP API specification that a custom node provider must implement.

---

## Required Endpoints

### GET /manifest

Returns the list of available nodes in the provider.

**Response:**

```json
{
  "nodes": [
    {
      "type": "meu-no-customizado",
      "name": "Meu Nó Customizado",
      "category": "Custom Nodes",
      "timeoutMs": 600000,
      "inputSchema": {
        "campo1": {
          "type": "string",
          "required": true,
          "description": "Descrição do campo"
        },
        "campo2": {
          "type": "number",
          "default": 10,
          "description": "Campo numérico opcional"
        }
      },
      "outputSchema": {
        "resultado": { "type": "string" },
        "timestamp": { "type": "string" }
      }
    }
  ]
}
```

> In the example above, `timeoutMs: 600000` sets a 10-minute timeout for this node. If omitted, the default is 30 minutes.

#### Node Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | `string` | Yes | Unique node identifier |
| `name` | `string` | Yes | Name displayed in the palette |
| `category` | `string` | No | Category in the palette (default: "Custom Nodes") |
| `inputSchema` | `object` | No | Schema for input fields |
| `outputSchema` | `object` | No | Schema for output data |
| `timeoutMs` | `number` | No | Execution timeout in milliseconds (default: 18000000 = 30 minutes) |

#### Input Schema

Each field in `inputSchema` can have:

| Property | Type | Description |
|----------|------|-------------|
| `type` | `string` | Type: `string`, `number`, `boolean`, `object`, `array`, `any` |
| `required` | `boolean` | Whether the field is required |
| `default` | `any` | Default value |
| `description` | `string` | Description displayed as a tooltip |
| `enum` | `string[]` | List of allowed values (renders as a dropdown) |
| `items` | `object` | Item schema (for type: `array`) |

#### Output Schema

The `outputSchema` defines the data the node produces after execution. Each field has:

| Property | Type | Description |
|----------|------|-------------|
| `type` | `string` | Data type |

Outputs are accessible via expressions: `{{ steps.meuNo.outputs.resultado }}`

---

### POST /execute

Executes a node with the provided data.

**Request:**

```json
{
  "nodeType": "meu-no-customizado",
  "inputs": {
    "campo1": "valor do campo",
    "campo2": 42
  },
  "runId": "run_abc123",
  "nodeId": "node_xyz789"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `nodeType` | `string` | Type of node to execute |
| `inputs` | `object` | Input field values |
| `runId` | `string` | Current execution ID |
| `nodeId` | `string` | Node ID in the flow |

**Response (Success):**

```json
{
  "status": "success",
  "logs": [
    "Processando campo1: valor do campo",
    "Resultado calculado com sucesso"
  ],
  "outputs": {
    "resultado": "dados processados",
    "timestamp": "2024-01-15T10:30:00Z"
  },
  "artifacts": []
}
```

**Response (Failure):**

```json
{
  "status": "failed",
  "logs": [
    "Processando campo1: valor do campo",
    "ERRO: Campo inválido"
  ],
  "outputs": {},
  "error": {
    "message": "Descrição detalhada do erro"
  },
  "artifacts": []
}
```

#### Response Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | `string` | Yes | `"success"` or `"failed"` |
| `logs` | `string[]` | No | Log messages (displayed in the details view) |
| `outputs` | `object` | No | Data produced by the node |
| `error` | `object` | No | Error details (when `failed`) |
| `error.message` | `string` | — | Human-readable error message |
| `artifacts` | `array` | No | Generated files (screenshots, etc.) |

---

## Artifacts

The `artifacts` field allows the node to return files such as screenshots, PDFs, or other documents. Each artifact can be sent as inline **base64**:

```json
{
  "artifacts": [
    {
      "type": "screenshot",
      "name": "resultado.png",
      "base64": "iVBORw0KGgoAAAANSUhEUg..."
    },
    {
      "type": "file",
      "name": "relatorio.pdf",
      "base64": "JVBERi0xLjQKJeLj..."
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `type` | `string` | Type: `screenshot`, `pdf`, `video`, `file` |
| `name` | `string` | File name |
| `base64` | `string` | Base64-encoded content |

Artifacts are automatically saved to QANode's storage and are available in the execution results.

---

## Optional Endpoint

### GET /health

Checks whether the provider is running. Used during the connection test.

**Response:**
```json
{
  "ok": true,
  "nodeCount": 5
}
```

---

## Authentication

If the provider requires authentication, configure the **Token** when registering the provider in QANode. The token will be sent as the `Authorization: Bearer {token}` header on all requests.

---

## Required Field Validation

QANode validates fields marked as `required: true` in the `inputSchema` **before** calling `/execute`. If a required field is empty, the execution fails without calling the provider.

---

## Timeouts

| Endpoint | Timeout |
|----------|---------|
| `GET /manifest` | 5 seconds |
| `POST /execute` | 30 minutes (default) — configurable via `timeoutMs` in the manifest |

The `/execute` timeout can be customized **per node** through the `timeoutMs` field in the manifest. This is useful for nodes that perform long-running operations such as data processing, scraping, or integration with external systems.

```json
{
  "nodes": [
    {
      "type": "processar-lote",
      "name": "Processar Lote",
      "timeoutMs": 1800000,
      "inputSchema": { ... }
    }
  ]
}
```

In the example above, the node will have a 30-minute timeout (1,800,000ms). If `timeoutMs` is not defined, the default is **30 minutes** (300,000ms).

If the provider does not respond within the timeout, the request is aborted and the node is marked as failed.

### Global Flow Timeout

Regardless of each node's individual timeout, QANode enforces a **global 24-hour timeout** per flow execution. If a flow (or suite) exceeds 24 hours of total execution time, it will be automatically stopped and marked as failed.

---

## Headers Sent by QANode

```
Content-Type: application/json
Authorization: Bearer {token}  (if configured)
```

---

## Best Practices

1. **Unique type names**: Use prefixes to avoid conflicts (e.g., `my-company-validator`)
2. **Descriptive logs**: Include logs that help diagnose issues
3. **Error handling**: Always return `status: "failed"` with a clear `error.message`
4. **Typed inputs**: Define a complete `inputSchema` so QANode generates appropriate forms
5. **Output Schema**: Define `outputSchema` so QANode shows available fields in autocomplete
6. **Idempotency**: If possible, implement `/execute` in an idempotent manner
7. **Timeout**: Use `timeoutMs` in the manifest for nodes that need more than 30 minutes. Also implement internal timeouts to prevent operations from hanging
