# Contrato da API de Provedores

Esta página documenta a especificação completa da API HTTP que um provedor de nós customizados deve implementar.

---

## Endpoints Obrigatórios

### GET /manifest

Retorna a lista de nós disponíveis no provedor.

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

> No exemplo acima, `timeoutMs: 600000` define um timeout de 10 minutos para este nó. Se omitido, o padrão é 30 minutos.

#### Campos do Nó

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `type` | `string` | Sim | Identificador único do nó |
| `name` | `string` | Sim | Nome exibido na paleta |
| `category` | `string` | Não | Categoria na paleta (padrão: "Custom Nodes") |
| `inputSchema` | `object` | Não | Schema dos campos de entrada |
| `outputSchema` | `object` | Não | Schema dos dados de saída |
| `timeoutMs` | `number` | Não | Timeout de execução em milissegundos (padrão: 18000000 = 30 minutos) |

#### Input Schema

Cada campo do `inputSchema` pode ter:

| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `type` | `string` | Tipo: `string`, `number`, `boolean`, `object`, `array`, `any` |
| `required` | `boolean` | Se o campo é obrigatório |
| `default` | `any` | Valor padrão |
| `description` | `string` | Descrição exibida como tooltip |
| `enum` | `string[]` | Lista de valores permitidos (renderiza como dropdown) |
| `items` | `object` | Schema dos itens (para type: `array`) |

#### Output Schema

O `outputSchema` define os dados que o nó produz após a execução. Cada campo tem:

| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `type` | `string` | Tipo do dado |

Os outputs ficam acessíveis via expressões: `{{ steps.meuNo.outputs.resultado }}`

---

### POST /execute

Executa um nó com os dados fornecidos.

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

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `nodeType` | `string` | Tipo do nó a executar |
| `inputs` | `object` | Valores dos campos de entrada |
| `runId` | `string` | ID da execução atual |
| `nodeId` | `string` | ID do nó no fluxo |

**Response (Sucesso):**

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

**Response (Falha):**

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

#### Campos da Response

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `status` | `string` | Sim | `"success"` ou `"failed"` |
| `logs` | `string[]` | Não | Mensagens de log (exibidas nos detalhes) |
| `outputs` | `object` | Não | Dados produzidos pelo nó |
| `error` | `object` | Não | Detalhes do erro (quando `failed`) |
| `error.message` | `string` | — | Mensagem de erro legível |
| `artifacts` | `array` | Não | Arquivos gerados (screenshots, etc.) |

---

## Artefatos

O campo `artifacts` permite que o nó retorne arquivos como screenshots, PDFs ou outros documentos. Cada artefato pode ser enviado como **base64** inline:

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

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `type` | `string` | Tipo: `screenshot`, `pdf`, `video`, `file` |
| `name` | `string` | Nome do arquivo |
| `base64` | `string` | Conteúdo codificado em base64 |

Os artefatos são automaticamente salvos no storage do QANode e ficam disponíveis nos resultados da execução.

---

## Endpoint Opcional

### GET /health

Verifica se o provedor está funcionando. Usado no teste de conexão.

**Response:**
```json
{
  "ok": true,
  "nodeCount": 5
}
```

---

## Autenticação

Se o provedor requerer autenticação, configure o **Token** ao registrar o provedor no QANode. O token será enviado como header `Authorization: Bearer {token}` em todas as requisições.

---

## Validação de Campos Obrigatórios

O QANode valida campos marcados como `required: true` no `inputSchema` **antes** de chamar `/execute`. Se um campo obrigatório estiver vazio, a execução falha sem chamar o provedor.

---

## Timeouts

| Endpoint | Timeout |
|----------|---------|
| `GET /manifest` | 5 segundos |
| `POST /execute` | 30 minutos (padrão) — configurável via `timeoutMs` no manifesto |

O timeout do `/execute` pode ser customizado **por nó** através do campo `timeoutMs` no manifesto. Isso é útil para nós que executam operações longas como processamento de dados, scraping ou integração com sistemas externos.

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

No exemplo acima, o nó terá um timeout de 30 minutos (1.800.000ms). Se `timeoutMs` não for definido, o padrão é **30 minutos** (300.000ms).

Se o provedor não responder dentro do timeout, a requisição é abortada e o nó é marcado como falha.

### Timeout Global de Fluxo

Independente do timeout de cada nó, o QANode impõe um **timeout global de 24 horas** por execução de fluxo. Se um fluxo (ou suíte) exceder 24 horas de execução total, ele será automaticamente interrompido e marcado como falha.

---

## Headers Enviados pelo QANode

```
Content-Type: application/json
Authorization: Bearer {token}  (se configurado)
```

---

## Boas Práticas

1. **Nomes de tipo únicos**: Use prefixos para evitar conflitos (ex: `minha-empresa-validador`)
2. **Logs descritivos**: Inclua logs que ajudem a diagnosticar problemas
3. **Tratamento de erros**: Sempre retorne `status: "failed"` com `error.message` claro
4. **Inputs tipados**: Defina `inputSchema` completo para que o QANode gere formulários adequados
5. **Output Schema**: Defina `outputSchema` para que o QANode mostre os campos disponíveis no autocomplete
6. **Idempotência**: Se possível, implemente o `/execute` de forma idempotente
7. **Timeout**: Use `timeoutMs` no manifesto para nós que precisam de mais de 30 minutos. Implemente também timeouts internos para evitar que operações fiquem penduradas
