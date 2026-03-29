# Per-execution Overrides

> **Available in:** QANode Enterprise

Overrides let you change variable values and credential fields only during the current pipeline execution, without modifying the permanent configuration stored in QANode.

---

## When to use them

Overrides are useful when your company needs to:

- point to a temporary PR environment
- change an API base URL
- use an ephemeral pipeline token
- switch to a temporary staging password
- execute the same suite against multiple environments without editing the flow

---

## Variable override

Use:

```bash
--var NAME=VALUE
```

Examples:

```bash
--var BASE_URL=https://preview.app
--var LOCALE=en-US
--var FEATURE_FLAG=true
```

Complete example:

```bash
npx @qanode/cli run scenario \
  --project-name "Checkout" \
  --scenario-name "Login API" \
  --var BASE_URL=https://preview.app \
  --wait
```

---

## Credential override

Use:

```bash
--credential CREDENTIAL.FIELD=VALUE
```

Examples:

```bash
--credential api-main.token=$API_TOKEN
--credential smtp-main.password=$SMTP_PASSWORD
--credential pg-staging.host=db-preview.internal
```

Complete example:

```bash
npx @qanode/cli run scenario \
  --scenario-id SCENARIO_ID \
  --credential api-main.token=$API_TOKEN \
  --credential api-main.baseUrl=https://preview.api \
  --wait
```

---

## Override behavior

Overrides:

- apply only to the current execution
- do not overwrite the value stored in the database
- do not create a new variable or credential record

In other words, QANode uses the override value only for that run.

---

## Relationship with saved variables and credentials

The persisted value remains the official reference in the product:

- [Variables](../variables/variables.md)
- [Credentials](../credentials/credentials.md)

The override is a temporary layer on top of that configuration.

---

## Best practices

- Use overrides only for values that actually change by environment or pipeline
- Store secrets in your CI/CD provider's **Secrets**
- Prefer full URLs, including `https://`, when the scenario navigates directly to a page
- If the target is identified by name, also provide the project

---

## What does not change automatically

Today the override is applied to the execution triggered by the pipeline. By itself, it does not imply that other derived flows automatically inherit the same context.

For that reason, treat the override as part of the CI/CD execution context, not as a permanent scenario change.

---

## Next Steps

- [Pipeline Examples](./pipeline-examples.md) — See complete implementations for GitHub Actions and Azure DevOps
