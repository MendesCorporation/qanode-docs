# File to Base64 Node

The **File to Base64** node receives a `fileRef` and returns the file content as Base64 and Data URI.

---

## Overview

[Overview](../../assets/images/nos-file-to-base64.mp4)

| Property | Value |
|----------|-------|
| **Type** | `file-to-base64` |
| **Category** | Files |
| **Color** | 🟤 Gold (#C89F65) |
| **Input** | `in` |
| **Output** | `out` |

---

## When to Use

Use this node when:

- an API expects a file inside a JSON field as Base64;
- you need to send file content in a message or payload;
- you want to convert a `fileRef` into a Data URI;
- another system does not accept multipart upload or binary upload.

---

## Configuration

| Field | Type | Description |
|-------|------|-------------|
| **File** | `fileRef` | File to convert |
| **Include Data URI** | `boolean` | Also returns `dataUri` with MIME prefix |

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `base64` | `string` | File content in Base64 |
| `dataUri` | `string` | Data URI when enabled |
| `name` | `string` | File name |
| `mimeType` | `string` | MIME type |
| `sizeBytes` | `number` | Original file size |

### Accessing Outputs

```
{{ steps["file-to-base64"].outputs.base64 }}
{{ steps["file-to-base64"].outputs.dataUri }}
{{ steps["file-to-base64"].outputs.mimeType }}
```

---

## Example: Send Base64 in JSON

```
[Generate File] → [File to Base64] → [HTTP Request]
```

HTTP body:

```json
{
  "fileName": "{{ steps["file-to-base64"].outputs.name }}",
  "mimeType": "{{ steps["file-to-base64"].outputs.mimeType }}",
  "content": "{{ steps["file-to-base64"].outputs.base64 }}"
}
```

---

## Round Trip

Use with **Base64 to File** when you need to convert both ways:

```
fileRef → Base64 → fileRef
```

---

## Tips

- Prefer `fileRef` between QANode nodes. Convert to Base64 only when the external system requires it.
- Base64 increases payload size; avoid it for very large files.
- Use `dataUri` when the receiving system expects the `data:mime/type;base64,...` format.

