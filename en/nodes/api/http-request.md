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
| **Method** | `string` | `GET`, `QUERY`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS` |
| **URL** | `string` | Endpoint URL (supports `{{ }}`) |
| **Headers** | `object` | HTTP headers as key-value pairs |
| **Query parameters** | `array` | Key-value pairs added to the URL |
| **Body type** | `string` | `None`, `JSON`, `Raw text`, `x-www-form-urlencoded`, `multipart/form-data`, or `Binary file` |
| **Response type** | `string` | `Auto`, `Text`, `JSON`, or `File` |
| **Credential** | `string` | Saved credential for authentication |
| **Timeout (ms)** | `number` or expression | Maximum time for this request; defaults to 30,000 ms |

### Raw Mode

Raw mode allows you to edit the request as JSON, providing full control.

### Request timeout

The timeout applies to **each HTTP call**. The default is **30,000 ms (30 seconds)**, and the configurable range is **1 ms to 86,400,000 ms (24 hours)**. You can enter a fixed number or an expression such as `{{ variables.API_TIMEOUT_MS }}`. The expression must resolve to a number within that range.

When a call exceeds the limit, the execution explicitly reports an HTTP Request timeout with the configured duration. In the execution details, the possible cause and suggestion refer to the request, not to page loading. If the API usually responds more slowly, increase this field appropriately and check the server's availability.

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

## HTTP Methods

| Method | Typical use |
|--------|-------------|
| **GET** | Fetch data or resource details |
| **QUERY** | Query data with structured filters or criteria in the request body |
| **POST** | Create resources or submit data |
| **PUT** | Replace a complete resource |
| **PATCH** | Partially update a resource |
| **DELETE** | Remove a resource |
| **HEAD** | Fetch only status and headers, without a response body |
| **OPTIONS** | Discover methods and options supported by an endpoint |

### QUERY Method

`QUERY`, defined by [RFC 10008](https://www.rfc-editor.org/rfc/rfc10008.html), is a safe and idempotent HTTP method for queries that need to carry structured criteria in the request body. It is not the same as URL **Query parameters**.

Compare:

```text
GET /orders?status=pending&from=2026-01-01
```

with:

```text
QUERY /orders
Content-Type: application/json

{
  "status": ["pending", "review"],
  "period": {
    "from": "2026-01-01",
    "to": "2026-01-31"
  },
  "sort": ["createdAt:desc"]
}
```

Use `QUERY` when the API documentation explicitly supports it, especially for large filters, nested objects, or queries that do not fit well in a URL. Continue using `GET` for ordinary retrieval.

QANode supports the same body types for `QUERY` as for `POST`, `PUT`, and `PATCH`:

- JSON;
- raw text;
- `x-www-form-urlencoded`;
- `multipart/form-data`;
- binary file.

When content is present, send a matching `Content-Type`. QANode sets it automatically for structured body modes unless it was supplied manually.

Example with a URL-encoded form:

```text
Method: QUERY
URL: https://api.example.com/search
Body type: x-www-form-urlencoded
Fields:
  term = notebook
  limit = 50
  includeArchived = false
```

> The server, proxy, gateway, and WAF must accept `QUERY`. If any layer does not recognize it, the response may be `405 Method Not Allowed` or `501 Not Implemented`. Enable the method in the infrastructure or use the alternate contract defined by the API.

---

## Body

For methods that accept a body (`QUERY`, `POST`, `PUT`, `PATCH`, `DELETE`, and `OPTIONS`), choose **Body type**. QANode does not send a body with `GET` or `HEAD`.

### None

Sends no body. This is the option used for `GET` and `HEAD`.

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

### QUERY — Structured search

```text
Method: QUERY
URL: https://api.example.com/orders/search
Body type: JSON
JSON: {
  "customerIds": ["1001", "1002"],
  "status": ["pending", "review"],
  "createdAfter": "2026-01-01T00:00:00Z"
}
```

Expressions can be used normally inside the criteria:

```json
{
  "customerId": "{{ variables.CUSTOMER_ID }}",
  "cursor": "{{ steps[\"previous-page\"].outputs.body.nextCursor }}"
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
