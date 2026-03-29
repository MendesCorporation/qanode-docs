# CI/CD Integration — Overview

> **Available in:** QANode Enterprise

QANode's CI/CD integration allows your company to trigger scenarios and suites from the pipeline, authenticate with integration tokens, and download the consolidated execution report without relying on a browser user session.

Instead of treating QANode as only a visual interface, your company can use it as an official part of build validation, pull request checks, release gates, or deployment pipelines.

---

## What the integration provides

- **Integration tokens** for pipeline and automation authentication
- **Official CLI (`@qanode/cli`)** for GitHub Actions, Azure DevOps, GitLab, Jenkins, and local scripts
- **Dedicated `/api/ci` routes** for scenarios, suites, run lookup, and report download
- **Execution by ID or by name** with project support when needed
- **Per-execution overrides** for variables and credentials
- **Pipeline-friendly output** with `success`, `failed`, or `cancelled`
- **Consolidated PDF report** to attach to the CI job

---

## High-level flow

The most common operational flow is:

1. The team creates an **integration token** in **Settings → Access Tokens**
2. The pipeline stores that token as a **secret**
3. The job runs the QANode CLI with `QANODE_URL` and `QANODE_TOKEN`
4. QANode creates a real execution in the instance
5. The pipeline waits for the final result and decides whether to continue or fail
6. Optionally, the job downloads the **consolidated PDF report**

---

## Main concepts

### Public QANode URL

`QANODE_URL` should point to the **public URL** your team already uses to open QANode in the browser.

In most corporate environments this means:

- the same public frontend domain
- with a proxy or load balancer forwarding `/api` to the QANode API

Examples:

```bash
https://qanode.company.com
https://qa.company.com/qanode
```

In local development, `http://localhost:3000` may also work when the frontend proxies `/api`.

### Integration token

This is a token created specifically for automation. It replaces human login in the pipeline and respects the permissions of the user or role associated with it.

### Scenario

It is an individual QANode automation made of the flow steps that validate a specific application behavior. In CI/CD, you can run a single scenario when you want to validate a critical path without executing the full suite.

### Suite

A suite groups multiple scenarios and is the most common resource for full regression, smoke tests, and release gates.

### Per-execution override

Overrides let you change variable values or credential fields **only for that pipeline execution**, without overwriting the persisted value in QANode.

---

## Required permissions

The main CI/CD permissions are:

| Permission | What it allows |
|------------|----------------|
| `settings.integration_token` | Create and revoke your own integration tokens |
| `settings.integration_token_all` | View all tokens, revoke tokens created by other people, and define the global expiration policy |
| `flow.run` | Execute scenarios through CLI/API |
| `suite.run` | Execute suites through CLI/API |
| `run.view` / `run.view.all` | Inspect executions and download the report |

Token permissions are configured in **Settings → Roles**.

---

## What the integration does not do automatically

Some behaviors are important to understand up front:

- the CLI **does not discover the token by itself**; it must be provided by the pipeline
- the CLI **does not need a browser session**
- the execution triggered by the pipeline is a **real QANode run**
- variable and credential overrides **do not permanently change** stored values
- the pipeline should reuse the **same run ID** when downloading the report instead of starting a second execution

---

## Section navigation

| Page | What you will learn |
|------|---------------------|
| [Integration Tokens](./integration-tokens.md) | How to generate, revoke, and govern pipeline tokens |
| [CI/CD CLI and API](./cli-api.md) | How to authenticate, trigger scenarios/suites, and use overrides |
| [Pipeline Examples](./pipeline-examples.md) | How to integrate with GitHub Actions and Azure DevOps |
| [Per-execution Overrides](./overrides.md) | How to override variables and credentials safely |

---

## Next Steps

- [Integration Tokens](./integration-tokens.md) — Learn how to generate, revoke, and govern the keys used by the pipeline
