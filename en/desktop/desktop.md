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

- **Windows** 10/11 (64-bit), **macOS** 11+ (Intel or Apple Silicon), or **Linux** 64-bit with AppImage
- **RAM**: 4 GB minimum (8 GB recommended)
- **Disk**: 2 GB for installation + space for data and evidence

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
Windows: %APPDATA%\@qanode\desktop\
macOS:   ~/Library/Application Support/@qanode/desktop/
Linux:   ~/.config/@qanode/desktop/
```

Inside the data folder:

```text
postgres/          # Embedded PostgreSQL data
storage/           # Artifacts, screenshots, and PDFs
desktop-main.log   # Startup and local services log
```

```text
%USERPROFILE%\Documents\QANode\
└── custom nodes/         # Local custom nodes
    └── sample-hash-node/ # Sample node
```

> Important: do not manually modify files under `postgres/`. Back up the entire data folder before any recovery procedure.

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

QANode Desktop supports light and dark themes integrated with the operating system's native window:

- **Light**: light UI with light title bar
- **Dark**: dark UI with dark title bar

---

## Updates

On **Windows**, QANode checks the published version and shows a notice only when it is newer than the installed version. Click **Download** to open the download page and install the new version manually. The app does not download or install Windows updates in the background.

On **macOS**, the built-in updater remains available: it tracks the download and offers the option to restart and install once the new version is ready.

---

## Desktop vs Web

| Aspect | Desktop | Web (Enterprise) |
|---|---|---|
| Database | Embedded PostgreSQL | External PostgreSQL |
| Installation | Single installer | Manual (Node.js + PostgreSQL) |
| Local Nodes | Supported | HTTP providers only |
| Multi-user | Single-user | Team usage |
| Network | Offline capable | Network required |
| Updates | Built-in updater on macOS; notice with manual download on Windows | Deployment environment update |

---

## Troubleshooting

### App does not start

- Verify another instance is not running
- Try running as administrator
- Check port conflicts

### Database does not initialize

- Open `desktop-main.log` in the data folder and look for `Fatal startup error`
- Do not delete the `postgres/` folder: it contains local projects, scenarios, and history
- Make a copy of the entire data folder before any recovery procedure
- Send the relevant log excerpt to support to identify whether the failure occurred during startup, schema update, or another local service

### Local nodes do not appear

- Check extension: `.node.js`, `.node.mjs`, `.node.cjs`
- Check exports: `manifest` and `execute`
- For `.node.js`, use CommonJS (`module.exports`) or configure `type: "module"` in `package.json`
- Open `desktop-main.log` in your system's data folder and search for `Failed loading`

### Quick import error verification

1. Open **Settings > Providers** and test `Desktop Local JS Provider`
2. Check `desktop-main.log` in your system's data folder
3. Look for messages like:
- `Failed loading ".../your-node.node.mjs": Cannot find package 'x'`
- `Unexpected token 'export'` (usually `.node.js` with ESM syntax and no `type: "module"`)

---

## Tips

- Use `.node.js` with CommonJS for simple compatibility
- Use `.node.mjs` when you need explicit ESM
- Keep one folder per node with its own `package.json`
- Use hot-reload for fast iteration
