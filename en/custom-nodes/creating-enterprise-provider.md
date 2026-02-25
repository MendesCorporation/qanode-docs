# Creating a Custom Node Provider (Enterprise)

This guide shows how to create an HTTP provider of custom nodes for QANode Enterprise. A provider is an HTTP server that implements the API contract and can be written in any language.

---

## Conventions

### File Naming

Node definition files must use the **`.node.`** extension followed by the language extension:

| Language | Extension |
|----------|-----------|
| JavaScript | `.node.js`, `.node.mjs`, `.node.cjs` |
| Python | `.node.py` |
| Java | `.node.java` |
| C# | `.node.cs` |
| Go | `.node.go` |

This convention distinguishes **node files** (which export `manifest` and `execute`) from **auxiliary files** (utils, helpers, etc.) that **should not** use `.node.` in their name.

### Organization: One Folder Per Node

Each custom node should have its own folder containing:

1. The node definition file (`.node.js` or equivalent)
2. Auxiliary files (helpers, utils — **without** `.node.` in the name)
3. `package.json` or equivalent for the node's own dependencies

```
meu-provedor/
├── server.js                    # HTTP server (auto-loader)
├── package.json                 # Server dependencies
├── validar-cpf/
│   ├── validar-cpf.node.js      # Node definition
│   ├── cpf-utils.js             # Auxiliary (does NOT use .node.)
│   └── package.json             # Node dependencies
├── validar-cnpj/
│   ├── validar-cnpj.node.js     # Node definition
│   ├── cnpj-utils.js            # Auxiliary
│   └── package.json
└── gerar-dados/
    ├── gerar-dados.node.js      # Node definition
    ├── generators.js            # Auxiliary
    └── package.json
```

> **Important:** Only files with `.node.` in the name are recognized as nodes. Other `.js` files in the same folder are ignored by the loader and can be freely used as auxiliary modules.

---

## Step 1: Create the Node Definition

Each `.node.js` file exports `manifest` and `execute`:

```javascript
// validar-cpf/validar-cpf.node.js

import { validateCPF } from './cpf-utils.js';

export const manifest = {
  type: 'validar-cpf',
  name: 'Validar CPF',
  category: 'Validação',
  // timeoutMs: 300000, // optional — default: 30 min
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

Auxiliary file (without `.node.` in the name):

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

## Step 2: Create the Server (Auto-Loader)

The HTTP server automatically scans node folders, detecting `.node.js` files:

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

// Auto-loader: finds *.node.js files recursively
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

// Loads all nodes
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

// GET /manifest — lists available nodes
app.get('/manifest', (req, res) => {
  res.json({ nodes });
});

// POST /execute — executes a node
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

// GET /health — health check (optional)
app.get('/health', (req, res) => {
  res.json({ ok: true, nodeCount: nodes.length });
});

app.listen(4010, () => {
  console.log(`Provedor rodando em http://localhost:4010 (${nodes.length} nós carregados)`);
});
```

---

## Step 3: Test Locally

```bash
# Install each node's dependencies
cd validar-cpf && npm install && cd ..
cd validar-cnpj && npm install && cd ..

# Start the server
node server.js

# Test the manifest
curl http://localhost:4010/manifest

# Test execution
curl -X POST http://localhost:4010/execute \
  -H "Content-Type: application/json" \
  -d '{"nodeType": "validar-cpf", "inputs": {"cpf": "123.456.789-09"}}'
```

---

## Step 4: Register in QANode

1. In QANode, go to **Settings** → **Custom Providers**
2. Click **+ Add Provider**
3. Fill in:
   - **Name**: `Validações`
   - **Base URL**: `http://localhost:4010`
   - **Token** (optional): Bearer authentication token
4. Click **Test Connection** → should return success with node count
5. Save

---

## Step 5: Use in a Flow

1. Open the flow editor
2. In the palette, locate the **Validação** (or **Custom Nodes**) category
3. Drag the **Validar CPF** node onto the canvas
4. Configure the **CPF** field: `{{ steps.extract.outputs.extracts.cpf }}`
5. Connect it to the flow and execute

---

## Example with Custom Timeout

For nodes that perform long-running operations, define `timeoutMs` in the manifest:

```javascript
// processar-lote/processar-lote.node.js

export const manifest = {
  type: 'processar-lote',
  name: 'Processar Lote',
  category: 'Processamento',
  timeoutMs: 1800000, // 30 minutes
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
  // Long-running operation...
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

If `timeoutMs` is not defined, the default is **30 minutes** (300,000ms).

---

## Example in Other Languages

The `.node.` convention applies in any language. The key is that your server's auto-loader knows how to identify node files:

### Python

```
meu-provedor-python/
├── server.py                    # FastAPI server
├── requirements.txt
├── validar-cpf/
│   ├── validar_cpf.node.py      # Node definition
│   ├── utils.py                 # Auxiliary (without .node.)
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

Follow the same structure: each node in its own folder with the `.node.{ext}` file and auxiliaries without `.node.` in the name. The HTTP server of your chosen language implements the auto-loader that finds `.node.*` files.

---

## Best Practices

1. **One folder per node** — keeps dependencies and auxiliaries isolated
2. **`.node.` naming** — only for node definition files, never for auxiliaries
3. **`package.json` per node** — each node manages its own dependencies
4. **Descriptive logs** — include logs that help diagnose issues
5. **`timeoutMs` in the manifest** — set for nodes that exceed 30 minutes
6. **Output Schema** — define `outputSchema` so QANode shows the fields in autocomplete
7. **Docker** — consider containerizing the provider to simplify deployment

---

## Next Steps

- [API Contract](api-contract.md) — Complete endpoint specification
- [Examples](enterprise-examples.md) — More examples in various languages
- [Local Desktop Nodes](local-desktop-nodes.md) — Alternative without an HTTP server
