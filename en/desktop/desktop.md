# Desktop Version

QANode Desktop is a **Community Edition** application with no restrictions on core flow features.

---

## Desktop Advantages

| Feature | Description |
|---|---|
| **Simple Installation** | Single installer, no external dependencies |
| **Embedded Database** | Integrated PostgreSQL, no manual setup |
| **Local Nodes** | Build custom nodes as local files (hot-reload) |
| **All-in-One** | Frontend, API, Worker, and database in one app |
| **Portable** | Works offline, no server required |

---

## Installation

See [Installation](../getting-started/installation.md) for full steps.

### Minimum Requirements

- **Windows** 10/11 (64-bit)
- **RAM**: 4 GB minimum (8 GB recommended)
- **Disk**: 500 MB for installation + data space

---

## First Run

On first startup, QANode:

1. Shows splash progress
2. Initializes embedded PostgreSQL
3. Runs migrations (creates tables)
4. Starts API server
5. Opens main UI

> First initialization may take a few extra seconds while database setup completes.

---

## Data Structure

QANode stores data under user folders:

```text
%APPDATA%\QANode\
├── pg-data/              # Embedded PostgreSQL data
├── storage/              # Artifacts (screenshots, PDFs)
└── logs/                 # App logs
```

```text
%USERPROFILE%\Documents\QANode\
└── custom nodes/         # Local custom nodes
    └── sample-hash-node/ # Sample node
```

> Important: do not manually modify files under `pg-data/`.

---

## Local Custom Nodes

Desktop supports local nodes: JavaScript files automatically detected and loaded.

### Nodes Folder

```text
%USERPROFILE%\Documents\QANode\custom nodes\
```

### Creating a Local Node

1. Create a file with extension `.node.js`, `.node.mjs`, or `.node.cjs`
2. Export `manifest` and `execute`
3. QANode auto-detects changes in ~1.5 seconds

### Example: `.node.js` (CommonJS)

```javascript
// my-node.node.js
module.exports = {
  manifest: {
    type: 'my-node',
    name: 'My Node',
    category: 'My Nodes',
    inputSchema: {
      text: { type: 'string', required: true },
    },
    outputSchema: {
      result: { type: 'string' },
    },
  },
  async execute({ inputs }) {
    return {
      status: 'success',
      outputs: { result: String(inputs.text || '').toUpperCase() },
      logs: [`Processed: ${inputs.text}`],
      artifacts: [],
    };
  },
};
```

### Example: `.node.mjs` (ESM)

```javascript
// my-node.node.mjs
export const manifest = {
  type: 'my-node-mjs',
  name: 'My Node MJS',
  category: 'My Nodes',
  inputSchema: {
    text: { type: 'string', required: true },
  },
  outputSchema: {
    result: { type: 'string' },
  },
};

export async function execute({ inputs }) {
  return {
    status: 'success',
    outputs: { result: String(inputs.text || '').toUpperCase() },
    logs: [`Processed: ${inputs.text}`],
    artifacts: [],
  };
}
```

> Full details: [Local Desktop Nodes](../custom-nodes/local-desktop-nodes.md).

---

## Hot Reload

Local provider monitors the custom nodes folder and reloads automatically:

- **Interval**: 1.5 seconds
- **Detection**: file modified time + size
- **Reload**: automatic, no app restart

---

## Native Theme

Desktop supports native light/dark theme integration:

- **Light**: light UI with light title bar
- **Dark**: dark UI with dark title bar

---

## Desktop vs Web

| Aspect | Desktop | Web (Enterprise) |
|---|---|---|
| Database | Embedded PostgreSQL | External PostgreSQL |
| Installation | Single installer | Manual (Node.js + PostgreSQL) |
| Local Nodes | Supported | HTTP providers only |
| Multi-user | Single-user | Team usage |
| Network | Offline capable | Network required |
| Updates | Installer update | Git pull + rebuild |

---

## Troubleshooting

### App does not start

- Verify another instance is not running
- Try running as administrator
- Check port conflicts

### Database does not initialize

- `%APPDATA%\QANode\pg-data` may be corrupted
- Rename/remove folder and restart (data loss expected)

### Local nodes do not appear

- Check extension: `.node.js`, `.node.mjs`, `.node.cjs`
- Check exports: `manifest` and `execute`
- For `.node.js`, use CommonJS (`module.exports`) or configure `type: "module"` in `package.json`
- Open `%APPDATA%\@qanode\desktop\desktop-main.log` and search `Failed loading`

### Quick import error verification

1. Open **Settings > Providers** and test `Desktop Local JS Provider`
2. Check `%APPDATA%\@qanode\desktop\desktop-main.log`
3. Look for messages like:
- `Failed loading ".../your-node.node.mjs": Cannot find package 'x'`
- `Unexpected token 'export'` (usually `.node.js` with ESM syntax and no `type: "module"`)

---

## Tips

- Use `.node.js` with CommonJS for simple compatibility
- Use `.node.mjs` when you need explicit ESM
- Keep one folder per node with its own `package.json`
- Use hot-reload for fast iteration
