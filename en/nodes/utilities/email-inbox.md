# Email Inbox Node

The **Email Inbox** node connects to a mailbox via IMAP and waits for or searches for messages matching defined criteria. Ideal for validating email delivery, extracting OTPs, and capturing confirmation links in test flows.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `email-inbox` |
| **Category** | Utilities |
| **Input** | `in` |
| **Output** | `out` |

---

## Prerequisite: Email Credential

Before using this node, create an **Email Credential** in the [Credentials](../../credentials/credentials.md#email) section. It stores the IMAP connection and authentication securely.

---

## Configuration Fields

### General

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **Credential** | select | — | Email credential to use |
| **Operation** | select | `Extract Email` | What to do when an email is found |
| **Folder** | text | `INBOX` | IMAP folder to monitor |
| **Only Unread** | boolean | `true` | Search only unread messages |
| **Mark As Read** | boolean | `false` | Mark found messages as read |

### Available Operations

| Value | Description |
|-------|-------------|
| `Extract Email` | Wait for and return the full email |
| `Extract OTP` | Extract a numeric code from the email body |
| `Extract Link` | Extract a URL from the email body |

### Search Filters

All filters are optional. When filled, the value must be **contained** in the email for it to match.

| Field | Description | Example |
|-------|-------------|---------|
| **From Contains** | Filter by sender | `noreply@gmail.com` |
| **To Contains** | Filter by recipient | `myemail@` |
| **Subject Contains** | Filter by subject | `Verification code` |
| **Body Contains** | Filter by body text | `your code is` |

### Date Filter

| Field | Options | Description |
|-------|---------|-------------|
| **Received After Mode** | `Step Start` / `Absolute Date/Time` / `No Date Filter` | Date filter mode |
| **Received After** | ISO 8601 | Used only when mode = `Absolute Date/Time` |

> **Step Start (recommended):** the node uses the UID of the next email at the moment execution begins, reliably ignoring previous messages regardless of timezone.

### Polling and Limits

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **Timeout (ms)** | number | `120000` | Maximum wait time (0 = no wait, checks once) |
| **Poll Interval (ms)** | number | `3000` | Interval between checks |
| **Max Scan** | number | `30` | Maximum emails to scan per cycle |
| **Max Matches** | number | `1` | Maximum emails to return |

### OTP Extraction (when operation = `Extract OTP`)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **OTP Regex (optional)** | regex | automatic | Custom expression to capture the code |
| **OTP Min Length** | number | `6` | Minimum code length |
| **OTP Max Length** | number | `6` | Maximum code length |

When no regex is provided, the node automatically searches for numeric sequences of the configured length.

### Link Extraction (when operation = `Extract Link`)

| Field | Type | Description |
|-------|------|-------------|
| **Link Domain Contains** | text | Filter links by domain (e.g. `accounts.google.com`) |
| **Link Regex (optional)** | regex | Custom expression to capture the URL |

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `found` | `boolean` | Whether any email was found |
| `matchCount` | `number` | Number of emails that matched |
| `attempts` | `number` | Number of checks performed |
| `email` | `object` | First email found (full object) |
| `emails` | `array` | All emails found (up to `maxMatches`) |
| `links` | `array` | URLs found in the first email |
| `otp` | `string` | Extracted code (only in `extractOtp`) |
| `link` | `string` | Extracted URL (only in `extractLink`) |

### `email` Object Structure

```json
{
  "uid": 12345,
  "messageId": "<abc@gmail.com>",
  "subject": "Your verification code",
  "from": "Company <noreply@company.com>",
  "to": "user@gmail.com",
  "date": "2026-03-08T12:00:00.000Z",
  "text": "Your code is 482910. Valid for 10 minutes.",
  "html": "<p>Your code is <strong>482910</strong>...</p>",
  "links": ["https://company.com/confirm?token=xyz"]
}
```

---

## Practical Examples

### Wait for sign-up confirmation email

```
[Click "Create account" in the browser]
    │
    ▼
[Email Inbox]
  Operation: Extract Email
  Subject Contains: "Confirm your email"
  From Contains: noreply@service.com
  Timeout: 60000ms
    │
    ▼
[HTTP Request: GET {{ steps.emailInbox.outputs.links[0] }}]
```

### Extract 6-digit OTP

```
[Click "Send code" in the app]
    │
    ▼
[Email Inbox]
  Operation: Extract OTP
  Subject Contains: "verification code"
  OTP Min Length: 6
  OTP Max Length: 6
  Timeout: 30000ms
    │
    ▼
[Type in OTP field: {{ steps.emailInbox.outputs.otp }}]
```

### Extract password reset link

```
[Click "Forgot password"]
    │
    ▼
[Email Inbox]
  Operation: Extract Link
  Subject Contains: "reset password"
  Link Domain Contains: accounts.company.com
  Timeout: 30000ms
    │
    ▼
[Navigate: {{ steps.emailInbox.outputs.link }}]
```

---

## Provider Configuration

### Gmail

| Mode | Required Setup |
|------|---------------|
| **Password / App Password** | Enable "Less secure app access" or generate an [App Password](https://myaccount.google.com/apppasswords) |
| **OAuth2** | Create an app in the Google Cloud Console with the credential's callback URI |

> For Gmail with OAuth2, click **Connect OAuth (Browser)** in the credential screen. Google requires that the callback URI registered in the Console exactly matches the **OAuth Callback URI** field shown in the credential form.

### Outlook / Microsoft 365

| Mode | Required Setup |
|------|---------------|
| **Password / App Password** | Works for accounts without MFA, or with app passwords |
| **OAuth2** | Register the app in Azure Active Directory with `IMAP.AccessAsUser.All` permission |

### Custom IMAP

Manually configure host, port, and security mode (SSL/TLS).

---

## Tips

- **Always use Received After Mode = Step Start** — avoids capturing emails that arrived before the test
- **Combine filters** — the more specific (`from` + `subject`), the more reliable the detection
- **Mark as read** — useful to avoid the same email being captured in consecutive runs
- **Timeout 0** — runs a single check without waiting; useful for assertions on already-received emails
- **Custom OTP Regex** — use when the code has a non-numeric format or appears in ambiguous context (e.g. `code: ([A-Z0-9]{8})`)
