# HTTP Request Node

The **HTTP Request** node allows you to make HTTP requests to REST APIs, SOAP services, or any web endpoint. It supports all common HTTP methods, multiple authentication types, and integration with saved credentials.

---

## Overview

| Property | Value |
|----------|-------|
| **Type** | `http-request` |
| **Category** | API |
| **Color** | Purple (#a855f7) |
| **Input** | `in` |
| **Output** | `out` |

---

## Configuration

### Builder Mode

Builder mode provides a visual interface for constructing the request:

| Field | Type | Description |
|-------|------|-------------|
| **Method** | `string` | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| **URL** | `string` | Endpoint URL (supports `{{ }}`) |
| **Headers** | `object` | HTTP headers (key-value pairs) |
| **Body** | `any` | Request body (JSON or plain text) |
| **Credential** | `string` | Saved credential for authentication |

### Raw Mode

Raw mode allows you to edit the request as pure JSON, providing full control.

---

## Authentication

The node supports multiple authentication methods:

### Using Saved Credentials

The most secure and recommended approach. Select a credential of type **HTTP/API** that contains the base URL and token:

1. In the **Credential** field, select the desired credential
2. The base URL and authentication headers will be applied automatically
3. In the **URL** field, provide only the path: `/api/users`

### Manual Authentication

| Type | Fields | Result |
|------|--------|--------|
| **Bearer Token** | Token | Header `Authorization: Bearer {token}` |
| **Basic Auth** | Username + Password | Header `Authorization: Basic {base64}` |
| **API Key** | Header Name + Token | Custom header with the token |

---

## HTTP Methods

| Method | Typical Use |
|--------|------------|
| **GET** | Fetch data (list, get details) |
| **POST** | Create resources, submit data |
| **PUT** | Replace a resource entirely |
| **PATCH** | Partially update a resource |
| **DELETE** | Remove a resource |

---

## Headers

Add custom headers as key-value pairs:

| Header | Example |
|--------|---------|
| `Content-Type` | `application/json` |
| `Accept` | `application/json` |
| `X-Custom-Header` | `my-value` |
| `Authorization` | `Bearer {{ variables.TOKEN }}` |

> Headers support `{{ }}` expressions for dynamic values.

---

## Body (Request Body)

For methods that accept a body (POST, PUT, PATCH), you can send:

### JSON

```json
{
  "name": "{{ variables.USER_NAME }}",
  "email": "{{ steps["web-flow"].outputs.extracts.email }}",
  "active": true
}
```

### Form Data

```json
{
  "username": "admin",
  "password": "{{ variables.ADMIN_PASS }}"
}
```

> The body supports `{{ }}` expressions in any value.

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `status` | `number` | HTTP status code (200, 404, 500, etc.) |
| `body` | `string` | Response body as plain text |
| `json` | `any` | Response body parsed as JSON |

### Accessing Outputs

```
// Response status
{{ steps["http-request"].outputs.status }}  →  200

// JSON body
{{ steps["http-request"].outputs.json }}  →  { "id": 1, "name": "João" }

// Specific JSON property
{{ steps["http-request"].outputs.json.name }}  →  "João"

// Array in JSON
{{ steps["http-request"].outputs.json.items[0].title }}  →  "Primeiro Item"

// Body as plain text
{{ steps["http-request"].outputs.body }}  →  '{"id": 1, "name": "João"}'
```

---

## Practical Examples

### GET — Fetch a user

```
Method: GET
URL: https://api.exemplo.com/users/1
Headers: { "Accept": "application/json" }
```

### POST — Create a resource

```
Method: POST
URL: https://api.exemplo.com/users
Headers: { "Content-Type": "application/json" }
Body: {
  "name": "Maria",
  "email": "maria@exemplo.com"
}
```

### PUT with Bearer authentication

```
Method: PUT
URL: https://api.exemplo.com/users/1
Auth: Bearer Token → {{ steps.login.outputs.json.token }}
Body: {
  "name": "Maria Silva",
  "role": "admin"
}
```

### Chaining requests

```
[HTTP Request: POST /login]
    │ outputs.json.token = "abc123"
    ▼
[HTTP Request: GET /api/profile]
    │ Header: Authorization = Bearer {{ steps.login.outputs.json.token }}
    ▼
[If: {{ steps.profile.outputs.status }} === 200]
    │ true → [Log: "Perfil: {{ steps.profile.outputs.json.name }}"]
```

---

## Error Handling

| Status | Meaning | Suggested Action |
|--------|---------|-----------------|
| `200-299` | Success | Continue normally |
| `400` | Bad request | Check body/params |
| `401` | Unauthorized | Check token/credential |
| `403` | Forbidden | Check permissions |
| `404` | Not found | Check URL |
| `500` | Server error | Check API |

Use the **If** node after HTTP Request to handle different status codes:

```
[HTTP Request] → [If: status === 200]
                    │ true → [Continue...]
                    │ false → [Log: "Error: {{ steps.api.outputs.status }}"]
```

---

## Tips

- **Use saved credentials** instead of placing tokens directly in fields — it is more secure and easier to maintain
- **Check the status** before using the JSON — an error response may not have the expected format
- **Use variables** for base URLs: `{{ variables.API_URL }}/endpoint` makes it easy to switch environments
- The `json` output returns `null` if the response is not valid JSON — in that case, use `body`
