# Desktop Version

The QANode desktop version is a **Community Edition** application with no restrictions on its core flow functionality.

---

## Advantages of the Desktop Version

| Feature | Description |
|---------|-------------|
| **Simple Installation** | A single installer, no external dependencies |
| **Embedded Database** | Integrated PostgreSQL, no configuration needed |
| **Local Nodes** | Create custom nodes as local files (hot-reload) |
| **All-in-One** | Frontend, API, Worker, and database in a single app |
| **Portable** | Works offline, no server required |

---

## Installation

Refer to the [Installation](../getting-started/installation.md) guide for detailed instructions.

### Minimum Requirements

- **Windows** 10/11 (64-bit)
- **RAM**: 4 GB minimum (8 GB recommended)
- **Disk**: 500 MB for installation + space for data

---

## First Run

On the first run, QANode:

1. Displays a **splash screen** with progress
2. Initializes the **embedded PostgreSQL**
3. Runs **migrations** (creates tables)
4. Starts the **API server**
5. Opens the **main interface**

<!-- ![Splash screen](../../assets/images/splash-screen.png) -->
*Image: Splash screen showing initialization progress with status bar*

> The first initialization may take a few extra seconds while the database is prepared.

---

## Data Structure

QANode stores data in the user folder:

```
%APPDATA%\QANode\
├── pg-data/              # Embedded PostgreSQL data
├── storage/              # Artifacts (screenshots, PDFs)
└── logs/                 # Application logs
```

```
%USERPROFILE%\Documents\QANode\
└── custom nodes/         # Local custom nodes
    └── sample-hash-node/ # Sample node
```

> **Important:** Do not manually modify files in `pg-data/`.

---

## Local Custom Nodes

The desktop version includes support for **local nodes** — JavaScript files that are automatically detected and loaded.

### Nodes Folder

```
%USERPROFILE%\Documents\QANode\custom nodes\
```

### Creating a Local Node

1. Create a file with the `.node.js` extension in the folder above
2. Export `manifest` and `execute`
3. QANode automatically detects it in ~1.5 seconds

```javascript
// meu-no.node.js
export const manifest = {
  type: 'meu-no',
  name: 'Meu Nó',
  category: 'Meus Nós',
  inputSchema: {
    texto: { type: 'string', required: true }
  },
  outputSchema: {
    resultado: { type: 'string' }
  }
};

export async function execute({ inputs }) {
  return {
    status: 'success',
    outputs: { resultado: inputs.texto.toUpperCase() },
    logs: [`Processado: ${inputs.texto}`],
    artifacts: []
  };
}
```

> For full details, see [Desktop Local Nodes](../custom-nodes/desktop-local-nodes.md).

---

## Hot-Reload

The local provider monitors the custom nodes folder and automatically reloads when it detects changes:

- **Interval**: 1.5 seconds
- **Detection**: Modification date + file size
- **Reload**: Automatic, without restarting the app

Ideal for rapid development of custom nodes.

---

## Native Theme

QANode desktop supports light and dark themes that integrate with the native Windows title bar:

- **Light Theme** — Light interface with white title bar
- **Dark Theme** — Dark interface with dark title bar

The theme can be changed through the interface and synchronizes with the operating system's title bar.

---

## Differences from the Web Version

| Aspect | Desktop | Web (Enterprise) |
|--------|---------|------------------|
| **Database** | Embedded PostgreSQL | External PostgreSQL |
| **Installation** | Single installer | Manual (Node.js + PostgreSQL) |
| **Local Nodes** | Supported (hot-reload) | HTTP providers only |
| **Multi-user** | Individual use | Shared team |
| **Network** | Works offline | Requires network |
| **Updates** | Manual installer | Git pull + rebuild |

---

## Troubleshooting

### The app does not start

- Check that another instance is not already running
- Try running as administrator
- Check that port 5432 is not in use (external PostgreSQL may conflict)

### Database does not initialize

- The `%APPDATA%\QANode\pg-data` folder may be corrupted
- Try renaming/removing the folder and restarting (data will be lost)

### Local nodes do not appear

- Check that the file has the `.node.js`, `.node.mjs`, or `.node.cjs` extension
- Check that it exports `manifest` and `execute`
- Open DevTools (Ctrl+Shift+I) to view error logs

---

## Tips

- Use the desktop version for **individual development and testing**
- Take advantage of **local nodes** for rapid prototyping
- **Hot-reload** eliminates the need to restart during development
