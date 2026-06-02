# HTTP Request Node

The **HTTP Request** node lets you make HTTP requests to REST APIs, SOAP services, or any web endpoint. It supports common HTTP methods, multiple authentication options, saved credentials, file uploads, and file responses.

---

## Overview

[Overview](../../assets/images/nos-http-request-visao-geral.mp4)

| Property | Value |
|----------|-------|
| **Type** | `http-request` |
| **Category** | API |
| **Color** | 🟣 Purple (#a855f7) |
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
| **Headers** | `object` | HTTP headers as key-value pairs |
| **Query parameters** | `array` | Key-value pairs added to the URL |
| **Body type** | `string` | `None`, `JSON`, `Raw text`, `x-www-form-urlencoded`, `multipart/form-data`, or `Binary file` |
| **Response type** | `string` | `Auto`, `Text`, `JSON`, or `File` |
| **Credential** | `string` | Saved credential for authentication |

### Raw Mode

Raw mode allows you to edit the request as JSON, providing full control.

### Import cURL

**Import cURL** converts a `curl` command into the visual request configuration. QANode tries to detect:

- HTTP method;
- URL;
- headers;
- query parameters;
- JSON or text body;
- `x-www-form-urlencoded`;
- `multipart/form-data`;
- binary file sent with `--data-binary @file`.

When the cURL references a local file, QANode shows the imported name/path and the user must attach or inform the corresponding `fileRef` in the file field.

---

## Authentication

The node supports multiple authentication methods.

### Using Saved Credentials

The safest and recommended approach. Select an **HTTP/API** credential containing the base URL and token:

1. Select the credential in **Credential**
2. The base URL and authentication headers are applied automatically
3. In **URL**, enter only the path, such as `/api/users`

### Manual Authentication

| Type | Fields | Result |
|------|--------|--------|
| **Bearer Token** | Token | Header `Authorization: Bearer {token}` |
| **Basic Auth** | Username + password | Header `Authorization: Basic {base64}` |
| **API Key** | Header name + token | Custom header with the token |

---

## Body

For methods that accept a body (POST, PUT, PATCH), choose **Body type**.

### None

Sends no body. This is the default for `GET` and `DELETE`.

### JSON

```json
{
  "name": "{{ variables.USER_NAME }}",
  "email": "{{ steps["web-flow"].outputs.extracts.email }}",
  "active": true
}
```

The JSON editor highlights syntax and accepts `{{ }}` expressions inside values.

### Raw text

Use for textual payloads, XML, SOAP, scripts, or any content that should not be treated as JSON.

```xml
<login>
  <user>{{ variables.USER }}</user>
</login>
```

### x-www-form-urlencoded

Sends fields as `application/x-www-form-urlencoded`.

| Field | Value |
|-------|-------|
| `username` | `admin` |
| `password` | `{{ variables.ADMIN_PASS }}` |

### multipart/form-data

Sends form fields in multipart format, including text and files.

| Field type | Description |
|------------|-------------|
| **Text** | Regular text field |
| **File** | Field that receives a `fileRef` |

Example:

```
Method: POST
URL: https://httpbin.org/post
Body type: multipart/form-data
Text field:
  description = upload test
File field:
  file = {{ steps["file-generate"].outputs.fileRef }}
```

### Binary file

Sends the contents of a single file as the request body. Use this when the API expects the file directly in the body, not inside a multipart form.

```
Body type: Binary file
File: {{ steps["file-generate"].outputs.fileRef }}
```

> QANode uses the file MIME type when available. If needed, add a `Content-Type` header manually.

---

## File Responses

The **Response type** field controls how the API response is handled:

| Type | Behavior |
|------|----------|
| **Auto** | Detects JSON/text or file from content and headers |
| **JSON** | Tries to parse the response as JSON |
| **Text** | Returns the body as text |
| **File** | Saves the response as an artifact and exposes `fileRef` |

Use **File** when the API returns PDF, image, CSV, Excel, ZIP, or another binary content.

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `status` | `number` | HTTP status code (200, 404, 500, etc.) |
| `body` | `any` | Response body as text or JSON, depending on the response |
| `fileRef` | `fileRef` | File returned by the API, when the response is handled as a file |

### Accessing Outputs

```
// Response status
{{ steps["http-request"].outputs.status }}  →  200

// JSON or text body
{{ steps["http-request"].outputs.body }}  →  { "id": 1, "name": "John" }

// Array in JSON
{{ steps["http-request"].outputs.body.items[0].title }}  →  "First Item"

// File returned by the API
{{ steps["http-request"].outputs.fileRef }}
```

> Legacy fields such as `json`, `headers`, `text`, `rawBody`, and `statusCode` may remain accessible in manual expressions during the compatibility window, but the variables panel suggests only the new surface: `status`, `body`, or `fileRef`.

---

## Practical Examples

### GET — Fetch user

```
Method: GET
URL: https://api.example.com/users/1
Headers: { "Accept": "application/json" }
```

### POST — Create resource

```
Method: POST
URL: https://api.example.com/users
Body type: JSON
JSON: {
  "name": "Maria",
  "email": "maria@example.com"
}
```

### POST — Multipart upload

```
Method: POST
URL: https://httpbin.org/post
Body type: multipart/form-data
Text field: description = upload test
File field: file = {{ steps["file-generate"].outputs.fileRef }}
```

### GET — Download file

```
Method: GET
URL: https://httpbin.org/image/png
Response type: File
```

---

## Error Handling

| Status | Meaning | Suggested action |
|--------|---------|------------------|
| `200-299` | Success | Continue normally |
| `400` | Bad request | Check body/params |
| `401` | Unauthorized | Check token/credential |
| `403` | Forbidden | Check permissions |
| `404` | Not found | Check URL |
| `500` | Server error | Check API |

Use an **If** node after HTTP Request to handle different status codes:

```
[HTTP Request] → [If: status === 200]
                    │ true → [Continue...]
                    │ false → [Log: "Error: {{ steps.api.outputs.status }}"]
```

---

## Tips

- Use saved credentials instead of putting tokens directly in fields.
- Check `status` before using `body`.
- Use variables for base URLs: `{{ variables.API_URL }}/endpoint`.
- Use `multipart/form-data` when the API expects a form upload.
- Use **Response type = File** for binary responses.

