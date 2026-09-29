# MCP — AI Integration

> **Available in:** QANode Desktop Community and Enterprise. Accessible tools depend on the edition and the user's permissions.

**MCP** connects AI tools to QANode so users can work with projects, scenarios, runs, dashboards, and bugs directly from an IDE, corporate chat, or any compatible client.

In practice, the connected MCP client calls QANode resources using the permissions of the authenticated user. Everything created or changed continues to exist inside QANode as a normal platform object.

---

## When To Use

Use MCP when you want to work with QANode without leaving the environment where you are already investigating, documenting, or implementing:

- create or adjust QANode scenarios with AI assistance;
- automate flows that involve web, API, database, files, SSH, or mobile;
- investigate an existing scenario that is failing;
- turn project documentation into initial scenarios;
- create dashboards or ask questions about QANode metrics in natural language;
- open or analyze bugs while working in the code.

MCP does not replace the visual editor. Everything created through MCP still appears in QANode as a normal project, scenario, node, run, dashboard, suite, or bug.

---

## How To Connect

In **Desktop Community**, open **Connect AI** in the app. This page guides you through connecting a compatible client and keeps the local MCP available while QANode is open.

In an **Enterprise** installation, use an access token and the API URL:

1. Generate a token in **Settings → Access Tokens**
2. Configure the compatible MCP client
3. Provide the QANode API URL
4. Send the token in the `Authorization` header

Endpoint:

```text
https://qanode.company.com/api/mcp
```

Header:

```text
Authorization: Bearer qnt_xxxxx
```

Generic example:

```json
{
  "mcpServers": {
    "qanode": {
      "url": "https://qanode.company.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${QANODE_TOKEN}"
      }
    }
  }
}
```

The exact format depends on the compatible client, but the URL and header are the same.

---

## Security And Permissions

MCP follows the same QANode permissions.

| Area | Rule |
|------|------|
| Projects | Access is limited to projects allowed for the user |
| Scenarios | Creation, editing, and execution follow the user's permissions |
| Runs | Run lookup follows view permissions |
| Credentials | Secrets are never returned as plain text |
| Secret variables | Secret values are masked |
| Dashboards | Queries and widgets follow data permissions |
| Bugs | Workflow, required fields, and transitions still apply |

The token does not grant extra access. It only authenticates the connected client as that user or integration.

---

## What MCP Lets You Do

| Area | What becomes available |
|------|--------------|
| **Projects** | Search projects, consult attached documents, and use that context in scenarios or diagnostics |
| **Scenarios** | Create flows, import nodes, add steps, connect, validate, and execute |
| **Smart Web** | Create web automations, validate screens, extract data, use WebMCP tools, and prepare initial browser state when requested |
| **Mobile** | Create Android/iOS journeys with Appium and save them as Mobile Flow nodes |
| **Database** | Inspect schema and create database queries or validations |
| **Files** | Generate, extract, and reuse files in flows |
| **Variables** | Create global or project variables for reuse |
| **Credentials** | Register credentials from data provided by the user |
| **Suites** | Create, edit, execute, schedule, and diagnose failures |
| **Bugs** | Open, comment, attach files, transition, and investigate bugs |
| **Dashboards** | Create panels, widgets, and quick queries over QANode data |
| **Components** | Create reusable components for recurring flows |

---

## How A Request Works

When the user requests an action from the connected client, the general flow is:

1. The client sends the request to QANode through MCP.
2. QANode validates authentication, permissions, and required data.
3. The action is applied to real QANode objects, such as a scenario, run, dashboard, or bug.
4. The result returns to the client with the data available to that user.

For scenarios and tests, information such as project, target system, credential, test data, expected result, and cleanup rules helps QANode create a flow closer to real usage.

Example:

```text
In the Task Manager project, create a scenario that opens localhost:3008,
logs in with the test user, creates a task, marks it as completed,
and validates in the database that the status is completed.
```

Sensitive actions, such as using credentials, creating data in real systems, running destructive scenarios, or transitioning bugs, should be handled with the same care they would require inside QANode.

---

## Smart Web And Target Reference

On pages with tables, cards, kanban boards, or repeated lists, the same button or status can appear several times on the screen.

The Smart Web **Target Reference** records which row, card, or item should be used as context before clicking, validating, or extracting a value. This applies both to flows created in the editor and to flows created or adjusted through MCP.

Example:

```text
Validate that the status is Paid on the order created in this run,
not on any old order shown on the page.
```

This helps QANode act on the correct record even when the data changes on every run.

---

## Web And Mobile Sessions

When a scenario is split across more than one Smart Web or Mobile node, session reuse determines whether the nodes continue in the same browser, web context, or mobile session.

This is useful when:

- the first node logs in and the second continues authenticated;
- a purchase or registration starts on one screen and finishes on another;
- the mobile app needs to keep the same state across parts of the flow.

For already authenticated web sessions, such as enterprise systems with SSO, use named persisted sessions. For common flows that log in during the run itself, a temporary session per run is usually enough.

When a later node needs to reuse the session opened by an earlier node, **Session Storage** must keep that session available. The behavior is the same in the editor and in actions performed through MCP.

---

## Skipped Nodes In Executions

Nodes marked as **Skip during run** stay visible on the canvas, but normal executions record those nodes as skipped and continue the flow without running them.

This behavior also applies to executions started through MCP or scenarios edited through MCP. It is useful for drafts, experiments, and temporarily isolated parts of a flow, but it does not replace fixing a real failure.

---

## Project Documents

MCP exposes project documents as context for creating scenarios, answering questions, or understanding business rules.

For large documents, QANode can first search for relevant snippets inside the project attachments. This helps the client find pages, sections, or parts of the text before opening the full document.

QANode can read:

- text-based PDFs;
- scanned PDFs or images with OCR;
- DOCX;
- TXT, JSON, and CSV;
- XLSX with spreadsheet preview;
- metadata from other files.

When the snippet is not enough, the client can request a page, a page range, or the full document.

When search is scoped to a project, it can also consider the names, descriptions, and annotations of scenarios the user is allowed to view. This makes decisions and rules recorded on the canvas discoverable together with attached documents.

---

## AI-Created Scenario Documentation

When a scenario is created through MCP, QANode adds a standard annotation to the canvas containing:

- scenario name;
- description;
- preconditions;
- test cases;
- expected result.

Each scenario node represents one test case. The node name describes the action, such as **Fill in the form on the website** or **Query status in the database**, while the annotation owns the `CT-01`, `CT-02`, and subsequent numbering.

If the user provides an exact scenario or node name, the client should preserve it. When no name is provided, the scenario receives a concise purpose title, such as **Login validation**, and nodes receive action-oriented labels.

The test case list stays synchronized when nodes are added, renamed, imported, or removed through MCP. Existing regular canvas annotations are not replaced; the standard documentation is kept as its own scenario annotation.

---

## Official QANode Documentation

In addition to your project documents, MCP can also search the official QANode documentation.

This feature helps understand how to configure a node, confirm product behavior, or find the right documentation page without leaving the connected client.

Examples:

```text
How do I configure a Smart Web Flow with persisted session?
```

```text
What options exist for creating dashboard widgets?
```

The search uses the requested language when available and returns the most relevant documentation snippets.

---

## Dashboards

Through MCP, the user can create dashboards or query QANode metrics in natural language.

Examples:

```text
How many runs did we have in May?
```

```text
Create a public dashboard with total runs this month,
success rate, recent failures, and daily evolution.
```

For simple widgets, QANode uses the visual query builder. For more complex queries, SQL mode can be used when the user has permission.

In clients that support **MCP Apps**, numerical queries can also open an interactive visualization with a table, metric, or charts compatible with the result. QANode offers only formats suitable for the returned columns and keeps the response available as text or structured data when the client does not support this interface.

---

## Visual Run Monitoring

Clients that support **MCP Apps** can display a visual inspector when a scenario or suite run is started or queried through MCP.

The inspector brings together:

- status and duration;
- scenario and node progress;
- labels defined on the canvas;
- main logs in collapsible sections;
- the failure and message from the affected point;
- evidence from the current run;
- the longest durations in the run.

For suites, nodes are grouped by scenario to preserve context in both sequential and parallel runs. While a run is in progress, the client can refresh the view until the final result. If the client does not support MCP Apps, the same data remains available as text or structured data.

---

## WebMCP And Initial Browser State

When recording a Smart Web Flow on a compatible Chromium page, the client can list the WebMCP tools declared by the website, call a tool with its defined parameters, and assert whether that capability is available in the current page state. The call runs during recording, and its structured result is available in the `webmcpResult` output after scenario execution.

MCP can also configure cookies, `localStorage`, `sessionStorage`, session headers, and initialization JavaScript before the first navigation. This setup is optional and should be used only when the user explicitly requests an initial state; it does not replace regular interface interaction steps.

Use protected variables for secrets and environment-specific values. See [Smart Web Flow](../nodes/web/smart-web-flow.md) for field and step details.

---

## Bugs

With MCP, the user can investigate and follow up on bugs from the connected client.

Examples:

- open a bug from a failed run;
- analyze logs and evidence from the original run;
- download bug attachments;
- comment when needed;
- transition to a queue or user, when allowed;
- open a sandbox to investigate without affecting the official scenario.

The diagnosis returns to the connected client. It does not automatically become a public bug comment.

---

## Best Practices

- Use different tokens for CI/CD and MCP when you want separate audit trails.
- Revoke tokens that are no longer in use.
- Prefer saved credentials instead of pasting passwords into prompts.
- Define explicit validations when the goal is a test scenario.
- Review automation-created scenarios before using them in critical suites.
- Use role-based dashboards when the panel contains team-specific data.
- For bugs, review the diagnosis before transitioning or commenting.

---

## Next Steps

- [Integration Tokens](../ci-cd/integration-tokens.md) — Generate and govern tokens used by CI/CD and MCP
- [Smart Web Flow](../nodes/web/smart-web-flow.md) — Understand recording, locators, and target reference
- [Mobile Flow](../nodes/mobile/mobile-flow.md) — Understand mobile recording with Appium
- [Dashboard - Enterprise](../dashboard/enterprise-dashboard.md) — See panels, widgets, and SQL
- [Bugs - Overview](../bugs/overview.md) — See workflow, sandbox, and attachments
