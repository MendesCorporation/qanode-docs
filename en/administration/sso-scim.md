# SSO and User Provisioning

QANode can centralize authentication in an identity provider (IdP) through **OpenID Connect (OIDC)** or **SAML 2.0**. The user lifecycle can be handled by **JIT** (creation on first sign-in) or **SCIM 2.0** (creation, activation, and deactivation sent by the IdP).

This guide explains what the IdP controls, what remains under QANode control, and how to configure the most common providers.

> **Availability:** only the Super Admin can configure this feature, under **Settings → General → Single sign-on**.

---

## Overview

SSO and SCIM solve different problems:

| Feature | Responsibility | What it does |
|---------|----------------|--------------|
| **OIDC or SAML** | Authentication | Confirms the identity and lets the user enter QANode with a corporate account |
| **JIT** | Provisioning during sign-in | Creates the QANode user on their first sign-in |
| **SCIM** | User lifecycle | Creates, activates, and deactivates QANode users from the IdP |
| **QANode roles** | Authorization | Defines which projects, pages, and actions the user can access |

In the current version, the IdP **does not control roles or permissions**. QANode receives identity and user status, but remains the source of truth for authorization.

```text
IdP (authentication and lifecycle)
       │
       ├── OIDC or SAML ──> identity and sign-in
       └── SCIM ──────────> create, activate, and deactivate
                                  │
                                  ▼
QANode (role, permissions, projects, and resources)
```

### Choosing a combination

| Scenario | Recommendation |
|----------|----------------|
| The provider supports OIDC | Use **OIDC** for authentication |
| The organization requires SAML | Use **SAML 2.0** |
| The provider supports SCIM | Use **SCIM** for provisioning |
| The provider has no SCIM support | Use **JIT** with a short revalidation interval |
| Fast deactivation is mandatory | Use **SCIM**; JIT alone does not synchronize deactivation to the local record |

---

## Before You Begin

Check the following before configuring the provider:

1. QANode must be published at an **HTTPS** URL reachable by the IdP. Do not use `localhost` in production.
2. Confirm that the Super Admin knows the local password and that their QANode email exactly matches the email returned by the IdP.
3. Choose a low-privilege **default role** for new users. Super Admin is never available as the default.
4. Decide whether provisioning will use **JIT** or **SCIM**.
5. In the IdP, grant application access only to the users or groups that should enter QANode.
6. Store the OIDC secret and SCIM token in a secrets vault.
7. Run the test from a Super Admin session before enabling SSO for the entire instance.

> QANode generates callback, ACS, and metadata URLs from its public URL. If the domain, protocol, or port is wrong, the provider will reject the return request.

---

## General QANode Configuration

Under **Settings → General → Single sign-on**, complete the common fields:

| Field | Description |
|-------|-------------|
| **Protocol** | `OpenID Connect (OIDC)` or `SAML 2.0` |
| **Provider name** | Name shown to administrators, such as `Microsoft Entra` or `Google Workspace` |
| **User provisioning** | `JIT — Create on first sign-in` or `SCIM — Managed by the IdP` |
| **Default role** | Role assigned to a user created by JIT or SCIM |
| **Revalidate SSO every** | Time between new IdP validations: 30 minutes, 1, 4, 8, or 24 hours |
| **Super Admin emergency local access** | Allows the Super Admin to keep signing in with the local password |

### Default role and permissions

- A new user receives the selected **default role**.
- The IdP does not send or replace roles in the current version.
- An administrator can change the role later under **Settings → Users**.
- A later synchronization does not undo that change.
- The **Super Admin** role cannot be assigned through JIT or SCIM.
- Apply least privilege: start with `Tester` or a restricted custom role and grant more permissions only when needed.

### Super Admin linking

The first test links the local Super Admin account to the external identity. The email returned by the IdP must be the same email registered in QANode, ignoring only uppercase/lowercase differences.

After the first link, QANode also uses the provider's stable identifier (`sub` in OIDC or `NameID` in SAML). Do not configure a temporary identifier or one that changes on every sign-in.

### Emergency local access

When enabled, only the Super Admin can continue using a local password. This option is intended for administrative recovery when the IdP is unavailable or the SSO configuration must be corrected.

Recommendations:

- store the password in a corporate vault;
- limit who can retrieve it;
- test the emergency procedure periodically;
- disable the option if company policy forbids local authentication.

---

## JIT Provisioning

In **JIT** mode, a user is created in QANode when they complete their first SSO sign-in.

### First sign-in

1. The user starts sign-in from QANode.
2. The IdP authenticates the person and returns a stable identifier, email, and name.
3. If a QANode user with the same email already exists, QANode links the external identity to that record.
4. Otherwise, QANode creates an active user with no local password and the default role.
5. Later sign-ins use the provider's immutable identifier.

Name and email can be updated from the SSO data. The role remains controlled by QANode.

### When the user is blocked in the IdP

JIT **does not send a deactivation to the QANode database**. The record may still appear active, but the user will be unable to renew authentication after the IdP blocks access.

The maximum time before an open session returns to the IdP is controlled by **Revalidate SSO every**. To reduce the access window without SCIM, use 30 or 60 minutes.

Do not use manual QANode deactivation as the source of truth for a JIT user who is still authorized in the IdP: a later valid SSO sign-in may reactivate the record. Remove application access in the IdP first.

> If the local record must be deactivated automatically, use SCIM. JIT creates users during sign-in; it is not full lifecycle synchronization.

---

## SCIM Provisioning

In **SCIM** mode, the IdP calls the QANode SCIM 2.0 API to create, update, activate, or deactivate users.

### Values provided by QANode

| Field | Use in the provider |
|-------|---------------------|
| **SCIM base URL** | Provisioning endpoint ending in `/api/scim/v2` |
| **Secret token** | Bearer token used by the provider to authenticate calls |

The token is displayed only when it is generated. Copy it directly to the IdP and store it securely. Rotating the token immediately invalidates the previous value.

### Supported resources

QANode manages SCIM users and supports:

- user creation and lookup;
- full and partial updates;
- activation and deactivation through `active`;
- SCIM deletion treated as **deactivation**, without physically deleting the user;
- filters required to find users by identity.

The current version does not synchronize groups, roles, passwords, photos, or permissions through SCIM. Disable group mapping in the provider.

### Recommended mapping

Keep only the required attributes:

| QANode SCIM attribute | Required | Recommended IdP source | Notes |
|-----------------------|----------|------------------------|-------|
| `userName` | Yes | Real corporate email | Must be a valid, unique email; use as matching attribute with precedence 1 |
| `active` | Recommended | User active state | Controls activation and deactivation |
| `displayName` | Recommended | Display name | Name shown in QANode |
| `name.givenName` | No | Given name | Optional |
| `name.familyName` | No | Family name | Optional |
| `emails[type eq "work"].value` | No | Same email used for `userName` | Helps providers that send an email collection |
| `externalId` | Recommended | Immutable IdP object ID | Do not use a nickname that can change |

QANode requires `userName` or the primary email received to be a valid email address. A technical UPN such as `GUID@tenant.onmicrosoft.com` must not be used when it does not represent the real sign-in email.

### Deactivation and reactivation

- When the IdP sends `active: false`, QANode deactivates the user and invalidates access.
- When it sends `active: true`, the user becomes active again, subject to the license user limit.
- A SCIM `DELETE` operation also deactivates the user; it does not permanently delete their data.
- SCIM-controlled users should be activated or deactivated in the IdP, not manually in QANode.
- The Super Admin is never deactivated or managed through SCIM.

### Synchronization windows

Synchronization timing is controlled by the provider. Some providers work in cycles, so a change may take several minutes. Use the provider's on-demand provisioning feature during rollout, then wait for the next cycle to validate normal operation.

---

## Configuring OIDC

OIDC is the recommended authentication option when the provider supports it. QANode uses the **Authorization Code flow with PKCE** and requests the `openid`, `profile`, and `email` scopes.

### QANode fields

| QANode field | Where to find it in the IdP |
|--------------|-----------------------------|
| **Issuer URL** | The `issuer` value from `/.well-known/openid-configuration` |
| **Client ID** | Public ID of the OIDC application |
| **Client secret** | Application secret value; do not use the secret ID or name |
| **Token endpoint authentication** | `client_secret_post` or `client_secret_basic`, as supported by the provider |
| **Redirect URI** | Copy from QANode into the IdP callback/redirect URI list |

> The issuer URL must exactly match the provider's published `issuer` value. Do not manually add or remove slashes, paths, or version segments.

The IdP must return:

- a stable, non-empty `sub`;
- an email in the `email` claim or, as a fallback, in `preferred_username`;
- a name in `name`, when available;
- an `email_verified` value other than `false`, when that claim is present.

### Microsoft Entra ID and External ID

1. Open **App registrations** and select the QANode application.
2. Under **Authentication**, add the QANode **Redirect URI** to the **Web** platform.
3. Under **Certificates & secrets**, create a secret and copy its **Value**. The value is shown once; the **Secret ID** does not work as the Client Secret.
4. Copy the **Application (client) ID** into Client ID.
5. Open **Endpoints** and access the **OpenID Connect metadata document**.
6. Copy the exact `issuer` value from the JSON into **Issuer URL**.

In Microsoft Entra workforce tenants, the issuer usually follows this format:

```text
https://login.microsoftonline.com/{tenant-id}/v2.0
```

In Microsoft Entra External ID, it normally uses the `ciamlogin.com` domain:

```text
https://{tenant}.ciamlogin.com/{tenant-id}/v2.0
```

Always use the value published by your tenant's document. Start with `client_secret_post`; switch to `client_secret_basic` only if the provider document lists support and the change is necessary.

### Google

1. Configure the OAuth consent screen in Google Cloud Console.
2. Create an **OAuth Client ID** of type **Web application**.
3. Add the QANode URI under **Authorized redirect URIs**.
4. Copy the Client ID and Client Secret.
5. Use this issuer:

```text
https://accounts.google.com
```

Google accepts both `client_secret_post` and `client_secret_basic`. The redirect URI must match exactly, including protocol, domain, port, path, and trailing slash.

### Okta

1. Create an **OIDC — Web Application** integration.
2. Add the QANode URI under **Sign-in redirect URIs**.
3. Assign users or groups to the application.
4. Copy the Client ID and Client Secret.
5. Use the selected Authorization Server's **Issuer URI**. For the default server, it is usually:

```text
https://{your-okta-domain}/oauth2/default
```

### Keycloak

1. Create an **OpenID Connect** client.
2. Enable **Standard flow** and client authentication.
3. Add the QANode URI under **Valid redirect URIs**.
4. Copy the Client ID and the secret from the credentials tab.
5. Use this issuer:

```text
https://{keycloak-host}/realms/{realm}
```

### Auth0

1. Create a **Regular Web Application**.
2. Add the QANode URI under **Allowed Callback URLs**.
3. Copy the Client ID and Client Secret.
4. Use the `issuer` value from discovery for the tenant or custom domain:

```text
https://{your-auth0-domain}/
```

### Zoho

Create an OIDC client for a web application, register the redirect URI, and copy the Client ID and Client Secret. Use the `issuer` published by the discovery document for your organization's **data center**. Do not assume all accounts use the `.com` domain; European, Indian, and other regional organizations may publish a different domain.

---

## Configuring SAML 2.0

With SAML, QANode is the **Service Provider (SP)** and the corporate platform is the **Identity Provider (IdP)**.

### QANode values to register in the IdP

| QANode field | Common provider name |
|--------------|----------------------|
| **Login URL** | Sign-on URL, Start URL, or Login URL |
| **Issuer / Entity ID** | Identifier, Audience URI, SP Entity ID, or Client ID |
| **Assertion Consumer Service URL (ACS)** | Reply URL, ACS URL, Single sign-on URL, or Recipient URL |
| **Metadata URL** | SP Metadata URL, when the provider can import metadata |
| **Logout URL** | Local return to the login page; it is not a SAML SLO endpoint |

> In the current version, SAML sign-in must start in QANode. Opening the application directly from the IdP catalog (IdP-initiated) or using the provider's own test button may fail because there is no sign-in state created by QANode.

### IdP values to register in QANode

| QANode field | Common provider name |
|--------------|----------------------|
| **IdP SSO URL** | Login URL, SSO URL, or SAML 2.0 Endpoint |
| **IdP issuer** | IdP Entity ID, Issuer, or Identifier |
| **X.509 signing certificate** | Certificate, Signing Certificate, or Base64/PEM Certificate |
| **Email attribute** | Exact attribute name containing the email; default is `email` |
| **Name attribute** | Exact attribute name containing the name; default is `displayName` |

When the provider supplies PEM, include `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----` when pasting the certificate.

### SAML compatibility requirements

Configure the provider with these rules:

| Setting | Expected value |
|---------|----------------|
| ACS binding | HTTP POST |
| SAML assertion | **Signed** |
| SAML response | May be signed or unsigned |
| Assertion encryption | Disabled |
| QANode AuthnRequest | Unsigned; the IdP must not require a signature |
| NameID | Stable and non-empty; persistent or email format |
| Email | Sent in the configured attribute or a known email claim |

QANode searches for email in the configured attribute and then in `email`, `mail`, the LDAP email OID, or NameID when it looks like an email address. Explicitly configuring the attribute still makes the integration more predictable.

### Microsoft Entra ID — SAML

Under **Enterprise applications → Single sign-on → SAML**:

| Microsoft Entra field | QANode value |
|-----------------------|---------------|
| **Identifier (Entity ID)** | Issuer / Entity ID |
| **Reply URL (ACS)** | ACS URL |
| **Sign-on URL** | Login URL |
| **Logout URL** | Leave blank |

Configure these attributes:

| Claim | Recommended source |
|-------|--------------------|
| `email` | `user.mail` or another attribute containing the real email |
| `displayName` | `user.displayname` |

Then copy to QANode:

- **Login URL** → IdP SSO URL;
- **Microsoft Entra Identifier** → IdP issuer;
- signing certificate → X.509 certificate.

Choose an option that signs the **assertion**. Do not enable encryption and do not require a signed AuthnRequest.

### Google Workspace — SAML

Under **Apps → Web and mobile apps → Add app → Add custom SAML app**:

1. Copy the Google **SSO URL**, **Entity ID**, and certificate into the IdP fields in QANode.
2. Set **ACS URL** to the QANode ACS URL.
3. Set **Entity ID** to the QANode Issuer / Entity ID.
4. Use the primary email as NameID with the `EMAIL` format.
5. **Signed response** can remain unchecked: Google still signs the assertion, which is what QANode requires.
6. Map primary email to `email` and display name to `displayName`, when available.
7. Turn the service on for the required organizational units or groups.

Start the test from QANode, not from Google's **Test SAML login** button.

### Okta — SAML

| Okta field | Value |
|------------|-------|
| **Single sign-on URL** | QANode ACS URL |
| **Use this for Recipient URL and Destination URL** | Enabled |
| **Audience URI (SP Entity ID)** | QANode Issuer / Entity ID |
| **Default RelayState** | Blank |
| **Name ID format** | EmailAddress |
| **Application username** | Email |
| **Assertion Signature** | Signed |
| **Assertion Encryption** | Unencrypted |
| **Signed Requests** | Disabled |

Add `email = user.email` and `displayName = user.displayName`. Then copy the Identity Provider Single Sign-On URL, Issuer, and X.509 certificate into QANode.

### OneLogin — SAML

Configure:

- **ACS (Consumer) URL** and **Recipient** with the ACS URL;
- **Audience** with the QANode Entity ID;
- blank RelayState;
- email as NameID;
- signed assertion and no encryption.

Copy the **SAML 2.0 Endpoint (HTTP)**, **Issuer URL**, and X.509 certificate into QANode.

### Zoho — SAML

Create a custom SAML application and use:

- **ACS URL** = QANode ACS URL;
- **Entity ID / Audience** = QANode Issuer;
- **Start URL** = QANode Login URL, when that field is available;
- **NameID** = corporate email;
- `email` and `displayName` attributes.

Copy the SAML login URL, Zoho issuer, and platform certificate into QANode.

### JumpCloud — SAML

Set the SP Entity ID, ACS URL, and Login URL to the QANode values. Use email as NameID and select an option that signs the **assertion**, or both response and assertion. The response-only signing default is not sufficient. Keep encryption off and do not require signed requests.

### Keycloak — SAML

Create a SAML client with:

- **Client ID** = QANode Entity ID;
- **Master SAML Processing URL** and **Valid Redirect URIs** = ACS URL;
- **Client signature required** = disabled;
- assertion signing = enabled;
- assertion encryption = disabled;
- mappers for `email` and `displayName`.

Use the realm SAML URL as the IdP SSO URL, the realm issuer, and the public signing certificate in QANode.

### PingOne, Duo, and Auth0 — SAML

These providers use the same standard SAML fields:

- ACS/Recipient/Single Sign-on URL = QANode ACS URL;
- Audience/Entity ID = QANode Entity ID;
- NameID = stable email;
- signed assertion;
- encryption disabled;
- signed request not required.

Copy the provider's SSO URL, IdP Issuer/Entity ID, and X.509 certificate.

---

## SCIM in Microsoft Entra ID and External ID

Microsoft Entra configures SCIM in an **Enterprise application**. This provisioning application can be separate from the OIDC app registration.

### Creating the SCIM application

1. Open **Enterprise applications → New application**.
2. Choose **Create your own application** and the option to integrate another application not found in the gallery.
3. Open **Provisioning** and choose automatic mode.
4. Enter the QANode SCIM base URL in **Tenant URL**.
5. Enter the token generated by QANode in **Secret Token**.
6. Select **Test Connection**.
7. Under **Mappings**, keep user provisioning and disable group provisioning.
8. Assign the users or groups that should be synchronized.
9. Use **Provision on demand** to test one user.
10. After validation, select **Start provisioning**.

### Recommended minimum Entra mapping

| QANode target | Entra source | Configuration |
|---------------|--------------|---------------|
| `userName` | Attribute containing the real email | Match objects = `Yes`, precedence = `1`, apply = `Always` |
| `active` | `Not([IsSoftDeleted])` | Apply = `Always` |
| `displayName` | `displayName` | Apply = `Always` |
| `emails[type eq "work"].value` | Same email attribute | Optional |
| `externalId` | `objectId` or another exposed immutable ID | Recommended |

The computed `IsSoftDeleted` attribute becomes true when the user is disabled, removed from the application, deleted, or goes out of scope. Keep `active ← Not([IsSoftDeleted])` so these actions deactivate the QANode account.

### Email warning for External ID

In External ID tenants:

- `mail` may be empty;
- `userPrincipalName` may be a technical identifier such as `GUID@tenant.onmicrosoft.com`;
- the real email may exist only in the user's **Identities** collection.

In that case, `userName ← mail` produces an empty value and QANode responds:

```text
A valid userName/email is required
```

Do not replace it with a technical UPN just to make the test pass. The final `userName` value must be the user's real email.

Recommended procedure:

1. Open **Provision on demand** and select the user.
2. Inspect **Modified attributes / Data flow**.
3. Confirm that the target `userName` value is a real email.
4. If `mail` is empty, copy or synchronize the sign-in email into a source attribute exposed by provisioning, such as `mail` or an appropriate extension.
5. Map that attribute to `userName` and optionally to `emails[type eq "work"].value`.
6. Run on-demand provisioning again before starting the automatic cycle.

If the **Identities** collection is not available as a source in the mapping UI, it cannot be used directly by that provisioning configuration. Materialize the email in another provisionable property or synchronize it through a tenant administration process.

### Relationship between the OIDC and SCIM applications

The portal does not automatically link the two applications. They work together because they use:

- the same tenant;
- the same assigned users or groups;
- the same normalized email;
- QANode as their shared destination.

Assign the same users or groups to both the OIDC sign-in application and the SCIM enterprise application. SCIM creates the user, and the first OIDC sign-in links the external identity by email.

---

## Saving, Testing, and Enabling

Selecting SSO does not enable the integration immediately. Use this safe sequence:

1. Enter the configuration.
2. Select **Save draft**.
3. Select **Test configuration**.
4. Sign in to the IdP with the same account as the current Super Admin.
5. Return to QANode and confirm the **Verified** status.
6. Select **Enable SSO**.

QANode only enables the exact configuration tested by the same Super Admin. Changing the provider URL, issuer, client, secret, certificate, or other provider data invalidates verification and requires another test.

### Effects of enabling SSO

When SSO is enabled:

- local MFA is disabled globally;
- local MFA secrets are removed;
- users cannot enable MFA while SSO remains active;
- pending invitation tokens are invalidated;
- inviting, resending invitations, accepting invitations, and user import are blocked;
- existing local sessions are invalidated;
- users sign in through OIDC or SAML;
- only the Super Admin may retain local sign-in when emergency access is enabled.

Enforce MFA in the IdP through Conditional Access or the equivalent provider policy.

### Editing an active configuration

Changing provider-specific data automatically disables SSO so an unverified configuration cannot lock out the organization. After a change:

1. save the draft;
2. test again;
3. enable SSO again.

Changing the default role or revalidation interval does not alter roles already assigned to users.

---

## Operations and Maintenance

### User sign-in

Users should select **Sign in with SSO** on the QANode login page. For SAML, do not use the application tile in the IdP portal as the primary flow.

### SAML certificate rotation

QANode stores one IdP signing certificate per configuration. Before the current certificate expires:

1. generate or activate the new certificate in the IdP;
2. copy the new certificate into QANode;
3. save the draft — SSO is disabled for safety;
4. test the integration;
5. enable it again;
6. revoke the old certificate in the IdP after validation.

Plan a short maintenance window because the current version does not automatically overlap two certificates.

### SCIM token rotation

1. Generate a new token in QANode.
2. Immediately update **Secret Token** in the IdP.
3. Test the connection.
4. Run an on-demand provisioning operation.

The previous token stops working after rotation.

### Disabling SSO

When SSO is disabled, local authentication becomes available again. Users created only through JIT or SCIM have no local password and must complete a password recovery/definition process before signing in locally.

The SCIM token is not removed automatically when SSO is disabled. Revoke it separately if provisioning must also stop.

---

## Troubleshooting

| Symptom | Likely cause | How to fix it |
|---------|--------------|---------------|
| `redirect_uri_mismatch` or `AADSTS50011` | The URI in the IdP differs from the URI sent by QANode | Copy the QANode URI again and register it as a **Web** callback without changing any character |
| OIDC test fails before the login page | Wrong issuer or inaccessible discovery | Open `issuer/.well-known/openid-configuration` and confirm that its `issuer` exactly matches the configured value |
| `invalid_client` | Wrong Client ID or secret | Use the secret **value**, check expiration, and confirm `client_secret_post/basic` |
| Super Admin email does not match | The IdP returned another claim or alias | Adjust the user/claim so it returns the email registered in QANode |
| SAML fails from the provider test button | IdP-initiated flow with no QANode state | Start from **Test configuration** in QANode |
| SAML reports missing or invalid signature | Only the response was signed | Configure the IdP to sign the assertion |
| SAML response cannot be processed | Encrypted assertion or mandatory signed AuthnRequest | Disable encryption and the signed-request requirement |
| SCIM returns `A valid userName/email is required` | Empty `userName` or technical UPN | Map a source containing the real email and inspect the target value in on-demand provisioning |
| Entra shows `Active in the source system = False` | User disabled, deleted, or out of scope | Check application assignment, account state, filters, and `IsSoftDeleted` |
| Entra shows `UnprocessableEntry` for matching | Matching source attribute is empty | Fix `userName`, keep match precedence 1, and run on-demand provisioning again |
| User was removed from the IdP but remains active in QANode with JIT | JIT does not synchronize deactivation | Use SCIM or wait for session revalidation; migrate to SCIM for full lifecycle control |
| SCIM user was not created | User is unassigned, out of scope, or over the license limit | Check assignment, filters, status, provisioning logs, and available seats |
| Role did not change after an IdP update | Roles are not synchronized | Change the role in QANode |
| SSO was disabled after saving | Critical provider data changed | Test again and re-enable SSO |

---

## Rollout Checklist

- [ ] The Super Admin can test and sign in through SSO.
- [ ] Emergency local access was tested or intentionally disabled.
- [ ] A new regular user receives the expected default role.
- [ ] An existing user with the same email is linked without duplication.
- [ ] Changing a role in QANode is not undone by the IdP.
- [ ] A user without application assignment cannot authenticate.
- [ ] SCIM user creation produces the correct QANode record.
- [ ] With SCIM, `active: false` deactivates the user.
- [ ] Reactivating the user in the IdP reactivates them in QANode.
- [ ] The mapping sends a real email in `userName`.
- [ ] The IdP MFA policy is applied.
- [ ] The session revalidation interval meets the company offboarding policy.
- [ ] Owners and rotation dates are defined for secrets, the SCIM token, and the SAML certificate.

---

## Provider References

- [Microsoft Entra — OIDC protocol](https://learn.microsoft.com/entra/identity-platform/v2-protocols-oidc)
- [Microsoft Entra — SCIM attribute mappings](https://learn.microsoft.com/entra/identity/app-provisioning/customize-application-attributes)
- [Microsoft Entra — SCIM provisioning](https://learn.microsoft.com/entra/identity/app-provisioning/use-scim-to-provision-users-and-groups)
- [Google — OpenID Connect](https://developers.google.com/identity/openid-connect/openid-connect)
- [Google Workspace — custom SAML application](https://support.google.com/a/answer/6087519)
- [Okta — SAML field reference](https://help.okta.com/en-us/content/topics/apps/aiw-saml-reference.htm)
- [Keycloak — administration guide](https://www.keycloak.org/docs/latest/server_admin/)
- [JumpCloud — SAML connector fields](https://jumpcloud.com/support/sso-application-connector-fields)
- [Duo — generic SAML SSO](https://duo.com/docs/sso-generic)
