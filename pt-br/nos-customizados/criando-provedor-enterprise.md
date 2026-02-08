# Criando um Provedor de Nós Customizados (Enterprise)

Este guia mostra como criar um provedor HTTP de nós customizados para o QANode Enterprise. Um provedor é um servidor HTTP que implementa o contrato de API e pode ser escrito em qualquer linguagem.

---

## Convenções

### Nomenclatura de Arquivos

Arquivos de definição de nó devem usar a extensão **`.node.`** seguida da extensão da linguagem:

| Linguagem | Extensão |
|-----------|----------|
| JavaScript | `.node.js`, `.node.mjs`, `.node.cjs` |
| Python | `.node.py` |
| Java | `.node.java` |
| C# | `.node.cs` |
| Go | `.node.go` |

Essa convenção distingue **arquivos de nó** (que exportam `manifest` e `execute`) de **arquivos auxiliares** (utils, helpers, etc.) que **não** devem usar `.node.` no nome.

### Organização: Uma Pasta por Nó

Cada nó customizado deve ter sua própria pasta contendo:

1. O arquivo de definição do nó (`.node.js` ou equivalente)
2. Arquivos auxiliares (helpers, utils — **sem** `.node.` no nome)
3. `package.json` ou equivalente para dependências próprias do nó

```
meu-provedor/
├── server.js                    # Servidor HTTP (auto-loader)
├── package.json                 # Dependências do servidor
├── validar-cpf/
│   ├── validar-cpf.node.js      # Definição do nó
│   ├── cpf-utils.js             # Auxiliar (NÃO usa .node.)
│   └── package.json             # Dependências do nó
├── validar-cnpj/
│   ├── validar-cnpj.node.js     # Definição do nó
│   ├── cnpj-utils.js            # Auxiliar
│   └── package.json
└── gerar-dados/
    ├── gerar-dados.node.js      # Definição do nó
    ├── generators.js            # Auxiliar
    └── package.json
```

> **Importante:** Somente arquivos com `.node.` no nome são reconhecidos como nós. Outros arquivos `.js` na mesma pasta são ignorados pelo loader e podem ser usados livremente como módulos auxiliares.

---

## Passo 1: Crie a Definição do Nó

Cada arquivo `.node.js` exporta `manifest` e `execute`:

```javascript
// validar-cpf/validar-cpf.node.js

import { validateCPF } from './cpf-utils.js';

export const manifest = {
  type: 'validar-cpf',
  name: 'Validar CPF',
  category: 'Validação',
  // timeoutMs: 300000, // opcional — padrão: 5 min
  inputSchema: {
    cpf: {
      type: 'string',
      required: true,
      description: 'CPF a ser validado (com ou sem formatação)'
    }
  },
  outputSchema: {
    valid: { type: 'boolean' },
    formatted: { type: 'string' },
    message: { type: 'string' }
  }
};

export async function execute({ inputs, runId, nodeId }) {
  const cpf = (inputs.cpf || '').replace(/\D/g, '');
  const valid = validateCPF(cpf);
  const formatted = cpf.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, '$1.$2.$3-$4');

  return {
    status: 'success',
    logs: [`CPF recebido: ${inputs.cpf}`, `Válido: ${valid}`],
    outputs: {
      valid,
      formatted,
      message: valid ? 'CPF válido' : 'CPF inválido'
    },
    artifacts: []
  };
}
```

Arquivo auxiliar (sem `.node.` no nome):

```javascript
// validar-cpf/cpf-utils.js

export function validateCPF(cpf) {
  if (cpf.length !== 11 || /^(\d)\1+$/.test(cpf)) return false;
  let sum = 0, rest;
  for (let i = 1; i <= 9; i++) sum += parseInt(cpf[i - 1]) * (11 - i);
  rest = (sum * 10) % 11;
  if (rest === 10 || rest === 11) rest = 0;
  if (rest !== parseInt(cpf[9])) return false;
  sum = 0;
  for (let i = 1; i <= 10; i++) sum += parseInt(cpf[i - 1]) * (12 - i);
  rest = (sum * 10) % 11;
  if (rest === 10 || rest === 11) rest = 0;
  return rest === parseInt(cpf[10]);
}
```

---

## Passo 2: Crie o Servidor (Auto-Loader)

O servidor HTTP escaneia automaticamente as pastas de nós, detectando arquivos `.node.js`:

```javascript
// server.js
import express from 'express';
import fs from 'fs';
import path from 'path';
import { pathToFileURL } from 'url';

const app = express();
app.use(express.json());

const nodes = [];
const executors = {};

// Auto-loader: busca arquivos *.node.js recursivamente
function findNodeFiles(dir) {
  const results = [];
  for (const entry of fs.readdirSync(dir, { withFileTypes: true })) {
    const fullPath = path.join(dir, entry.name);
    if (entry.isDirectory()) {
      results.push(...findNodeFiles(fullPath));
    } else if (entry.name.endsWith('.node.js') || entry.name.endsWith('.node.mjs')) {
      results.push(fullPath);
    }
  }
  return results;
}

// Carrega todos os nós
const nodeFiles = findNodeFiles(path.resolve('.'));
for (const filePath of nodeFiles) {
  try {
    const mod = await import(pathToFileURL(filePath).href);
    const manifest = mod.manifest || mod.default?.manifest;
    const execute = mod.execute || mod.default?.execute;
    if (manifest?.type && typeof execute === 'function') {
      nodes.push(manifest);
      executors[manifest.type] = execute;
      console.log(`Loaded: ${manifest.name} (${manifest.type}) from ${path.basename(filePath)}`);
    }
  } catch (err) {
    console.warn(`Failed to load ${filePath}: ${err.message}`);
  }
}

// GET /manifest — lista os nós disponíveis
app.get('/manifest', (req, res) => {
  res.json({ nodes });
});

// POST /execute — executa um nó
app.post('/execute', async (req, res) => {
  const { nodeType, inputs, runId, nodeId } = req.body;
  const executor = executors[nodeType];
  if (!executor) {
    return res.json({
      status: 'failed',
      logs: [],
      outputs: {},
      error: { message: `Nó não encontrado: ${nodeType}` }
    });
  }
  try {
    const result = await executor({ inputs, runId, nodeId });
    res.json(result);
  } catch (err) {
    res.json({
      status: 'failed',
      logs: [err.stack || err.message],
      outputs: {},
      error: { message: err.message }
    });
  }
});

// GET /health — verificação de saúde (opcional)
app.get('/health', (req, res) => {
  res.json({ ok: true, nodeCount: nodes.length });
});

app.listen(4010, () => {
  console.log(`Provedor rodando em http://localhost:4010 (${nodes.length} nós carregados)`);
});
```

---

## Passo 3: Teste Localmente

```bash
# Instale dependências de cada nó
cd validar-cpf && npm install && cd ..
cd validar-cnpj && npm install && cd ..

# Inicie o servidor
node server.js

# Teste o manifesto
curl http://localhost:4010/manifest

# Teste a execução
curl -X POST http://localhost:4010/execute \
  -H "Content-Type: application/json" \
  -d '{"nodeType": "validar-cpf", "inputs": {"cpf": "123.456.789-09"}}'
```

---

## Passo 4: Registre no QANode

1. No QANode, vá em **Configurações** → **Provedores Customizados**
2. Clique em **+ Adicionar Provedor**
3. Preencha:
   - **Nome**: `Validações`
   - **URL Base**: `http://localhost:4010`
   - **Token** (opcional): Token de autenticação Bearer
4. Clique em **Testar Conexão** → deve retornar sucesso com contagem de nós
5. Salve

---

## Passo 5: Use no Fluxo

1. Abra o editor de fluxos
2. Na paleta, localize a categoria **Validação** (ou **Custom Nodes**)
3. Arraste o nó **Validar CPF** para o canvas
4. Configure o campo **CPF**: `{{ steps.extract.outputs.extracts.cpf }}`
5. Conecte ao fluxo e execute

---

## Exemplo com Timeout Customizado

Para nós que executam operações longas, defina `timeoutMs` no manifesto:

```javascript
// processar-lote/processar-lote.node.js

export const manifest = {
  type: 'processar-lote',
  name: 'Processar Lote',
  category: 'Processamento',
  timeoutMs: 1800000, // 30 minutos
  inputSchema: {
    dados: { type: 'string', required: true, description: 'Caminho ou JSON dos dados' },
    formato: { type: 'string', enum: ['csv', 'json', 'xml'], default: 'json' }
  },
  outputSchema: {
    processados: { type: 'number' },
    erros: { type: 'number' },
    resultado: { type: 'array' }
  }
};

export async function execute({ inputs }) {
  // Operação longa...
  const resultado = await processarDados(inputs.dados, inputs.formato);

  return {
    status: 'success',
    logs: [`Processados: ${resultado.length} registros`],
    outputs: {
      processados: resultado.length,
      erros: 0,
      resultado
    },
    artifacts: []
  };
}
```

Se `timeoutMs` não for definido, o padrão é **5 minutos** (300.000ms).

---

## Exemplo em Outras Linguagens

A convenção `.node.` se aplica em qualquer linguagem. O importante é que o auto-loader do seu servidor saiba identificar os arquivos de nó:

### Python

```
meu-provedor-python/
├── server.py                    # Servidor FastAPI
├── requirements.txt
├── validar-cpf/
│   ├── validar_cpf.node.py      # Definição do nó
│   ├── utils.py                 # Auxiliar (sem .node.)
│   └── requirements.txt
└── gerar-dados/
    ├── gerar_dados.node.py
    └── requirements.txt
```

```python
# validar-cpf/validar_cpf.node.py

from .utils import validate_cpf_digits

manifest = {
    "type": "validar-cpf",
    "name": "Validar CPF",
    "category": "Validação",
    "inputSchema": {
        "cpf": {"type": "string", "required": True}
    },
    "outputSchema": {
        "valid": {"type": "boolean"}
    }
}

def execute(inputs, run_id=None, node_id=None):
    cpf = inputs.get("cpf", "").replace(".", "").replace("-", "")
    valid = validate_cpf_digits(cpf)
    return {
        "status": "success",
        "logs": [f"CPF: {cpf}, Válido: {valid}"],
        "outputs": {"valid": valid},
        "artifacts": []
    }
```

### Java / C# / Go

Siga a mesma estrutura: cada nó em sua pasta com o arquivo `.node.{ext}` e os auxiliares sem `.node.` no nome. O servidor HTTP da linguagem escolhida implementa o auto-loader que busca os arquivos `.node.*`.

---

## Boas Práticas

1. **Uma pasta por nó** — mantém dependências e auxiliares isolados
2. **Nomenclatura `.node.`** — apenas para arquivos de definição de nó, nunca para auxiliares
3. **`package.json` por nó** — cada nó gerencia suas próprias dependências
4. **Logs descritivos** — inclua logs que ajudem a diagnosticar problemas
5. **`timeoutMs` no manifesto** — defina para nós que excedem 5 minutos
6. **Output Schema** — defina `outputSchema` para que o QANode mostre os campos no autocomplete
7. **Docker** — considere containerizar o provedor para facilitar o deploy

---

## Próximos Passos

- [Contrato da API](contrato-api.md) — Especificação completa dos endpoints
- [Exemplos](exemplos-enterprise.md) — Mais exemplos em várias linguagens
- [Nós Locais Desktop](nos-locais-desktop.md) — Alternativa sem servidor HTTP
