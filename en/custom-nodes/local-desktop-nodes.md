# Local Nodes in the Desktop Version

In the desktop version of QANode, you can create custom nodes **locally** as JavaScript files, without needing a separate HTTP server. QANode automatically detects the files and makes the nodes available in the palette.

---

## How It Works

The desktop version includes a **built-in local provider** that:

1. Monitors a folder on your computer
2. Detects `.node.js`, `.node.mjs`, or `.node.cjs` files
3. Automatically loads the nodes defined in those files
4. Reloads when changes are detected (**hot-reload**)

---

## Custom Nodes Folder

Local nodes are stored in the following folder:

```
%USERPROFILE%\Documents\QANode\custom nodes\
```

On Windows, typically:
```
C:\Users\YourUser\Documents\QANode\custom nodes\
```

The recommended organization is **one folder per node**, each containing the `.node.js` file, auxiliary files, and a `package.json` for dependencies:

```
custom nodes/
├── validar-cpf/
│   ├── validar-cpf.node.js    # Node definition
│   ├── cpf-utils.js           # Auxiliary (does NOT use .node.)
│   └── package.json           # Node dependencies
├── formatar-data/
│   ├── formatar-data.node.js
│   └── package.json
└── sample-hash-node/          # Sample node (created automatically)
    ├── sample-hash.node.js
    └── package.json
```

> **Important:** Only files with `.node.` in the name are recognized as nodes. Other `.js` files in the same folder are ignored and can be used as auxiliary modules.

---

## File Format

Each `.node.js` file must export two elements:

1. **`manifest`** — Node definition (type, name, inputs, outputs)
2. **`execute`** — Execution function

### Basic Structure

```javascript
// meu-no/meu-no.node.js

export const manifest = {
  type: 'meu-no-customizado',
  name: 'Meu Nó Customizado',
  category: 'Meus Nós',
  timeoutMs: 600000, // 10 minutes (optional, default: 30 minutes)
  inputSchema: {
    campo1: { type: 'string', required: true, description: 'Descrição do campo' },
    campo2: { type: 'number', default: 10 }
  },
  outputSchema: {
    resultado: { type: 'string' },
    processedAt: { type: 'string' }
  }
};

export async function execute({ inputs, runId, nodeId }) {
  const resultado = `Processado: ${inputs.campo1} (x${inputs.campo2})`;

  return {
    status: 'success',
    logs: [
      `Recebido: campo1="${inputs.campo1}", campo2=${inputs.campo2}`,
      `Resultado: ${resultado}`
    ],
    outputs: {
      resultado,
      processedAt: new Date().toISOString()
    },
    artifacts: []
  };
}
```

---

## Complete Example: Hash Generator

This is the sample node that QANode creates automatically on first run:

```javascript
// sample-hash-node/sample-hash.node.js

import crypto from 'crypto';

export const manifest = {
  type: 'sample-hash',
  name: 'Sample Hash Generator',
  category: 'Custom Nodes',
  inputSchema: {
    text: {
      type: 'string',
      required: true,
      description: 'Text to hash'
    },
    algorithm: {
      type: 'string',
      enum: ['md5', 'sha256', 'sha512'],
      default: 'sha256',
      description: 'Hash algorithm'
    }
  },
  outputSchema: {
    hash: { type: 'string' },
    algorithm: { type: 'string' },
    timestamp: { type: 'string' }
  }
};

export async function execute({ inputs }) {
  const { text, algorithm = 'sha256' } = inputs;
  const hash = crypto.createHash(algorithm).update(text).digest('hex');

  return {
    status: 'success',
    logs: [
      `Input: "${text.substring(0, 50)}"`,
      `Algorithm: ${algorithm}`,
      `Hash: ${hash}`
    ],
    outputs: {
      hash,
      algorithm,
      timestamp: new Date().toISOString()
    },
    artifacts: []
  };
}
```

---

## Hot-Reload

QANode monitors the custom nodes folder and **automatically reloads** when it detects changes:

- **Check interval**: 1.5 seconds
- **Detection**: Based on file modification date and size
- **Reload**: Automatic, no need to restart QANode

### Development Workflow

1. Create or edit a `.node.js` file in the nodes folder
2. Save the file
3. In ~1.5 seconds, QANode detects the change
4. The updated node appears in the editor palette

> **Tip:** Open the QANode console (DevTools) to see reload logs:
> ```
> [CustomProvider] Reloaded: meu-no.node.js (2 nodes)
> ```

---

## Using NPM Dependencies

Local nodes can use native Node.js modules (`crypto`, `path`, `fs`, etc.) directly. For external npm dependencies, each node has its own `package.json`:

```
custom nodes/
└── formatar-data/
    ├── formatar-data.node.js   # Node definition
    ├── date-helpers.js         # Auxiliary (without .node.)
    └── package.json            # Node dependencies
```

```bash
cd "%USERPROFILE%\Documents\QANode\custom nodes\formatar-data"
npm init -y
npm install dayjs
```

```javascript
// formatar-data/formatar-data.node.js
import dayjs from 'dayjs';

export const manifest = {
  type: 'formatar-data',
  name: 'Formatar Data',
  inputSchema: {
    data: { type: 'string', required: true },
    formato: { type: 'string', default: 'DD/MM/YYYY' }
  },
  outputSchema: {
    formatted: { type: 'string' }
  }
};

export async function execute({ inputs }) {
  const formatted = dayjs(inputs.data).format(inputs.formato);
  return {
    status: 'success',
    logs: [`${inputs.data} → ${formatted}`],
    outputs: { formatted },
    artifacts: []
  };
}
```

---

## Differences: Local Nodes vs HTTP Provider

| Aspect | Local Nodes | HTTP Provider |
|--------|-------------|---------------|
| **Setup** | Create a file in the folder | Create and run a server |
| **Language** | JavaScript only | Any language |
| **Hot-Reload** | Automatic | Automatic |
| **Availability** | Desktop version only | Enterprise only |
| **Deployment** | Local only | Can be remote |
| **Dependencies** | npm (local) | Any package manager |

---

## Tips

- Use the `*.node.js` convention — QANode only detects files with this extension. Auxiliary files **must not** use `.node.` in the name
- **One folder per node** — each node with its `.node.js`, auxiliaries, and `package.json`
- The `category` field in the manifest controls which group the node appears in on the palette
- Use `console.log()` in execute for debugging — the output appears in the Electron console
- Start with the sample node (sample-hash) as a template
- For long-running operations, define `timeoutMs` in the manifest (default: 30 min, e.g., `timeoutMs: 1800000` for 30 min)
