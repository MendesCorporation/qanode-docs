# Generate File Node

The **Generate File** node creates files inside a QANode execution. It can generate TXT, JSON, CSV, Excel, or PDF files from content entered in the panel or produced by expressions from previous nodes.

---

## Overview

[Overview](../../assets/images/nos-generate-file.mp4)

| Property | Value |
|----------|-------|
| **Type** | `file-generate` |
| **Category** | Files |
| **Color** | 🟤 Gold (#C89F65) |
| **Input** | `in` |
| **Output** | `out` |

---

## When to Use

Use this node when the flow needs to:

- create CSV or Excel test data;
- generate JSON to send to an API;
- build a helper TXT file;
- create a simple PDF for attachment, download, or validation;
- produce a file that will be used by HTTP Request, Smart Web Flow, Mobile Flow, SSH, a component, or Custom JavaScript.

---

## Configuration

| Field | Type | Description |
|-------|------|-------------|
| **File type** | selection | TXT, JSON, CSV, Excel, or PDF |
| **File name** | text | Final file name, including extension |

The remaining fields change according to the selected type.

### TXT

| Field | Description |
|-------|-------------|
| **Content** | File text. Accepts `{{ }}` expressions |

### JSON

| Field | Description |
|-------|-------------|
| **JSON** | Valid JSON object or array |

### CSV

| Field | Description |
|-------|-------------|
| **Mode** | Manual rows or from list |
| **Columns** | Column names |
| **Rows** | Values for each row when using manual mode |
| **List** | Array from another node when using list mode |
| **Mapping** | Item field used for each column |

### Excel

| Field | Description |
|-------|-------------|
| **Sheet name** | Worksheet name inside the file |
| **Mode** | Manual rows or from list |
| **Columns / Rows / List** | Same logic as CSV |

### PDF

| Field | Description |
|-------|-------------|
| **Title** | Optional title shown in the PDF |
| **Content** | Text or JSON to render in the PDF |

---

## Manual Table and From List

For CSV and Excel, there are two main ways to build the table.

### Manual rows

Use this when the data is small or fixed.

```
Columns: id, name, email
Row 1: 1, Maria, maria@company.com
Row 2: 2, John, john@company.com
```

### From list

Use this when the data already comes from an API, database, component, or previous node.

```
List: {{ steps["query"].outputs.rows }}
Columns: id, name, email
Mappings: id, name, email
```

If the list contains objects, QANode can infer common columns, but in shared flows it is better to define the columns explicitly.

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `fileRef` | `fileRef` | Reference to the generated file |
| `name` | `string` | File name |
| `mimeType` | `string` | File MIME type |
| `sizeBytes` | `number` | Size in bytes |

### Accessing Outputs

```
{{ steps["file-generate"].outputs.fileRef }}
{{ steps["file-generate"].outputs.name }}
{{ steps["file-generate"].outputs.mimeType }}
{{ steps["file-generate"].outputs.sizeBytes }}
```

Use `fileRef` to pass the file to other nodes. The other fields are useful for logs, validations, and messages.

---

## Example: Generate Test Data CSV

```
File type: CSV
File name: customers.csv
Mode: From list
List: {{ steps["query-customers"].outputs.rows }}
Columns:
  - id
  - name
  - email
```

Then the file can be used with:

```
{{ steps["file-generate"].outputs.fileRef }}
```

---

## Tips

- Include the extension in the file name: `.txt`, `.json`, `.csv`, `.xlsx`, or `.pdf`.
- For CSV and Excel, prefer **From list** when the source data is already an array.
- Use clear names, such as `customer-data.csv` or `order-report.pdf`.
- To upload a file to an API or web page, always pass the `fileRef`, not the text content.

