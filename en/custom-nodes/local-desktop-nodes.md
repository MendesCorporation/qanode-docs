# Local Nodes in Desktop Version

In QANode Desktop, you can create custom nodes locally without running a separate HTTP server. The local runtime detects node files and exposes them through the internal local provider.

---

## How It Works (full cycle)

1. Local runtime scans `%USERPROFILE%\Documents\QANode\custom nodes\`
2. It loads `*.node.js`, `*.node.mjs`, `*.node.cjs` files
3. It validates required exports (`manifest`, `execute`, `manifest.type`, `manifest.name`)
4. It publishes valid nodes in local `/manifest`
5. Worker fetches `/manifest` and validates `required` inputs from `inputSchema`
6. Worker calls `/execute` with `nodeType`, `inputs`, `runId`, `nodeId`
7. Node result returns to flow with `logs`, `outputs`, and `artifacts`

---

## Folder and Recommended Structure

```text
%USERPROFILE%\Documents\QANode\custom nodes\
```

```text
custom nodes/
├── my-node/
│   ├── my-node.node.js
│   ├── helpers.js
│   └── package.json
└── another-node/
    ├── another-node.node.mjs
    └── package.json
```

> Only files containing `.node.` in the name are loaded as nodes.

---

## Module Rules by Extension

- `.node.js`: use CommonJS (`module.exports`) by default
- `.node.mjs`: use ESM (`export const`, `export async function`)
- `.node.cjs`: use CommonJS

Common error:

- `Unexpected token 'export'` in `.node.js`: file is ESM without proper module setup.

---

## Node File Contract

Each node file must export:

1. `manifest`
2. `execute`

If any required piece is missing, the file is ignored.

### Manifest: required vs optional fields

| Field | Type | Required | QANode usage |
|---|---|---|---|
| `type` | `string` | Yes | Unique node identifier |
| `name` | `string` | Yes | Display name in palette |
| `category` | `string` | No | Palette group |
| `timeoutMs` | `number` | No | Node timeout in ms; invalid/missing uses provider default |
| `inputSchema` | `object` | No | Input fields in properties panel |
| `outputSchema` | `object` | No | Output fields for autocomplete |
| `...extra` | `any` | No | Extra metadata, no default core behavior |

### inputSchema: field format

| Property | Type | Required | Notes |
|---|---|---|---|
| `type` | `string` | Recommended | `string`, `number`, `boolean`, `object`, `array`, `any` |
| `required` | `boolean` | No | If true, worker blocks execution when missing |
| `default` | `any` | No | Default value |
| `description` | `string` | No | UI help text |
| `enum` | `array` | No | Allowed values |
| `items` | `object` | No | Item schema for arrays |

### outputSchema: field format

| Property | Type | Required |
|---|---|---|
| `type` | `string` | Recommended |

---

## Execute: input parameters

Your `execute` function receives:

| Parameter | Type | Required | Source |
|---|---|---|---|
| `nodeType` | `string` | No | Requested node type |
| `inputs` | `object` | Yes (practical) | Node data from flow |
| `runId` | `string` | No | Execution ID |
| `nodeId` | `string` | No | Node ID in flow |

Payload example:

```json
{
  "nodeType": "my-node",
  "inputs": {
    "field1": "value",
    "field2": 10
  },
  "runId": "run_123",
  "nodeId": "node_abc"
}
```

---

## Execute: output contract (required vs recommended)

### Recommended full response

| Field | Type | Required | Notes |
|---|---|---|---|
| `status` | `string` | Required | `success` or `failed` |
| `logs` | `string[]` | Recommended | Diagnostic logs |
| `outputs` | `object` | Recommended | Node output data |
| `artifacts` | `array` | Recommended | Files (screenshot/pdf/file/etc.) |
| `error` | `object` | When failed | Example: `{ "message": "..." }` |

### Actual local runtime behavior

- If response includes `status`, runtime uses it directly.
- If response has no `status`, runtime may wrap as `status: "success"` and treat object as `outputs`.
- If `execute` throws, runtime returns `status: "failed"` with `error.message`.

---

## Correct `.node.js` Example (CommonJS)

```javascript
// my-node/my-node.node.js
module.exports = {
  manifest: {
    type: "my-node",
    name: "My Node",
    category: "Custom",
    timeoutMs: 600000,
    inputSchema: {
      text: { type: "string", required: true, description: "Input text" },
      repeat: { type: "number", default: 1 }
    },
    outputSchema: {
      result: { type: "string" },
      processedAt: { type: "string" }
    }
  },
  async execute({ inputs = {}, runId, nodeId }) {
    const text = String(inputs.text || "");
    const repeat = Number(inputs.repeat || 1);
    const result = text.repeat(Math.max(1, repeat));

    return {
      status: "success",
      logs: [
        `runId=${runId || "n/a"} nodeId=${nodeId || "n/a"}`,
        `text=${text}`,
        `repeat=${repeat}`
      ],
      outputs: {
        result,
        processedAt: new Date().toISOString()
      },
      artifacts: []
    };
  }
};
```

---

## Correct `.node.mjs` Example (ESM)

```javascript
// my-node-mjs/my-node-mjs.node.mjs
export const manifest = {
  type: "my-node-mjs",
  name: "My Node MJS",
  category: "Custom",
  inputSchema: {
    text: { type: "string", required: true }
  },
  outputSchema: {
    result: { type: "string" }
  }
};

export async function execute({ inputs = {} }) {
  const result = String(inputs.text || "").toUpperCase();
  return {
    status: "success",
    logs: [`result=${result}`],
    outputs: { result },
    artifacts: []
  };
}
```

---

## Artifacts Format

Each `artifacts` item can be:

```json
{
  "type": "screenshot",
  "name": "evidence.png",
  "base64": "iVBORw0KGgoAAA..."
}
```

Common fields:

- `type`: `screenshot`, `pdf`, `video`, `file`
- `name`: file name
- `base64`: base64 content
- `path`: optional when file already exists elsewhere

---

## How to Verify Import/Load Errors

### 1) Local provider test

1. Open **Settings > Providers**
2. Test `Desktop Local JS Provider`
3. If node is missing, it did not load

### 2) Detailed desktop log

File:

```text
%APPDATA%\@qanode\desktop\desktop-main.log
```

Look for:

- `Failed loading "...file...": ...`
- `Cannot find package '...'`
- `Unexpected token 'export'`

### 3) PowerShell verification

```powershell
$providers = Invoke-RestMethod -Uri "http://127.0.0.1:30000/api/providers"
$p = $providers | Where-Object { $_.name -eq "Desktop Local JS Provider" }
Invoke-RestMethod -Uri ($p.baseUrl + "/manifest")
Invoke-RestMethod -Method Post -Uri ($p.baseUrl + "/reload")
Invoke-RestMethod -Uri ($p.baseUrl + "/health")
```

### 4) Direct execution test

```powershell
Invoke-RestMethod -Method Post `
  -Uri ($p.baseUrl + "/execute") `
  -ContentType "application/json" `
  -Body '{"nodeType":"my-node","inputs":{"text":"ok"}}'
```

---

## Most Common Errors

- `Cannot find package 'x' imported from ...`
  - Dependency missing in node folder

- `Unexpected token 'export'` in `.node.js`
  - CJS/ESM mismatch for extension

- Node not listed in `/manifest`
  - Missing `manifest`
  - Missing/non-function `execute`
  - Missing/invalid `manifest.type`
  - Missing/invalid `manifest.name`

---

## Final Quality Checklist

- Correct extension (`.node.js`, `.node.mjs`, `.node.cjs`)
- Unique `manifest.type`
- Human-friendly `manifest.name`
- `inputSchema` and `outputSchema` defined
- `execute` includes error handling
- Response includes `status`, `logs`, `outputs`, `artifacts`
- Dependencies installed in node folder
