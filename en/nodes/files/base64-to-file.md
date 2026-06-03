# Base64 to File Node

The **Base64 to File** node receives Base64 content or a Data URI and creates a file inside QANode, returning a `fileRef` for the next nodes.

---

## Overview

[Overview](../../assets/images/nos-base64-to-file.mp4)

| Property | Value |
|----------|-------|
| **Type** | `file-from-base64` |
| **Category** | Files |
| **Color** | 🟤 Gold (#C89F65) |
| **Input** | `in` |
| **Output** | `out` |

---

## When to Use

Use this node when:

- an API returns a file as Base64;
- a custom integration produces a Data URI;
- you need to transform Base64 into `fileRef` before sending it to another node;
- you want to convert Base64 to a real file and then extract it.

---

## Configuration

| Field | Type | Description |
|-------|------|-------------|
| **Base64** | `string` | Base64 content or Data URI |
| **File name** | `string` | Final file name |

The user does not need to fill in the MIME type. QANode tries to infer it automatically from:

- Data URI prefix, when present;
- file signature;
- file extension.

Example Data URI:

```
data:application/pdf;base64,JVBERi0xLjQK...
```

Example plain Base64:

```
JVBERi0xLjQK...
```

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `fileRef` | `fileRef` | Reference to the generated file |
| `name` | `string` | File name |
| `mimeType` | `string` | Inferred MIME type |
| `sizeBytes` | `number` | File size |

### Accessing Outputs

```
{{ steps["file-from-base64"].outputs.fileRef }}
{{ steps["file-from-base64"].outputs.name }}
{{ steps["file-from-base64"].outputs.mimeType }}
{{ steps["file-from-base64"].outputs.sizeBytes }}
```

---

## Examples

### API returns Base64 PDF

```
[HTTP Request] → returns body.documentBase64
[Base64 to File]
  Base64: {{ steps["http-request"].outputs.body.documentBase64 }}
  File name: contract.pdf
```

Then:

```
{{ steps["file-from-base64"].outputs.fileRef }}
```

### Convert Data URI image

```
Base64: {{ steps["custom-js"].outputs.result.imageDataUri }}
File name: screenshot.png
```

---

## Round Trip

This node is the opposite of **File to Base64**:

```
[File to Base64] → [Base64 to File]
```

or:

```
[Base64 to File] → [Extract File]
```

---

## Tips

- Prefer Data URI when the source already provides MIME type information.
- If using plain Base64, provide a file name with extension to help inference.
- To validate the content, chain it with **Extract File**.

