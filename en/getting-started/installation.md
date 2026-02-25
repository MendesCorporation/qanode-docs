# Installation


---

## Desktop Version

The desktop version is the easiest way to get started. It includes all the necessary components in a single installer, including an embedded PostgreSQL database.

### Requirements

- **Operating System**: Windows 10/11 (64-bit)
- **RAM**: 8 GB minimum (16 GB recommended)
- **Disk Space**: 2 GB for installation

### Installation on Windows

1. Download the `QANode Setup.exe` installer from https://www.qanode.com
2. Run the installer and follow the instructions
3. When finished, QANode will open automatically

[QANode installer on Windows](../../assets/images/instalacao-nsis.png)
*Image: QANode NSIS installer screen on Windows*

On first launch, QANode will:
- Initialize the embedded PostgreSQL database
- Create the necessary tables (automatic migrations)
- Start the API server
- Open the application interface

> **Note:** The first startup may take a few extra seconds while the database is being prepared. A splash screen will be displayed during this process.

### Data Folder

The desktop version stores its data in:

```
%USERPROFILE%\AppData\Roaming\QANode\
├── pg-data/         # Embedded PostgreSQL data
├── storage/         # Artifacts (screenshots, PDFs, etc.)
└── custom nodes/    # Local custom nodes
```

> **Important:** Never manually modify the files in `pg-data/`. Use the QANode interface to manage your data.

---


## Troubleshooting

### The desktop version does not start

- Check that another instance of QANode is not already running
- Try running as administrator
- Check that port 5432 is not in use by another PostgreSQL instance

---

## Next Steps

- [Quick Start](quick-start.md) — Create your first test
- [Core Concepts](concepts.md) — Understand the platform structure
