# SSH Command Node

The **SSH Command** node executes commands on remote servers over SSH. It supports multiple sequential steps, password or private-key authentication, file upload/download, and output capture.

---

## Overview

[Overview](../../assets/images/nos-ssh-visao-geral.mp4)

| Property | Value |
|----------|-------|
| **Type** | `ssh-command` |
| **Category** | Infrastructure |
| **Color** | 🔵 Light Blue (#0ea5e9) |
| **Input** | `in` |
| **Output** | `out` |

---

## Connection

### Using Saved Credentials (Recommended)

Select an **SSH** credential in the **Credential** field.

### Manual Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Host** | `string` | Server address |
| **Port** | `number` | SSH port (default: 22) |
| **Username** | `string` | Username |
| **Password** | `string` | Password authentication |
| **Private Key** | `string` | SSH private key |
| **Passphrase** | `string` | Private-key passphrase, when protected |

### Authentication Methods

| Method | Required Fields |
|--------|-----------------|
| **Password** | Username + Password |
| **Private Key** | Username + Private Key + optional Passphrase |

---

## Configuration

### Global Timeout

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **Timeout (ms)** | `number` | 30000 | Maximum time for the full execution |

### SSH Steps

The node supports multiple steps executed sequentially in the same SSH connection. Use **+ Add step** to include commands, file uploads, and downloads.

| Step Type | Description |
|-----------|-------------|
| **Command** | Runs a command in the remote shell |
| **Send file** | Sends a `fileRef` to the remote server via SFTP |
| **Download file** | Downloads a remote file via SFTP, with SCP fallback |

All steps run in the same SSH session. Commands like `cd /my/folder` affect following steps when they use **Current folder**.

### Command Step

| Field | Type | Description |
|-------|------|-------------|
| **Label** | `string` | Descriptive step name |
| **Command** | `string` | Command to execute (supports `{{ }}`) |
| **Wait for Match** | `boolean` | Wait for text to appear in output |
| **Match String** | `string` | Text to wait for |
| **Use Regex** | `boolean` | Interpret the match as a regex |
| **Match Timeout (ms)** | `number` | Maximum match wait time |

### Send File Step

| Field | Type | Description |
|-------|------|-------------|
| **Label** | `string` | Descriptive step name |
| **File** | `fileRef` | File to send |
| **Destination** | selection | **Current folder** or **Remote path** |
| **Remote path** | `string` | Absolute or relative server path when destination is remote path |
| **Keep file name** | `boolean` | Uses the original file name |
| **File name** | `string` | New server filename when **Keep file name** is off |
| **Create remote folders** | `boolean` | Creates missing directories before upload |

With **Destination = Current folder**, QANode sends the file to the current SSH shell directory. This enables:

```
Step 1: cd /opt/my-app/uploads
Step 2: Send file → Destination: Current folder
```

With **Destination = Remote path**, provide the path directly. If it ends with `/`, the file is saved inside that folder.

### Download File Step

| Field | Type | Description |
|-------|------|-------------|
| **Label** | `string` | Descriptive step name |
| **Source** | selection | **Current folder** or **Remote path** |
| **File name** | `string` | Filename when source is current folder |
| **Remote path** | `string` | File path on the server when source is remote path |
| **Variable name** | `string` | Key used in `outputs.files` |
| **Output file name** | `string` | Name saved in QANode. If empty, uses the remote name |

If the extension is omitted from the remote path, QANode tries to resolve the file by name in the remote folder. For example, `/tmp/data` can find `/tmp/data.csv` when that file exists on the server.

The MIME type is inferred automatically from content and filename.

### Wait for Match

**Wait for Match** lets a command step wait until specific text appears in the output. Useful for:

- waiting for services to start;
- waiting for interactive prompts;
- confirming that an operation completed.

Example:

```
Command: systemctl restart nginx
Wait for Match: true
Match String: "Active: active (running)"
Timeout: 10000
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `stdout` | `string` | Complete standard output |
| `exitCode` | `number` | Exit code (0 = success) |
| `commands` | `array` | Result of each individual command step |
| `files` | `object` | Downloaded files, when there is a **Download file** step |
| `fileRef` | `fileRef` | Shortcut to the last downloaded file |
| `fileInfo` | `object` | Downloaded file metadata, such as name, MIME type, size, remote path, and transfer method |

### Accessing Outputs

```
// Full output
{{ steps.ssh.outputs.stdout }}

// Exit code
{{ steps.ssh.outputs.exitCode }}

// Specific command result
{{ steps.ssh.outputs.commands[0].stdout }}
{{ steps.ssh.outputs.commands[0].exitCode }}

// Downloaded file by variable name
{{ steps.ssh.outputs.files.report }}

// Shortcut to the last downloaded file
{{ steps.ssh.outputs.fileRef }}
```

---

## Practical Examples

### Check service status

```
Step 1:
  Label: Check Nginx
  Command: systemctl status nginx
```

### Application deploy

```
Step 1:
  Label: Go to directory
  Command: cd /opt/my-app

Step 2:
  Label: Pull repository
  Command: git pull origin main

Step 3:
  Label: Install dependencies
  Command: npm install --production

Step 4:
  Label: Restart service
  Command: pm2 restart my-app
  Wait for Match: true
  Match String: "online"
```

### Send and download files in the current folder

```
Step 1:
  Label: Enter folder
  Command: cd /root/tests

Step 2:
  Label: Send data
  Type: Send file
  File: {{ steps["file-generate"].outputs.fileRef }}
  Destination: Current folder
  Keep file name: true

Step 3:
  Label: Download result
  Type: Download file
  Source: Current folder
  File name: result.csv
  Variable name: result
```

After the download:

```
{{ steps.ssh.outputs.files.result }}
{{ steps.ssh.outputs.fileInfo.result.name }}
```

---

## Flow with Validation

```
[SSH Command: "deploy"]
    │
    ▼
[If: {{ steps.ssh.outputs.exitCode }} === 0]
    │ true → [HTTP Request: GET /health-check]
    │            │
    │            ▼
    │         [Assert: status === 200]
    │
    │ false → [Log: "Deploy failed: {{ steps.ssh.outputs.stderr }}"]
              → [Stop and Fail]
```

---

## Tips

- **Use saved credentials** to avoid exposing passwords/keys in flows.
- **Check `exitCode`**: `0` indicates success, other values indicate errors.
- Use expressions in commands: `echo "Deploy to {{ variables.ENVIRONMENT }}"`.
- **Wait for Match** is essential for long-running commands such as restarts and builds.
- All steps run in the **same SSH session**, so the working directory is preserved between steps.
- Use **Current folder** when a previous command already positioned the shell in the correct directory.
- Use **Remote path** when you want to be explicit and independent from a previous `cd`.
- For downloads, prefer clear **Variable names**, such as `report`, `data`, or `evidence`.
- Commands with `{{ }}` are evaluated before execution.
