# SSH Command Node

The **SSH Command** node allows you to execute commands on remote servers via the SSH protocol. It supports multiple steps (sequential commands), authentication by password or private key, and output capture.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `ssh-command` |
| **Category** | Infrastructure |
| **Color** | Light Blue (#0ea5e9) |
| **Input** | `in` |
| **Output** | `out` |

---

## Connection

### Using Saved Credentials (Recommended)

Select a credential of type **SSH** in the **Credential** field.

### Manual Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Host** | `string` | Server address |
| **Port** | `number` | SSH port (default: 22) |
| **Username** | `string` | Username |
| **Password** | `string` | Password (password authentication) |
| **Private Key** | `string` | SSH private key (key-based authentication) |
| **Passphrase** | `string` | Key passphrase (if key is protected) |

### Authentication Methods

| Method | Required Fields |
|--------|----------------|
| **Password** | Username + Password |
| **Private Key** | Username + Private Key + Passphrase (optional) |

---

## Configuration

### Global Timeout

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **Timeout (ms)** | `number` | 30000 | Maximum time for total execution |

### Steps (SSH Steps)

The node supports multiple steps executed sequentially within the same SSH connection:

| Field | Type | Description |
|-------|------|-------------|
| **Label** | `string` | Descriptive name for the step |
| **Command** | `string` | Command to execute (supports `{{ }}`) |
| **Wait for Match** | `boolean` | Wait for text to appear in the output |
| **Match String** | `string` | Text to wait for |
| **Use Regex** | `boolean` | Interpret match as a regular expression |
| **Match Timeout (ms)** | `number` | Maximum time to wait for match |

### Wait for Match

The **Wait for Match** feature allows a step to wait until specific text appears in the command output. Useful for:
- Waiting for services to start
- Waiting for interactive prompts
- Confirming that an operation has completed

**Example:**
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
| `stderr` | `string` | Error output |
| `exitCode` | `number` | Exit code (0 = success) |
| `steps` | `array` | Result of each individual step |

### Accessing Outputs

```
// Full output
{{ steps.ssh.outputs.stdout }}

// Exit code
{{ steps.ssh.outputs.exitCode }}

// Result of a specific step
{{ steps.ssh.outputs.steps[0].stdout }}
{{ steps.ssh.outputs.steps[0].exitCode }}
```

---

## Practical Examples

### Check service status

```
Step 1:
  Label: Check Nginx
  Command: systemctl status nginx
```

### Application deployment

```
Step 1:
  Label: Go to directory
  Command: cd /opt/minha-app

Step 2:
  Label: Pull repository
  Command: git pull origin main

Step 3:
  Label: Install dependencies
  Command: npm install --production

Step 4:
  Label: Restart service
  Command: pm2 restart minha-app
  Wait for Match: true
  Match String: "online"
```

### Collect server information

```
Step 1:
  Label: Disk usage
  Command: df -h /

Step 2:
  Label: Memory
  Command: free -m

Step 3:
  Label: Processes
  Command: ps aux --sort=-%mem | head -10
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
    │ false → [Log: "Deploy falhou: {{ steps.ssh.outputs.stderr }}"]
              → [Stop and Fail]
```

---

## Tips

- **Use saved credentials** to avoid exposing passwords/keys in flows
- **Check the exitCode** — `0` indicates success, other values indicate an error
- Use **expressions** in commands: `echo "Deploy em {{ variables.ENVIRONMENT }}"`
- **Wait for Match** is essential for long-running commands (restarts, builds)
- All steps run within the **same SSH session** — the working directory is preserved between steps
- Commands with `{{ }}` are evaluated before execution
