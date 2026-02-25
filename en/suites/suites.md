# Test Suites

**Suites** allow you to group multiple scenarios (flows) for sequential or non-sequential execution. They are ideal for regression testing, smoke tests, and scheduled executions.

---

## Creating a Suite

1. In the sidebar, click **Suites**
2. Click **+ New Suite**
3. Fill in:

| Field | Required | Description |
|-------|----------|-------------|
| **Name** | Yes | Suite name |
| **Project** | Yes | Project the suite belongs to |
| **Scenarios** | Yes | Select the scenarios and their order |
| **Stop on Failure** | No | Stop at the first failing scenario |
| **Execution Mode** | No | Sequential (default) or Non-sequential |

4. Click **Create**

<!-- ![Create suite](../../assets/images/criar-suite.png) -->
*Image: Suite creation form with scenario selection and stop-on-failure toggle*

---

## Execution Mode

The suite offers two execution modes:

### Sequential (default)

Scenarios are executed **one at a time**, in the defined order. You can reorder them by dragging the scenarios in the list.

```
Suite: Login Regression (Sequential)
├── 1. Login with valid credentials    ⏳ waiting
├── 2. Login with incorrect password   ⏳ waiting
├── 3. Login with invalid email        ⏳ waiting
├── 4. Password recovery               ⏳ waiting
└── 5. Logout                          ⏳ waiting
→ Executes: 1, then 2, then 3...
```

Use sequential mode when:
- Scenarios **depend on each other** (e.g., login before operations)
- You need a **guaranteed execution order**
- You want to use the **Stop on Failure** option to stop at the first failure

### Non-sequential

Scenarios are executed **independently**, without a guaranteed order. All scenarios are triggered and executed in parallel.

```
Suite: Login Regression (Non-sequential)
├── 1. Login with valid credentials    ▶ running
├── 2. Login with incorrect password   ▶ running
├── 3. Login with invalid email        ▶ running
├── 4. Password recovery               ▶ running
└── 5. Logout                          ▶ running
→ All execute at the same time
```

Use non-sequential mode when:
- Scenarios are **independent** of each other
- You want to **reduce the total execution time** of the suite
- Execution order **does not matter**

---

## Stop on Failure

> **Note:** The "Stop on Failure" option only takes effect in **Sequential** mode. In non-sequential mode, since all scenarios are executed independently, this option does not apply.

When **Stop on Failure** is enabled:
- If a scenario fails, the following scenarios **are not executed**
- Useful when scenarios depend on each other (e.g., login before operations)

When disabled:
- All scenarios are executed regardless of failures
- Useful for full regression suites where you want to see all failures

---

## Running a Suite

### Manual Execution

1. In the suite list, click the **Execute** button (▶️)
2. All scenarios will be executed in order
3. The result appears in the execution list

### Scheduled Execution

Suites can be executed automatically at defined times.

---

## Scheduling

### Configuring Scheduling

1. Edit the suite
2. In the **Scheduling** section, enable the toggle
3. Enter the **cron expression**
4. The system will show the next 5 execution times

[Suite scheduling](../../assets/images/suites.mp4)
*Image: Scheduling section with cron field, activation toggle, and preview of upcoming times*

### Cron Expressions

The cron format has 5 fields:

```
┌───────────── minute (0-59)
│ ┌─────────── hour (0-23)
│ │ ┌───────── day of month (1-31)
│ │ │ ┌─────── month (1-12)
│ │ │ │ ┌───── day of week (0-7, 0 and 7 = Sunday)
│ │ │ │ │
* * * * *
```

### Common Examples

| Expression | Meaning |
|------------|---------|
| `0 8 * * *` | Every day at 8:00 |
| `0 8 * * 1-5` | Monday to Friday at 8:00 |
| `0 */2 * * *` | Every 2 hours |
| `30 9 * * 1` | Every Monday at 9:30 |
| `0 0 * * *` | Every day at midnight |
| `0 6,12,18 * * *` | At 6am, 12pm, and 6pm |
| `*/15 * * * *` | Every 15 minutes |
| `0 9 1 * *` | First day of the month at 9:00 |

### Enable/Disable

You can enable and disable scheduling without removing the cron expression. When disabled, the expression is preserved for future reactivation.

---

## Suite Results

After execution, the suite displays:

| Information | Description |
|-------------|-------------|
| **Overall Status** | Success (all passed) or Failure (some failed) |
| **Executed Scenarios** | How many were processed |
| **Successful Scenarios** | How many passed |
| **Failed Scenarios** | How many failed |
| **Total Duration** | Total execution time |

Each individual scenario also has its detailed result (logs, screenshots, etc.).

---

## Alarms

You can configure **alarms** to be notified when suites fail. See the [Administration](../administration/enterprise-administration.md) section to configure alarms.

---

## Tips

- **Order strategically** — in sequential mode, place setup scenarios (login, preparation) first
- **Use "Stop on Failure"** when scenarios depend on each other (sequential mode)
- **Disable "Stop on Failure"** for full regression suites
- **Use non-sequential mode** for regression suites with independent scenarios — reduces total time
- **Schedule during off-peak hours** — avoid conflict with manual usage
- **Monitor with alarms** — configure notifications for failures in scheduled suites - Enterprise version
