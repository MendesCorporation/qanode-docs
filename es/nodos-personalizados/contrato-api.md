# Contrato de la API de Proveedores

Esta página documenta la especificación completa de la API HTTP que un proveedor de nodos personalizados debe implementar.

---

## Endpoints Requeridos

### GET /manifest

Retorna la lista de nodos disponibles en el proveedor.

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

> En el ejemplo anterior, `timeoutMs: 600000` define un timeout de 10 minutos para este nodo. Si se omite, el valor predeterminado es 30 minutos.

#### Campos del Nodo

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `type` | `string` | Sí | Identificador único del nodo |
| `name` | `string` | Sí | Nombre mostrado en la paleta |
| `category` | `string` | No | Categoría en la paleta (predeterminado: "Custom Nodes") |
| `inputSchema` | `object` | No | Esquema de los campos de entrada |
| `outputSchema` | `object` | No | Esquema de los datos de salida |
| `timeoutMs` | `number` | No | Timeout de ejecución en milisegundos (predeterminado: 18000000 = 30 minutos) |

#### Input Schema

Cada campo del `inputSchema` puede tener:

| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `type` | `string` | Tipo: `string`, `number`, `boolean`, `object`, `array`, `any` |
| `required` | `boolean` | Si el campo es obligatorio |
| `default` | `any` | Valor predeterminado |
| `description` | `string` | Descripción mostrada como tooltip |
| `enum` | `string[]` | Lista de valores permitidos (se renderiza como dropdown) |
| `items` | `object` | Esquema de los ítems (para type: `array`) |

#### Output Schema

El `outputSchema` define los datos que el nodo produce tras la ejecución. Cada campo tiene:

| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `type` | `string` | Tipo del dato |

Los outputs son accesibles mediante expresiones: `{{ steps.meuNo.outputs.resultado }}`

---

### POST /execute

Ejecuta un nodo con los datos proporcionados.

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

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `nodeType` | `string` | Tipo del nodo a ejecutar |
| `inputs` | `object` | Valores de los campos de entrada |
| `runId` | `string` | ID de la ejecución actual |
| `nodeId` | `string` | ID del nodo en el flujo |

**Response (Éxito):**

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

**Response (Fallo):**

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

#### Campos de la Response

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `status` | `string` | Sí | `"success"` o `"failed"` |
| `logs` | `string[]` | No | Mensajes de log (mostrados en los detalles) |
| `outputs` | `object` | No | Datos producidos por el nodo |
| `error` | `object` | No | Detalles del error (cuando `failed`) |
| `error.message` | `string` | — | Mensaje de error legible |
| `artifacts` | `array` | No | Archivos generados (screenshots, etc.) |

---

## Artefactos

El campo `artifacts` permite que el nodo retorne archivos como screenshots, PDFs u otros documentos. Cada artefacto puede enviarse como **base64** inline:

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

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `type` | `string` | Tipo: `screenshot`, `pdf`, `video`, `file` |
| `name` | `string` | Nombre del archivo |
| `base64` | `string` | Contenido codificado en base64 |

Los artefactos se guardan automáticamente en el storage de QANode y quedan disponibles en los resultados de la ejecución.

---

## Endpoint Opcional

### GET /health

Verifica si el proveedor está funcionando. Se usa en la prueba de conexión.

**Response:**
```json
{
  "ok": true,
  "nodeCount": 5
}
```

---

## Autenticación

Si el proveedor requiere autenticación, configura el **Token** al registrar el proveedor en QANode. El token se enviará como header `Authorization: Bearer {token}` en todas las solicitudes.

---

## Validación de Campos Requeridos

QANode valida los campos marcados como `required: true` en el `inputSchema` **antes** de llamar a `/execute`. Si un campo requerido está vacío, la ejecución falla sin llamar al proveedor.

---

## Timeouts

| Endpoint | Timeout |
|----------|---------|
| `GET /manifest` | 5 segundos |
| `POST /execute` | 30 minutos (predeterminado) — configurable mediante `timeoutMs` en el manifiesto |

El timeout de `/execute` puede personalizarse **por nodo** a través del campo `timeoutMs` en el manifiesto. Esto es útil para nodos que ejecutan operaciones largas como procesamiento de datos, scraping o integración con sistemas externos.

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

En el ejemplo anterior, el nodo tendrá un timeout de 30 minutos (1.800.000ms). Si `timeoutMs` no está definido, el valor predeterminado es **30 minutos** (300.000ms).

Si el proveedor no responde dentro del timeout, la solicitud se cancela y el nodo se marca como fallido.

### Timeout Global del Flujo

Independientemente del timeout de cada nodo, QANode impone un **timeout global de 24 horas** por ejecución de flujo. Si un flujo (o suite) supera las 24 horas de ejecución total, será interrumpido automáticamente y marcado como fallido.

---

## Headers Enviados por QANode

```
Content-Type: application/json
Authorization: Bearer {token}  (si está configurado)
```

---

## Buenas Prácticas

1. **Nombres de tipo únicos**: Usa prefijos para evitar conflictos (ej: `mi-empresa-validador`)
2. **Logs descriptivos**: Incluye logs que ayuden a diagnosticar problemas
3. **Manejo de errores**: Siempre retorna `status: "failed"` con un `error.message` claro
4. **Inputs tipados**: Define un `inputSchema` completo para que QANode genere formularios adecuados
5. **Output Schema**: Define `outputSchema` para que QANode muestre los campos disponibles en el autocompletado
6. **Idempotencia**: Si es posible, implementa `/execute` de forma idempotente
7. **Timeout**: Usa `timeoutMs` en el manifiesto para nodos que necesiten más de 30 minutos. Implementa también timeouts internos para evitar que las operaciones queden colgadas
