# Nos Locais na Versao Desktop

Na versao desktop do QANode, voce pode criar nos customizados localmente, sem subir um servidor HTTP separado. O runtime local detecta os arquivos e publica os nos como provider interno.

---

## Como Funciona (ciclo completo)

1. O runtime local varre `%USERPROFILE%\Documents\QANode\custom nodes\`
2. Carrega arquivos `*.node.js`, `*.node.mjs`, `*.node.cjs`
3. Valida exports obrigatorios (`manifest`, `execute`, `manifest.type`, `manifest.name`)
4. Publica os nos no endpoint local `/manifest`
5. O worker busca `/manifest`, valida campos `required` do `inputSchema`
6. O worker chama `/execute` com `nodeType`, `inputs`, `runId`, `nodeId`
7. O resultado do node volta para o fluxo com `logs`, `outputs` e `artifacts`

---

## Pasta e Estrutura Recomendada

```text
%USERPROFILE%\Documents\QANode\custom nodes\
```

```text
custom nodes/
├── meu-no/
│   ├── meu-no.node.js
│   ├── helpers.js
│   └── package.json
└── outro-no/
    ├── outro-no.node.mjs
    └── package.json
```

> Apenas arquivos com `.node.` no nome sao carregados como no.

---

## Regras de Modulo por Extensao

- `.node.js`: use CommonJS (`module.exports`) por padrao
- `.node.mjs`: use ESM (`export const`, `export async function`)
- `.node.cjs`: use CommonJS

Erro comum:

- `Unexpected token 'export'` em `.node.js`: arquivo esta em ESM sem `type: "module"`.

---

## Contrato do Arquivo do No

Cada arquivo precisa exportar:

1. `manifest`
2. `execute`

Se faltar algum obrigatorio, o arquivo e ignorado no carregamento.

### Manifest: campos obrigatorios e opcionais

| Campo | Tipo | Obrigatorio | Uso no QANode |
|---|---|---|---|
| `type` | `string` | Sim | Identificador unico do no |
| `name` | `string` | Sim | Nome exibido na paleta |
| `category` | `string` | Nao | Grupo na paleta |
| `timeoutMs` | `number` | Nao | Timeout desse no (ms). Se ausente/invalido, usa padrao global do provider |
| `inputSchema` | `object` | Nao | Define campos de entrada no painel de propriedades |
| `outputSchema` | `object` | Nao | Define campos de saida para autocomplete |
| `...extra` | `any` | Nao | Metadados extras permitidos, sem comportamento padrao no core |

### inputSchema: formato de cada campo

| Propriedade | Tipo | Obrigatorio | Observacao |
|---|---|---|---|
| `type` | `string` | Recomendado | `string`, `number`, `boolean`, `object`, `array`, `any` |
| `required` | `boolean` | Nao | Se `true`, worker bloqueia execucao sem valor |
| `default` | `any` | Nao | Valor inicial |
| `description` | `string` | Nao | Ajuda na UI |
| `enum` | `array` | Nao | Lista de valores permitidos |
| `items` | `object` | Nao | Schema de item para arrays |

### outputSchema: formato

| Propriedade | Tipo | Obrigatorio |
|---|---|---|
| `type` | `string` | Recomendado |

---

## Execute: parametros de entrada

Sua funcao `execute` recebe um objeto com:

| Parametro | Tipo | Obrigatorio | Origem |
|---|---|---|---|
| `nodeType` | `string` | Nao | Tipo do no solicitado |
| `inputs` | `object` | Sim (pratico) | Dados do no no fluxo |
| `runId` | `string` | Nao | ID da execucao |
| `nodeId` | `string` | Nao | ID do no no fluxo |

Exemplo de payload recebido:

```json
{
  "nodeType": "meu-no",
  "inputs": {
    "campo1": "valor",
    "campo2": 10
  },
  "runId": "run_123",
  "nodeId": "node_abc"
}
```

---

## Execute: retorno (obrigatorio x recomendado)

### Retorno completo (recomendado)

| Campo | Tipo | Obrigatorio | Observacao |
|---|---|---|---|
| `status` | `string` | Obrigatório | `success` ou `failed` |
| `logs` | `string[]` | Recomendado | Logs de diagnostico |
| `outputs` | `object` | Recomendado | Dados de saida do no |
| `artifacts` | `array` | Recomendado | Arquivos (screenshot/pdf/file etc.) |
| `error` | `object` | Quando `failed` | Ex: `{ "message": "..." }` |

### Comportamento real do runtime local

- Se voce retornar objeto **com** `status`, ele e usado como esta
- Se voce retornar objeto **sem** `status`, runtime envolve como `status: "success"` e usa o objeto como `outputs`
- Se voce lancar erro (`throw`), runtime retorna `status: "failed"` com `error.message`

---

## Exemplo Correto `.node.js` (CommonJS)

```javascript
// meu-no/meu-no.node.js
module.exports = {
  manifest: {
    type: "meu-no",
    name: "Meu No",
    category: "Custom",
    timeoutMs: 600000,
    inputSchema: {
      texto: { type: "string", required: true, description: "Texto de entrada" },
      repetir: { type: "number", default: 1 }
    },
    outputSchema: {
      resultado: { type: "string" },
      processedAt: { type: "string" }
    }
  },
  async execute({ inputs = {}, runId, nodeId }) {
    const texto = String(inputs.texto || "");
    const repetir = Number(inputs.repetir || 1);
    const resultado = texto.repeat(Math.max(1, repetir));

    return {
      status: "success",
      logs: [
        `runId=${runId || "n/a"} nodeId=${nodeId || "n/a"}`,
        `texto=${texto}`,
        `repetir=${repetir}`
      ],
      outputs: {
        resultado,
        processedAt: new Date().toISOString()
      },
      artifacts: []
    };
  }
};
```

---

## Exemplo Correto `.node.mjs` (ESM)

```javascript
// meu-no-mjs/meu-no-mjs.node.mjs
export const manifest = {
  type: "meu-no-mjs",
  name: "Meu No MJS",
  category: "Custom",
  inputSchema: {
    texto: { type: "string", required: true }
  },
  outputSchema: {
    resultado: { type: "string" }
  }
};

export async function execute({ inputs = {} }) {
  const resultado = String(inputs.texto || "").toUpperCase();
  return {
    status: "success",
    logs: [`resultado=${resultado}`],
    outputs: { resultado },
    artifacts: []
  };
}
```

---

## Artifacts (formato)

Cada item em `artifacts` pode ser:

```json
{
  "type": "screenshot",
  "name": "evidencia.png",
  "base64": "iVBORw0KGgoAAA..."
}
```

Campos comuns:

- `type`: `screenshot`, `pdf`, `video`, `file`
- `name`: nome do arquivo
- `base64`: conteudo em base64
- `path`: opcional quando ja existe caminho externo

---

## Como Verificar Erros de Importacao

### Log de erro detalhado

Arquivo:

```text
%APPDATA%\@qanode\desktop\desktop-main.log
```

Procure por:

- `Failed loading "...arquivo...": ...`
- `Cannot find package '...'`
- `Unexpected token 'export'`

### 3) Verificacao via PowerShell

```powershell
$providers = Invoke-RestMethod -Uri "http://127.0.0.1:30000/api/providers"
$p = $providers | Where-Object { $_.name -eq "Desktop Local JS Provider" }
Invoke-RestMethod -Uri ($p.baseUrl + "/manifest")
Invoke-RestMethod -Method Post -Uri ($p.baseUrl + "/reload")
Invoke-RestMethod -Uri ($p.baseUrl + "/health")
```

### 4) Teste de execucao direta

```powershell
Invoke-RestMethod -Method Post `
  -Uri ($p.baseUrl + "/execute") `
  -ContentType "application/json" `
  -Body '{"nodeType":"meu-no","inputs":{"texto":"ok"}}'
```

---

## Erros Mais Comuns e Causa

- `Cannot find package 'x' imported from ...`
  - Dependencia nao instalada na pasta do no

- `Unexpected token 'export'` em `.node.js`
  - Mistura CJS/ESM na extensao errada

- No nao aparece no `/manifest`
  - `manifest` ausente
  - `execute` ausente/nao-funcao
  - `manifest.type` ausente/invalido
  - `manifest.name` ausente/invalido

---

## Checklist Final de Qualidade

- Extensao correta (`.node.js`, `.node.mjs`, `.node.cjs`)
- `manifest.type` unico
- `manifest.name` legivel
- `inputSchema` e `outputSchema` definidos
- `execute` com tratamento de erro
- Retorno com `status`, `logs`, `outputs`, `artifacts`
- Dependencias instaladas localmente no no
