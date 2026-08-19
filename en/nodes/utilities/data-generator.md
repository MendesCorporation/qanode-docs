# Data Generator Node

The **Data Generator** node creates synthetic test data without external services. A single execution can generate from 1 to 1,000 records containing identity, document, address, company, internet, text, numeric, date, payment card, and commerce data.

Use it whenever a scenario needs new users, orders, products, documents, or other values on each run. Configured fields appear in the variable picker and can be used directly by subsequent nodes.

> Generated data is synthetic. A structurally valid document follows the applicable format and check-digit rules, but does not necessarily represent a real person, company, address, phone number, or payment method.

---

## General Configuration

| Field | Description |
|-------|-------------|
| **Language and regional format** | Language and regional conventions used to generate values |
| **Default country** | Country used by documents, phones, addresses, and companies unless overridden by a field |
| **Quantity** | Number of generated records, from 1 to 1,000 |
| **Characters** | Keeps natural characters or normalizes text to ASCII by removing accents |
| **Seed** | Optional value that reproduces the same data with the same configuration |
| **Generated fields** | Properties included in every record |

Available regional formats include Brazilian and European Portuguese; US, UK, Canadian, and Australian English; Spanish, French, German, Italian, Japanese, and Simplified Chinese.

---

## Adding Fields

Click **Add field**, select the data type, and enter a unique **Output name**. That name becomes the key used in expressions by subsequent nodes.

| Category | Examples |
|----------|----------|
| **Identity and personal data** | first and last name, full name, email, username, password, birth date, age, phone, and fictitious avatar |
| **Documents and cards** | personal and business documents, passport, card number, issuer, CVV, and expiry |
| **Address and location** | full address, street, building number, district, city, state, postal code, country, and coordinates |
| **Company** | legal and trade name, department, job title, domain, corporate email, and phone |
| **Internet and development** | UUID, URL, domain, IPv4, IPv6, MAC address, hash, token, slug, hex color, and user agent |
| **Text, numbers, and values** | word, sentence, paragraph, integer, decimal, currency amount, percentage, boolean, dates, list, sequence, string, and template |
| **Commerce** | product, SKU, price, quantity, currency, order number, tracking code, and order status |

Available options depend on the selected type and may include country, format, structural validity, card issuer, mask, length, prefix, suffix, domain, currency, decimal places, numeric and date ranges, value lists, and password composition.

### Coherent records

Related fields share the same context within a record. For example:

- first name, last name, full name, username, and email belong to the same synthetic person;
- city, state, postal code, and full address belong to the same synthetic address;
- card number, issuer, CVV, and expiry belong to the same synthetic card;
- product, SKU, price, currency, order, and tracking code share the same commerce context.

Each item receives its own context when multiple records are generated.

---

## Valid and Invalid Data

Compatible fields can be set to **Structurally valid** or **Structurally invalid**. Invalid mode is useful for testing rejection rules and error messages.

This option is available for personal and business documents, passports, phone numbers, payment card numbers, and custom masks. It deliberately changes the value structure and does not query government, carrier, or issuer databases.

---

## Masks, Sequences, and Templates

Custom masks use:

| Character | Result |
|-----------|--------|
| `#` | Digit |
| `A` | Uppercase letter |
| `a` | Lowercase letter |
| `*` | Alphanumeric character |

Example: `ORD-####-AA`.

A **Sequence** can define its starting value, increment, and zero padding. A **Template** combines record values with fixed text, sequences, and masks:

```text
USR-{sequence}-{firstName}-###
```

Inserted values such as `{firstName}` are preserved literally. Any `A`, `a`, `*`, or `#` character inside the name is not interpreted again as a mask.

---

## Outputs

| Output | Description |
|--------|-------------|
| `data` | Array containing all generated records |
| `count` | Number of generated records |
| `first` | First generated record |

If the node is labeled **Generate user** and contains fields named `name` and `email`, use:

```text
{{ steps["Generate user"].outputs.first.name }}
{{ steps["Generate user"].outputs.first.email }}
{{ steps["Generate user"].outputs.count }}
{{ steps["Generate user"].outputs.data }}
```

To process multiple records, pass `outputs.data` to a [Loop](../flow-control/loop.md) node.

---

## AI-Assisted Generation

An AI connected to QANode through MCP can inspect the Data Generator contract, create its configuration, and use the output names in later steps. Describe the data by its testing purpose, for example:

```text
Create ten Brazilian customers with name, an email at example.com,
a valid CPF, an international phone number, and a sequential order number.
```

Review output names and validity rules before running, especially for negative test data.

---

## Best Practices

- Use a seed when you need to reproduce a failure with the same data.
- Use short, unique output names.
- Generate only the fields required by the scenario.
- Use ASCII when the tested system does not accept accents or non-Latin characters.
- Never treat generated values as real personal data, credentials, or payment information.
