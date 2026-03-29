# Integration Tokens

> **Available in:** QANode Enterprise

Integration tokens are dedicated credentials for automation. They allow pipelines and scripts to call the QANode CLI or API without relying on a browser login session.

---

## Where to find them

Go to:

**Settings → Access Tokens**

This screen is only visible to users with the required permissions.

---

## Permissions

| Permission | What it allows |
|------------|----------------|
| `settings.integration_token` | Create and revoke your own tokens |
| `settings.integration_token_all` | View all tokens, revoke tokens created by other people, and control the global expiration policy |

In the default product setup:

- **Super Admin** and **Admin** usually have full access
- **Architect** may have access to create their own tokens depending on the configured role

> **Important:** A token inherits the operational capabilities of the user who uses it. In other words, having a token does not grant extra access by itself; it only provides non-interactive authentication.

---

## Creating a token

1. Go to **Settings → Access Tokens**
2. Click **Generate Token**
3. Define:
   - the token **name**
   - the **expiration period**, when the global policy allows individual choice
4. Copy the generated value
5. Store that value in a pipeline **secret**

The full token is shown only at creation time. After that, the interface keeps only reference and audit information.

---

## Token prefix

QANode integration tokens use the prefix:

```bash
qnt_
```

This makes them easier to identify in logs, environment variables, and secret stores.

---

## Global expiration policy

Users with `settings.integration_token_all` can configure the global policy in:

**Settings → Access Tokens → Global Token Expiration Policy**

The available options are:

- **Each user chooses** — each person decides whether the token expires and after how long
- **Fixed expiration** — every new token expires after the same period defined by administration

When the global policy is fixed, users no longer choose an individual expiration in the creation modal.

---

## Revocation

Revoking a token immediately stops it from being used in new CLI commands or API calls.

You can revoke:

- your own tokens with `settings.integration_token`
- any token in the instance with `settings.integration_token_all`

---

## Security best practices

- Store `QANODE_TOKEN` in your CI/CD provider's **Secrets**
- Never commit tokens to versioned files
- Prefer one token per pipeline or integrated system
- Revoke tokens that are no longer used
- Use expiration when your company requires periodic rotation

---

## Example with a secret

```bash
export QANODE_URL=https://qanode.company.com
export QANODE_TOKEN=qnt_xxxxx
```

In GitHub Actions or Azure DevOps, this value is usually stored as a job secret rather than plain text in the repository.

---

## Audit trail

QANode records events related to integration tokens, such as:

- creation
- revocation
- updates to the global expiration policy

This helps track who created or revoked each operational key.

---

## Next Steps

- [CI/CD CLI and API](./cli-api.md) — See how to authenticate, trigger executions, and inspect results
