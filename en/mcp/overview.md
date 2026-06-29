# MCP — AI Integration

> **Available in:** QANode Enterprise

**MCP** connects AI tools to QANode so users can request actions from an IDE, corporate chat, or any compatible client.

With this integration, an AI client can operate QANode with the permissions of the authenticated user: create scenarios, adjust flows, inspect runs, read project documentation, create dashboards, investigate failures, work with bugs, and use the platform's visual nodes.

---

## When To Use

Use MCP when you want to:

- ask an AI assistant to create or adjust QANode scenarios;
- automate flows that involve web, API, database, files, SSH, or mobile;
- investigate an existing scenario that is failing;
- turn project documentation into initial scenarios;
- create dashboards or ask questions about QANode metrics in natural language;
- open or analyze bugs while working in the code.

MCP does not replace the visual editor. Everything created through MCP still appears in QANode as a normal project, scenario, node, run, dashboard, suite, or bug.

---

## How To Connect

1. Generate a token in **Settings → Access Tokens**
2. Configure the MCP client used by your AI tool
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

The exact format depends on the AI client, but the URL and header are the same.

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

The token does not grant extra access. It only authenticates the AI client as that user or integration.

---

## What You Can Request

| Area | Example uses |
|------|--------------|
| **Projects** | Search projects, search snippets inside documents, and read attachments when needed |
| **Scenarios** | Create flows, import nodes, add steps, connect, validate, and execute |
| **Smart Web** | Record web navigation, validate screens, extract data, and reuse sessions when needed |
| **Mobile** | Record Android/iOS journeys with Appium and save them as Mobile Flow nodes |
| **Database** | Inspect schema and create database queries or validations |
| **Files** | Generate, extract, and reuse files in flows |
| **Variables** | Create global or project variables for reuse |
| **Credentials** | Create credentials when the user provides the required data |
| **Suites** | Create, edit, execute, schedule, and diagnose failures |
| **Bugs** | Open, comment, attach files, transition, and investigate bugs |
| **Dashboards** | Create panels, widgets, and quick queries over QANode data |
| **Components** | Create reusable components when the user explicitly asks for one |

---

## How To Ask

Clearer requests produce better results. Include:

- the project where the scenario should be created;
- the target URL or system;
- credentials or the name of an existing credential;
- allowed test data;
- expected result;
- cleanup rules, such as deleting created test data at the end;
- whether the flow should only be created or also executed.

Example:

```text
In the Task Manager project, create a scenario that opens localhost:3008,
logs in with the test user, creates a task, marks it as completed,
and validates in the database that the status is completed.
```

When important information is missing, confirm the data before creating records, changing real data, or executing destructive actions.

---

## Smart Web And Target Reference

On pages with tables, cards, kanban boards, or repeated lists, the same button or status can appear several times on the screen.

In these cases, use the Smart Web **Target Reference**. It tells QANode which row, card, or item should be used as the reference before clicking, validating, or extracting a value.

Example:

```text
Validate that the status is Paid on the order created in this run,
not on any old order shown on the page.
```

This helps QANode act on the correct record even when the data changes on every run.

---

## Web And Mobile Sessions

When a scenario is split across more than one Smart Web or Mobile node, state whether the nodes need to continue in the same session.

Use this when:

- the first node logs in and the second continues authenticated;
- a purchase or registration starts on one screen and finishes on another;
- the mobile app needs to keep the same state across parts of the flow.

For already authenticated web sessions, such as enterprise systems with SSO, use named persisted sessions. For common flows that log in during the run itself, a temporary session per run is usually enough.

---

## Project Documents

MCP lets AI clients use project documents as context for creating scenarios, answering questions, or understanding business rules.

For large documents, the AI client can first search for relevant snippets inside the project attachments. This helps it find pages, sections, or parts of the text before opening the full document.

QANode can read:

- text-based PDFs;
- scanned PDFs or images with OCR;
- DOCX;
- TXT, JSON, and CSV;
- XLSX with spreadsheet preview;
- metadata from other files.

When the snippet is not enough, ask the AI client to read a page, a page range, or the full document.

---

## Official QANode Documentation

In addition to your project documents, MCP can also search the official QANode documentation.

Use this when you want to ask how to configure a node, understand a feature, or confirm the behavior of a product area without leaving the AI client.

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

You can ask an AI client to create dashboards or answer one-off questions about QANode data.

Examples:

```text
How many runs did we have in May?
```

```text
Create a public dashboard with total runs this month,
success rate, recent failures, and daily evolution.
```

For simple widgets, QANode uses the visual query builder. For more complex queries, SQL mode can be used when the user has permission.

---

## Bugs

With MCP, an AI client can help investigate and follow up on bugs.

Examples:

- open a bug from a failed run;
- analyze logs and evidence from the original run;
- download bug attachments;
- comment when the user asks;
- transition to a queue or user, when allowed;
- open a sandbox to investigate without affecting the official scenario.

The diagnosis returns in the chat. It does not automatically become a public bug comment.

---

## Best Practices

- Use different tokens for CI/CD and MCP when you want separate audit trails.
- Revoke tokens that are no longer in use.
- Prefer saved credentials instead of pasting passwords into prompts.
- Request explicit validations when the goal is a test scenario.
- Review automation-created scenarios before using them in critical suites.
- Use role-based dashboards when the panel contains team-specific data.
- For bugs, ask for the diagnosis before transitioning or commenting.

---

## Next Steps

- [Integration Tokens](../ci-cd/integration-tokens.md) — Generate and govern tokens used by CI/CD and MCP
- [Smart Web Flow](../nodes/web/smart-web-flow.md) — Understand recording, locators, and target reference
- [Mobile Flow](../nodes/mobile/mobile-flow.md) — Understand mobile recording with Appium
- [Dashboard - Enterprise](../dashboard/enterprise-dashboard.md) — See panels, widgets, and SQL
- [Bugs - Overview](../bugs/overview.md) — See workflow, sandbox, and attachments
