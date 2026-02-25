# Introduction to QANode

**QANode** is a test automation platform that allows you to create, run, and manage automated tests through a **visual flow editor** based on nodes. With an intuitive drag-and-connect interface, you can build complex test scenarios without writing code — QANode, through its extensibility with custom nodes, is prepared to run any type of test.

---

## What is QANode?

QANode was designed for QA teams, developers, and test engineers who need a modern, visual tool for test automation. The platform covers the main testing areas:

### Web Automation
Test user interfaces using **Playwright**, the most modern automation framework on the market. QANode offers two specialized nodes:

- **Web Flow** — Automation based on CSS selectors, XPath, data-testid, and text
- **Smart Locators** — Automation with semantic locators (`getByRole`, `getByLabel`, `getByPlaceholder`, etc.)

Both support Chromium, headless mode, screenshot capture, and session reuse.

### API Testing
Execute HTTP requests with full support for:
- GET, POST, PUT, PATCH, and DELETE methods
- Bearer, Basic, and API Key authentication
- Custom headers and request body
- Integration with saved credentials

### Database
Connect and validate data in:
- **PostgreSQL**, **MySQL**, **MariaDB**, **Oracle** — with a visual query builder or direct SQL
- **MongoDB** — with find, insert, update, delete, and aggregation pipeline operations

### Infrastructure
Execute remote commands via **SSH** with support for multiple steps, password or private key authentication, and output capture.

### Extensibility
Create **custom nodes** in any programming language (Node.js, Python, Java, C#, Go, etc.) through the HTTP provider system. (Community Edition supports JavaScript only)

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Visual Editor** | Interactive canvas with drag-and-connect nodes |
| **18+ Native Nodes** | Flow control, web, API, database, infrastructure, and utilities |
| **Custom Nodes** | Extensible with HTTP providers in any language |
| **Projects and Suites** | Organize tests in projects and group them into suites |
| **Scheduling** | Run tests automatically with cron expressions |
| **Dashboard** | Customizable panels with real-time charts and metrics |
| **PDF Reports** | Automatic generation with configurable templates |
| **Variables and Credentials** | Centralized and secure management |
| **Access Control** | Users, roles, and granular permissions |
| **Audit** | Complete log of all system actions |
| **Chrome Recorder** | Extension that records interactions and generates flows |
| **Desktop Version** | Standalone application with embedded database |
| **Expressions** | `{{ }}` interpolation system with access to outputs and variables |
| **Real Time** | Live updates of execution status |

---

## Architecture

QANode is composed of four main components:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Frontend    │    │  API Server  │    │   Worker     │
│  (React)     │◄──►│  (Fastify)   │◄──►│  (Executor)  │
│  Vite + TS   │    │  REST API    │    │  Playwright  │
└─────────────┘    └──────┬───────┘    └─────────────┘
                          │
                   ┌──────▼───────┐
                   │  PostgreSQL   │
                   │  (Prisma ORM) │
                   └──────────────┘
```

- **Frontend** — User interface in React with TypeScript, built with Vite
- **API Server** — REST server in Fastify with JWT authentication
- **Worker** — Execution engine that processes flows and controls Playwright
- **PostgreSQL** — Relational database for data storage (via Prisma ORM)

### Desktop Version

The desktop version packages all components into a single **Electron** application, including an embedded PostgreSQL. Just install and use — no infrastructure configuration required.

---

## Community Edition vs Enterprise

This documentation covers the **Community Edition**, which is free. The table below summarizes the differences:

| Feature | Community | Enterprise |
|---------|-----------|------------|
| Flow Editor | ✅ | ✅ |
| All Native Nodes | ✅ | ✅ |
| Custom Nodes | ✅ | ✅ |
| Projects and Suites | ✅ | ✅ |
| Scheduling | ✅ | ✅ |
| Variables and Credentials | ✅ | ✅ |
| PDF Reports | ✅ | ✅ |
| Chrome Step Recorder | ✅ | ✅ |
| Web Version | ❌ | ✅ |
| Customizable Dashboard | ❌ | ✅  |
| Multi-user | ❌ | ✅  |
| MFA (2FA Authentication) | ❌ | ✅ |
| Audit Logs | ❌ | ✅ |
| Report Generator | ❌ | ✅ |
| Alarm Scheduling | ❌ | ✅ |
| Webhooks | ❌ | ✅ |
| Dedicated Support | ❌ | ✅ |

---

## Next Steps

- [Installation](installation.md) — Set up QANode in your environment
- [Quick Start](quick-start.md) — Create your first test in minutes
- [Core Concepts](concepts.md) — Understand the platform structure
