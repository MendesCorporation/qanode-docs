# CI/CD CLI and API

> **Available in:** QANode Enterprise

This page shows how to use the official QANode CLI and the dedicated CI/CD routes to integrate the product with enterprise pipelines.

---

## Official CLI

The official package is:

```bash
@qanode/cli
```

You can use it without a global install:

```bash
npx @qanode/cli help
```

Or install it as a development dependency:

```bash
npm install --save-dev @qanode/cli
```

Then run:

```bash
npx qanode help
```

> **Recommendation:** In pipelines, `npx @qanode/cli ...` is usually enough and avoids depending on a preinstalled global package.

---

## Environment variables

The two main variables are:

| Variable | Description |
|----------|-------------|
| `QANODE_URL` | Public QANode URL |
| `QANODE_TOKEN` | Integration token or session token |

Example:

```bash
export QANODE_URL=https://qanode.company.com
export QANODE_TOKEN=qnt_xxxxx
```

### About the URL

`QANODE_URL` should be the same public address your team already uses in the browser.

There is no need to know the internal API port when your company uses:

- a reverse proxy
- a load balancer
- a single public domain for frontend + backend

In local development, `http://localhost:3000` may work when the frontend forwards `/api`.

---

## Main commands

### Validate authentication

```bash
npx @qanode/cli auth me
```

### Run a scenario by ID

```bash
npx @qanode/cli run scenario --scenario-id SCENARIO_ID --wait
```

### Run a scenario by project and name

```bash
npx @qanode/cli run scenario \
  --project-name "Checkout" \
  --scenario-name "Login API" \
  --wait
```

### Run a suite by ID

```bash
npx @qanode/cli run suite --suite-id SUITE_ID --wait
```

### Run a suite by project and name

```bash
npx @qanode/cli run suite \
  --project-name "Backoffice" \
  --suite-name "Login Regression" \
  --wait
```

### Inspect a run

```bash
npx @qanode/cli runs get --run-id RUN_ID
```

### Wait for an existing run

```bash
npx @qanode/cli runs wait --run-id RUN_ID --timeout 300
```

### Download the consolidated report

```bash
npx @qanode/cli runs artifacts --run-id RUN_ID --out ./artifacts
```

The command prioritizes the consolidated PDF report and saves it as:

```bash
report_<runId>.pdf
```

---

## CLI output

Without `--json`, commands that return a final result print only:

```bash
success
failed
cancelled
```

This is intentional to make pipeline usage simpler.

With `--json`, the CLI prints the full API payload.

### Important difference between `run ... --json` and `run ... --wait --json`

Without `--wait`:

```bash
npx @qanode/cli run suite --suite-id SUITE_ID --json
```

returns the initial run creation payload, which contains `runId`.

With `--wait`:

```bash
npx @qanode/cli run suite --suite-id SUITE_ID --wait --json
```

returns the final run object, which contains `id`.

> **Important:** To download the report, reuse the ID from the same execution you already started. Do not trigger a second run just to discover the identifier.

---

## Dedicated CI/CD routes

In addition to the CLI, QANode exposes routes specifically designed for automation:

| Method | Route | Use |
|--------|-------|-----|
| `POST` | `/api/ci/runs/scenario` | Starts a scenario execution |
| `POST` | `/api/ci/runs/suite` | Starts a suite execution |
| `GET` | `/api/ci/runs/:id` | Reads the current status and final result |

These routes use the same token provided in `QANODE_TOKEN`.

---

## Execution by ID or by name

You can identify the target in two ways:

### By ID

More precise and ideal for stable automations.

```bash
--scenario-id
--suite-id
--project-id
```

### By name

More human-friendly.

```bash
--project-name
--scenario-name
--suite-name
```

When using names, prefer also sending the project to avoid ambiguities.

---

## Timeouts and polling

The main automation flags are:

| Flag | Use |
|------|-----|
| `--wait` | Waits for execution completion |
| `--timeout` | Defines the maximum wait time in seconds |
| `--interval` | Polling interval between status checks |

Example:

```bash
npx @qanode/cli run suite \
  --suite-id SUITE_ID \
  --wait \
  --interval 5 \
  --timeout 600
```

---

## Exit codes

The CLI process returns:

| Code | Meaning |
|------|---------|
| `0` | Execution completed successfully |
| `2` | Execution completed with failure or cancellation |
| `1` | CLI, authentication, or API error |

This allows the CLI to be used as a pipeline gate without complex output parsing.

---

## Next Steps

- [Per-execution Overrides](./overrides.md) — Learn how to override variables and credentials only for the current run
