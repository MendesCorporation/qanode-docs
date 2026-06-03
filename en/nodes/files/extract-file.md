# Extract File Node

The **Extract File** node reads a file received as `fileRef` and returns text or structured data according to the selected extraction type.

---

## Overview

[Overview](../../assets/images/nos-extract-file.mp4)

| Property | Value |
|----------|-------|
| **Type** | `file-extract` |
| **Category** | Files |
| **Color** | 🟤 Gold (#C89F65) |
| **Input** | `in` |
| **Output** | `out` |

---

## When to Use

Use this node when the flow needs to:

- read TXT or Markdown as text;
- validate or reuse JSON content;
- transform CSV into structured rows;
- read Excel spreadsheets;
- extract text from PDF;
- use OCR as fallback when the PDF is image-based.

---

## Configuration

| Field | Type | Description |
|-------|------|-------------|
| **File** | `fileRef` | Input file |
| **Extraction type** | selection | TXT, JSON, CSV, Excel, or PDF |

The following fields change according to the extraction type.

### TXT

| Field | Description |
|-------|-------------|
| **Encoding** | Encoding used to read the text, such as `utf8` |

### JSON

| Field | Description |
|-------|-------------|
| **Encoding** | Encoding used to read the file |

The content must be valid JSON.

### CSV

| Field | Description |
|-------|-------------|
| **Delimiter** | Column separator, such as `,` or `;` |
| **Has header** | Uses the first line as column names |
| **Encoding** | File encoding |

### Excel

| Field | Description |
|-------|-------------|
| **Sheet name** | Sheet to read. If empty, uses the first sheet |
| **Header row** | Row used as header |

### PDF

| Field | Description |
|-------|-------------|
| **Page range** | Pages to extract, such as `1-3,5` |
| **OCR** | Enables OCR fallback for image-based PDFs |
| **OCR languages** | OCR languages, such as `por+eng+spa` |
| **OCR scale** | Scale used when rendering before OCR |

---

## OCR in PDF

When the PDF has real text, QANode tries to extract it directly. If the PDF appears to be image-based, OCR can be used automatically as fallback.

Use OCR for:

- scanned PDFs;
- payslips and receipts as images;
- scanned documents;
- files without a text layer.

Common languages:

| Value | Languages |
|-------|-----------|
| `por` | Portuguese |
| `eng` | English |
| `spa` | Spanish |
| `por+eng+spa` | Portuguese, English, and Spanish |

---

## Outputs

Outputs depend on the selected type.

### TXT

| Output | Type | Description |
|--------|------|-------------|
| `text` | `string` | Extracted text |

### JSON

| Output | Type | Description |
|--------|------|-------------|
| `json` | `any` | Parsed JSON |
| `text` | `string` | Original text |

### CSV and Excel

| Output | Type | Description |
|--------|------|-------------|
| `rows` | `array` | Extracted rows |
| `columns` | `array` | Detected columns |
| `rowCount` | `number` | Number of rows |
| `sheets` | `array` | Sheets found, for Excel |

### PDF

| Output | Type | Description |
|--------|------|-------------|
| `text` | `string` | Extracted text |
| `pages` | `array` | Text by page |
| `pageCount` | `number` | Number of processed pages |

---

## Examples

### Extract CSV

```
File: {{ steps["file-generate"].outputs.fileRef }}
Extraction type: CSV
Delimiter: ,
Has header: true
```

Later use:

```
{{ steps["file-extract"].outputs.rows[0].email }}
{{ steps["file-extract"].outputs.rowCount }}
```

### Extract PDF text

```
File: {{ steps["http-request"].outputs.fileRef }}
Extraction type: PDF
Page range: 1-2
OCR: true
OCR languages: por+eng+spa
```

---

## Type Validation

The node validates whether the file is compatible with the extraction type. If the file is not supported, the field turns red in the panel and execution fails with a clear message, for example:

```
Unsupported file: spreadsheet.xlsx
```

---

## Tips

- Choose the extraction type according to the real file content.
- For scanned PDFs, enable OCR; for PDFs with real text, leave OCR as fallback.
- For CSV with `;`, change the delimiter before running.
- Use `rows` to feed loops, components, file generation, or validations.

