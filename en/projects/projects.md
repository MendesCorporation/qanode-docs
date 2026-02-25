# Projects

**Projects** are the main organizational unit in QANode. Each project groups test scenarios (flows), suites, documentation, and executions related to a system, feature, or team.

---

## Creating a Project

1. In the sidebar, click **Projects**
2. Click **+ New Project**
3. Fill in the fields:

| Field | Required | Description |
|-------|----------|-------------|
| **Name** | Yes | Project name |
| **Description** | No | Detailed description |
| **Status** | Yes | Current project status |
| **Start Date** | No | Planned start date |
| **End Date** | No | Planned completion date |

4. Click **Create**

<!-- ![Create project](../../assets/images/criar-projeto.png) -->
*Image: Project creation modal with all fields filled in*

---

## Project Status

| Status | Description |
|--------|-------------|
| **Active** | Project in progress |
| **Completed** | Project successfully finished |
| **Cancelled** | Project cancelled |
| **Archived** | Project archived (does not appear in the main list) |

To change the status, open the project and use the options in the header:
- **Complete** → Marks as completed
- **Cancel** → Marks as cancelled
- **Archive** → Moves to archived

---

## Project Page

The project detail page has the following tabs:

### Overview

Displays statistics and a summary of the project:

- **Total Scenarios** — Number of test flows
- **Total Suites** — Number of suites
- **Pending/Approved/Failed Scenarios** — Scenario statuses
- **Recent Executions** — List of the latest executions
- **Progress** — Comparison between elapsed time and executed scenarios

<!-- ![Project overview](../../assets/images/projeto-visao-geral.png) -->
*Image: Overview page showing statistics cards, recent executions, and progress bar*

### Scenarios

Lists all scenarios (flows) in the project:

| Information | Description |
|-------------|-------------|
| **Name** | Scenario name |
| **Status** | Last result (pending, success, failure) |
| **Creation Date** | When it was created |

**Actions:**
- **+ New Scenario** — Creates a new flow and opens the editor
- **Execute** — Runs the scenario quickly
- **Edit** — Opens the flow editor
- **Delete** — Removes the scenario

### Suites

Lists the project suites with scheduling information and last result.

> For details about suites, see [Test Suites](../suites/suites.md).

### Executions

History of all project executions:

| Information | Description |
|-------------|-------------|
| **Scenario/Suite** | Name of what was executed |
| **Status** | Result (success, failure, running) |
| **Duration** | Execution time |
| **Date** | Date and time of execution |

### Documentation

Project file management:

- **Upload** — Upload documents (requirements, specifications, etc.)
- **Download** — Download attached documents
- **Delete** — Remove documents

---

## Filtering and Search

In the project list, you can:

- **Search by name** — Search field at the top
- **Filter by status** — Filter dropdown (Active, Completed, etc.)

---

## Duplicate Project

To create a copy of an existing project:

1. In the project list, click the options menu (⋮) for the project
2. Select **Duplicate**
3. A new project will be created with the same scenarios

---

## Tips

- Use **status** to track the project lifecycle
- **Archive** old projects to keep the list clean
- **Start and end dates** help track progress vs. planning
- Use the **Documentation** tab to keep requirements and specifications alongside the project
