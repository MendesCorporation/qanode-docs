# QANode — Documentation

Welcome to the official documentation of **QANode Community Edition**, the test automation platform with a visual flow editor.

---

## Getting Started

| Page | Description |
|------|-------------|
| [Introduction](getting-started/introduction.md) | What is QANode and platform overview |
| [Installation](getting-started/installation.md) | How to install and configure QANode |
| [Quick Start](getting-started/quick-start.md) | Create your first test in minutes |
| [Core Concepts](getting-started/concepts.md) | Projects, flows, nodes, suites, and executions |

---

## Flow Editor

| Page | Description |
|------|-------------|
| [Editor Overview](flow-editor/overview.md) | Interface, canvas, node palette, and properties panel |
| [Working with Nodes](flow-editor/working-with-nodes.md) | Add, connect, configure, and remove nodes |
| [Execution & Debugging](flow-editor/execution-debugging.md) | Run flows, view results, and debug failures |

---

## Node Reference

### Flow Control
| Node | Description |
|------|-------------|
| [If](nodes/flow-control/if.md) | Conditional branch (true/false) |
| [Switch](nodes/flow-control/switch.md) | Multiple branch by value or condition |
| [Loop](nodes/flow-control/loop.md) | Repeat by count, array, or condition |
| [Merge](nodes/flow-control/merge.md) | Join multiple execution paths |

### Web
| Node | Description |
|------|-------------|
| [Web Flow](nodes/web/web-flow.md) | Web automation with multiple steps and CSS/XPath selectors |
| [Smart Locators](nodes/web/smart-locators.md) | Web automation with Playwright semantic locators |

### API
| Node | Description |
|------|-------------|
| [HTTP Request](nodes/api/http-request.md) | HTTP requests (GET, POST, PUT, PATCH, DELETE) |

### Database
| Node | Description |
|------|-------------|
| [PostgreSQL](nodes/database/postgresql.md) | Queries and operations on PostgreSQL |
| [MySQL](nodes/database/mysql.md) | Queries and operations on MySQL |
| [MariaDB](nodes/database/mariadb.md) | Queries and operations on MariaDB |
| [Oracle](nodes/database/oracle.md) | Queries and operations on Oracle |
| [MongoDB](nodes/database/mongodb.md) | Operations on MongoDB (find, insert, update, etc.) |

### Infrastructure
| Node | Description |
|------|-------------|
| [SSH Command](nodes/infrastructure/ssh-command.md) | Remote SSH command execution |

### Utilities
| Node | Description |
|------|-------------|
| [Set Variable](nodes/utilities/set-variable.md) | Define variables at runtime |
| [Log](nodes/utilities/log.md) | Record messages in the execution log |
| [Wait](nodes/utilities/wait.md) | Wait for a time duration or condition |
| [Stop and Fail](nodes/utilities/stop-and-fail.md) | Stop the flow with a failure status |
| [Custom JavaScript](nodes/utilities/custom-javascript.md) | Execute custom JavaScript code |

---

## Custom Nodes

| Page | Description |
|------|-------------|
| [Overview](custom-nodes/overview.md) | How the provider system works |
| [Creating a Provider - Enterprise](custom-nodes/creating-enterprise-provider.md) | Step-by-step guide to create an HTTP provider |
| [API Contract](custom-nodes/api-contract.md) | Required endpoints and data format |
| [Examples](custom-nodes/enterprise-examples.md) | Examples in Node.js, Python, Java, C#, and Go |
| [Desktop: Local Nodes](custom-nodes/local-desktop-nodes.md) | Creating local nodes in the desktop version |
| [QANode.MD (AI)](QANode.MD) | AI guide to build nodes and troubleshoot issues |

---

## Management

| Page | Description |
|------|-------------|
| [Projects](projects/projects.md) | Creating and managing test projects |
| [Test Suites](suites/suites.md) | Grouping flows and scheduling |
| [Variables](variables/variables.md) | Global and secret variables |
| [Credentials](credentials/credentials.md) | Secure credential management |

---

## Monitoring & Reports

| Page | Description |
|------|-------------|
| [Dashboard - Enterprise](dashboard/enterprise-dashboard.md) | Panels, widgets, and charts |
| [Reports - Enterprise](reports/enterprise-reports.md) | PDF report generation and email delivery |

---

## Reference

| Page | Description |
|------|-------------|
| [Expressions](expressions/expressions.md) | Expression system `{{ }}` and interpolation |
| [Administration - Enterprise](administration/enterprise-administration.md) | Users, roles, permissions, SMTP, and audit |
| [Desktop Version](desktop/desktop.md) | Installation and exclusive desktop features |
| [Chrome Extension](desktop/chrome-extension.md) | Browser test recorder |
