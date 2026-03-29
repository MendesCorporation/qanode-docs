# Pipeline Examples

> **Available in:** QANode Enterprise

This page provides ready-to-use examples for the most common CI/CD providers.

---

## GitHub Actions

Complete example running a suite, waiting for the result, and downloading the consolidated PDF report.

```yaml
name: Run QANode Suite

on:
  workflow_dispatch:
  push:
    branches: [main, master]

jobs:
  qanode:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Run QANode suite
        id: run_suite
        run: |
          set +e
          RUN_JSON=$(npx @qanode/cli run suite \
            --project-name "Backoffice" \
            --suite-name "Login Regression" \
            --wait \
            --timeout 300 \
            --json)
          EXIT_CODE=$?
          set -e

          echo "$RUN_JSON" > qanode-run.json
          echo "run_id=$(echo "$RUN_JSON" | jq -r '.id')" >> "$GITHUB_OUTPUT"
          exit $EXIT_CODE
        env:
          QANODE_URL: ${{ secrets.QANODE_URL }}
          QANODE_TOKEN: ${{ secrets.QANODE_TOKEN }}

      - name: Download QANode report
        if: always()
        run: |
          npx @qanode/cli runs artifacts \
            --run-id "${{ steps.run_suite.outputs.run_id }}" \
            --out ./artifacts
        env:
          QANODE_URL: ${{ secrets.QANODE_URL }}
          QANODE_TOKEN: ${{ secrets.QANODE_TOKEN }}

      - name: Upload report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: qanode-report
          path: ./artifacts/
          if-no-files-found: ignore
```

### Why capture `run_id` this way

This pattern avoids a common mistake: triggering the suite twice.

The correct flow is:

1. start the run
2. wait for completion
3. extract the `id` from the same run returned in JSON
4. download the report using that `id`

---

## Azure DevOps

Equivalent example in Azure DevOps:

```yaml
steps:
- task: NodeTool@0
  inputs:
    versionSpec: '20.x'
  displayName: Setup Node.js

- script: |
    set +e
    RUN_JSON=$(npx @qanode/cli run suite \
      --project-name "Backoffice" \
      --suite-name "Login Regression" \
      --wait \
      --timeout 300 \
      --json)
    EXIT_CODE=$?
    set -e

    echo "$RUN_JSON" > qanode-run.json
    echo "##vso[task.setvariable variable=QANODE_RUN_ID]$(echo "$RUN_JSON" | jq -r '.id')"
    exit $EXIT_CODE
  displayName: Run QANode suite
  env:
    QANODE_URL: $(QANODE_URL)
    QANODE_TOKEN: $(QANODE_TOKEN)

- script: |
    npx @qanode/cli runs artifacts \
      --run-id "$(QANODE_RUN_ID)" \
      --out ./artifacts
  displayName: Download QANode report
  condition: always()
  env:
    QANODE_URL: $(QANODE_URL)
    QANODE_TOKEN: $(QANODE_TOKEN)
```

---

## Example with a variable override

Use this pattern when the application under test is published dynamically during the job:

```bash
npx @qanode/cli run suite \
  --project-name "Task Manager" \
  --suite-name "Task Manager Suite" \
  --var base_url_ci=https://build-preview.example.com \
  --wait
```

---

## Example with a credential override

Use this when the pipeline receives a temporary secret:

```bash
npx @qanode/cli run scenario \
  --project-name "Checkout" \
  --scenario-name "Login API" \
  --credential api-main.token=$API_TOKEN \
  --wait
```

---

## Consolidated report

The command:

```bash
npx @qanode/cli runs artifacts --run-id RUN_ID --out ./artifacts
```

prioritizes the consolidated execution PDF and saves it with a unique name:

```bash
report_<runId>.pdf
```

This file is the best option to:

- attach to the CI job
- audit the execution result
- send to another system

---

## Operational recommendations

- Store `QANODE_URL` and `QANODE_TOKEN` in **Secrets**
- Use `--wait` when you want the pipeline to fail together with the execution
- Use `--json` when you need to capture the run ID or keep extra execution metadata
- Use `--timeout` to avoid jobs hanging too long
- Always reuse the same `run_id` when downloading the report
