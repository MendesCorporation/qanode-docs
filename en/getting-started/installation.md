# Installation


---

## Desktop Version

The desktop version is the easiest way to get started. It includes all the necessary components in a single installer, including an embedded PostgreSQL database.

### Requirements

- **Windows** 10/11, 64-bit
- **macOS** 11 or later, Intel or Apple Silicon
- **Linux** 64-bit with AppImage support
- **RAM**: 8 GB minimum (16 GB recommended)
- **Disk Space**: 2 GB for installation, plus the space used by evidence and executions

### Installation on Windows

1. Download the `QANode Setup.exe` installer from https://www.qanode.com
2. Run the installer and follow the instructions
3. When finished, QANode will open automatically

[QANode installer on Windows](../../assets/images/instalacao-nsis.png)
*Image: QANode NSIS installer screen on Windows*

### Installation on macOS

1. Download the `.dmg` file for your processor: Apple Silicon (`arm64`) or Intel (`x64`)
2. Open the DMG and move QANode to **Applications**
3. Open QANode from the **Applications** folder or macOS search

The application is signed and notarized by Apple. After an update, macOS may take a few moments to refresh the icon and search index.

### Installation on Linux

1. Download the `.AppImage` file
2. Mark the file as executable
3. Open the AppImage

Example:

```bash
chmod +x QANode-Setup-x64.AppImage
./QANode-Setup-x64.AppImage
```

On first launch, QANode will:
- Initialize the embedded PostgreSQL database
- Create the necessary tables (automatic migrations)
- Start the API server
- Open the application interface

> **Note:** The first startup may take a few extra seconds while the database is being prepared. A splash screen will be displayed during this process.

### Data Folder

The desktop version stores the database, evidence, files, and logs in the user's data folder:

```text
Windows: %APPDATA%\@qanode\desktop\
macOS:   ~/Library/Application Support/@qanode/desktop/
Linux:   ~/.config/@qanode/desktop/
```

Inside it, `postgres/` contains the embedded database, `storage/` stores artifacts, and `desktop-main.log` records application startup.

> **Important:** Never manually modify or delete the `postgres/` folder. Before any recovery procedure, make a copy of the entire data folder.

---


## Troubleshooting

### The desktop version does not start

- Check that another instance of QANode is not already running
- Open `desktop-main.log` in the data folder and look for `Fatal startup error`
- On Windows, confirm that the Microsoft Visual C++ Redistributable x64 is up to date
- Do not delete the `postgres/` folder; preserve the database and send the log to support for diagnosis

---

## Next Steps

- [Quick Start](quick-start.md) — Create your first test
- [Core Concepts](concepts.md) — Understand the platform structure
