# Core Concepts

Before diving deeper into QANode, it is important to understand the key concepts that form the foundation of the platform.

---

## Hierarchy Overview

```
Projects
├── Scenarios (Flows)
│   ├── Nodes
│   │   ├── Configuration
│   │   ├── Connections (Edges)
│   │   └── Outputs/Inputs
│   └── Executions (Runs)
│       ├── Steps
│       ├── Logs
│       ├── Screenshots
│       └── PDF Report
└── Suites
    ├── Ordered Scenarios
    ├── Scheduling (Cron)
    └── Suite Executions
```

---

## Projects

A **project** is the highest level of organization. It groups test scenarios and suites related to a system, feature, or team.

Each project has:
- **Name** and **description**
- **Status**: Active, Completed, Cancelled, or Archived
- **Dates**: Planned start and end dates
- **Documentation**: Attached files (requirements, specifications, etc.)
- **Statistics**: Number of scenarios, suites, and recent executions

---

## Scenarios (Flows)

A **scenario** (also called a **flow**) is a test case represented visually as a graph of connected nodes. Each scenario describes a sequence of actions to be executed — such as navigating a website, calling an API, or querying a database.

Scenarios are composed of:
- **Nodes**: Individual actions (click, type, make a request, etc.)
- **Connections (Edges)**: Links between nodes that define the execution order
- **Configurations**: Parameters for each node

### Visual Example

```
[Navigate] → [Fill email] → [Fill senha] → [Click login] → [Assert welcome]
```

In this example, each box is a node and the arrows represent connections indicating the execution flow.

---

## Nodes

**Nodes** are the fundamental building blocks of a scenario. Each node performs a specific action and can produce outputs that are consumed by subsequent nodes.

### Node Categories

| Category | Color | Nodes |
|----------|-------|-------|
| **Flow Control** | 🟡 Yellow | If, Switch, Loop, Merge |
| **Web** | 🔵 Blue | Web Flow, Smart Locators |
| **API** | 🟣 Purple | HTTP Request |
| **Database** | 🟢 Green | PostgreSQL, MySQL, MariaDB, Oracle, MongoDB |
| **Infrastructure** | 🔵 Light Blue | SSH Command |
| **Utilities** | ⚪ Gray | Set Variable, Log, Wait, Stop and Fail, Custom JavaScript |
| **Custom Nodes** | 🩷 Pink | Nodes from external providers |

### Anatomy of a Node

Each node has:

- **Input Handles** (●): Points where connections from other nodes arrive
- **Output Handles** (●): Points from which connections leave to other nodes
- **Configuration**: Fields specific to the node type
- **Outputs**: Data produced after execution (accessible via expressions)
- **Label**: Identifying name of the node in the flow

### Outputs and the Expression System

When a node is executed, it produces **outputs** — data that can be used by subsequent nodes. To access this data, use the **expression** system with the `{{ }}` syntax:

```
{{ steps["Node Name"].outputs.property }}
```

**Examples:**
```
{{ steps["http-request"].outputs.json.token }}    → Token from an API response
{{ steps.login.outputs.extracts.userName }}   → Text extracted from a page
{{ steps.query.outputs.rows[0].email }}       → First email from a SQL result
```

> For full details, see the [Expressions](../expressions/expressions.md) documentation.

---

## Connections (Edges)

**Connections** link the output of one node to the input of another, defining the execution order. QANode executes nodes in **topological order** — meaning a node is only executed after all nodes connected to its input have been completed.

### Conditional Connections

Some nodes have multiple conditional outputs:

- **If** → `true` and `false` outputs
- **Switch** → outputs for each case + `default`
- **Loop** → `loop` (next iteration) and `done` (end of loop) outputs

Only the path corresponding to the evaluated condition will be executed. Nodes in inactive paths will be **skipped**.

---

## Executions (Runs)

An **execution** is an instance of a scenario being processed. When running a scenario, QANode:

1. Resolves the topological order of the nodes
2. Executes each node sequentially
3. Evaluates expressions and injects data from previous outputs
4. Records logs, screenshots, and artifacts
5. Generates a PDF report at the end

Each execution has:
- **Status**: `running`, `success`, or `failed`
- **Duration**: Total execution time
- **Steps**: Individual result of each node
- **Artifacts**: Screenshots, PDFs, and other generated files
- **Logs**: Detailed messages from each step

---

## Test Suites

A **suite** groups multiple scenarios for sequential execution. Suites are useful for:

- Running multiple scenarios in order
- Scheduling periodic executions (daily, hourly, etc.)
- Stopping on the first failure (optional)

### Scheduling

Suites can be scheduled using **cron expressions**:

| Expression | Meaning |
|------------|---------|
| `0 8 * * *` | Every day at 8:00 AM |
| `0 */2 * * *` | Every 2 hours |
| `0 9 * * 1-5` | Monday to Friday at 9:00 AM |
| `30 18 * * 5` | Friday at 6:30 PM |

---

## Variables

**Variables** are global reusable values in any scenario. Supported types:

| Type | Use |
|------|-----|
| `string` | Text (URLs, names, messages) |
| `number` | Numbers (timeouts, limits) |
| `boolean` | Flags (true/false) |
| `json` | Complex objects and arrays |
| `secret` | Encrypted values (passwords, tokens) |

Access variables in your flows with: `{{ variables.variableName }}`

---

## Credentials

**Credentials** store connection information securely and centrally. Supported types:

- **HTTP/API** — Base URL, authentication, headers
- **PostgreSQL**, **MySQL**, **MariaDB**, **Oracle** — Database connection data
- **MongoDB** — URI or connection data
- **SSH** — Host, user, password, or private key

Credentials can be referenced by nodes that need a connection, preventing passwords from being exposed in flows.


---

## Next Steps

Now that you know the core concepts:

- [Flow Editor](../flow-editor/overview.md) — Learn to use the visual editor
- [Node Reference](../nodes/flow-control/if.md) — Learn about all nodes in detail
- [Expressions](../expressions/expressions.md) — Master the interpolation system
