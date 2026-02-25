# Expression System

QANode uses an **expression** system with `{{ }}` syntax that allows you to inject dynamic data into any node configuration field. With expressions, you can access outputs from previous nodes, global variables, and environment variables.

---

## Basic Syntax

```
{{ expression }}
```

Everything inside `{{ }}` is evaluated as JavaScript in the flow's execution context.

---

## Available Context

### `steps` — Outputs from Previous Nodes

Access data produced by any node that has already been executed:

```
{{ steps["Node Name"].outputs.property }}
```

| Component | Description |
|-----------|-------------|
| `steps` | Object containing all executed nodes |
| `["Node Name"]` | Node identifier — see rules below |
| `.outputs` | Node output data |
| `.property` | Specific output field |

#### How the identifier works in steps

The name used in `steps` is the node's **identifier**. By default, every node uses its **type** in lowercase as the identifier (e.g., `if`, `http-request`, `postgres-query`). You can change the identifier by setting a **custom label** in the properties panel.

**Notation rule:**

| Situation | Identifier | Notation | Example |
|-----------|------------|----------|---------|
| Simple default type | `if`, `merge`, `loop` | Dot `.` | `steps.if.outputs.result` |
| Default type with hyphen | `http-request`, `postgres-query` | Brackets `["..."]` | `steps["http-request"].outputs.status` |
| Simple custom label | `login`, `api`, `query` | Dot `.` | `steps.login.outputs.json.token` |
| Custom label with space | `Login API`, `DB Query` | Brackets `["..."]` | `steps["DB Query"].outputs.rows` |

> **Tip:** If you rename the node label to a single word without spaces (e.g., `login`, `api`, `query`), you can use the dot notation, which is simpler: `steps.login.outputs.json.token`

**Duplicate nodes:** If there are two nodes with the same identifier, the second one receives a numeric suffix (e.g., "http-request 2", "http-request 3").

#### Default node identifiers (types)

| Node Type | Default Identifier | Notation |
|-----------|--------------------|----------|
| If | `if` | `steps.if` |
| Switch | `switch` | `steps.switch` |
| Loop | `loop` | `steps.loop` |
| Merge | `merge` | `steps.merge` |
| HTTP Request | `http-request` | `steps["http-request"]` |
| Postgres | `postgres-query` | `steps["postgres-query"]` |
| MySQL | `mysql-query` | `steps["mysql-query"]` |
| MariaDB | `mariadb-query` | `steps["mariadb-query"]` |
| Oracle | `oracle-query` | `steps["oracle-query"]` |
| MongoDB | `mongodb-query` | `steps["mongodb-query"]` |
| Web Flow | `web-flow` | `steps["web-flow"]` |
| Smart Locators | `smart-locators` | `steps["smart-locators"]` |
| SSH Command | `ssh-command` | `steps["ssh-command"]` |
| Custom JavaScript | `custom-js` | `steps["custom-js"]` |
| Set Variable | `set-variable` | `steps["set-variable"]` |
| Log | `log` | `steps.log` |

**Examples with default identifiers:**

```
// HTTP status (type "http-request" — requires brackets)
{{ steps["http-request"].outputs.status }}  →  200

// JSON response body
{{ steps["http-request"].outputs.json.name }}  →  "João"

// Database array
{{ steps["postgres-query"].outputs.rows[0].email }}  →  "joao@exemplo.com"

// Record count
{{ steps["postgres-query"].outputs.rowCount }}  →  5

// Text extracted from web page
{{ steps["web-flow"].outputs.extracts.titulo }}  →  "Dashboard"

// Assertion result
{{ steps["web-flow"].outputs.asserts.loginOk }}  →  true

// Browser session ID
{{ steps["smart-locators"].outputs.sessionId }}  →  "abc-123"

// SSH output
{{ steps["ssh-command"].outputs.stdout }}  →  "active (running)"
{{ steps["ssh-command"].outputs.exitCode }}  →  0

// If node
{{ steps.if.outputs._branch }}  →  "true" or "false"

// Loop node
{{ steps.loop.outputs._loopCount }}  →  10
```

**Examples with custom labels (no space — dot notation):**

```
// If you rename the HTTP Request node to "api":
{{ steps.api.outputs.status }}  →  200
{{ steps.api.outputs.json.name }}  →  "João"

// If you rename the Custom JavaScript node to "transform":
{{ steps.transform.outputs.result }}  →  { "total": 42 }

// If you rename the Postgres node to "query":
{{ steps.query.outputs.rows[0].email }}  →  "joao@exemplo.com"
```

### `variables` — Global Variables

Access variables registered in the system:

```
{{ variables.VARIABLE_NAME }}
```

**Examples:**

```
// String
{{ variables.API_URL }}  →  "https://api.exemplo.com"

// Number
{{ variables.DEFAULT_TIMEOUT }}  →  30000

// Boolean
{{ variables.FEATURE_FLAG }}  →  true

// JSON (property access)
{{ variables.CONFIG.retries }}  →  3
{{ variables.CONFIG.timeout }}  →  5000
```

---

## Usage Modes

### Single Expression (Direct Value)

When the entire field is an expression, the returned value retains its original type (number, object, array, boolean):

```
Field: {{ steps["postgres-query"].outputs.rows }}
Result: [{ "id": 1, "name": "João" }, { "id": 2, "name": "Maria" }]  (array)

Field: {{ steps["http-request"].outputs.status }}
Result: 200  (number)

Field: {{ steps["web-flow"].outputs.asserts.loginOk }}
Result: true  (boolean)
```

### Interpolation (Inside Text)

When the expression is mixed with text, the result is always a string:

```
Field: Hello, {{ steps["postgres-query"].outputs.rows[0].name }}!
Result: "Hello, João!"  (string)

Field: {{ variables.API_URL }}/api/users/{{ steps["http-request"].outputs.json.id }}
Result: "https://api.exemplo.com/api/users/42"  (string)

Field: Status: {{ steps["http-request"].outputs.status }} - {{ steps["http-request"].outputs.json.message }}
Result: "Status: 200 - OK"  (string)
```

---

## JavaScript Expressions

Inside `{{ }}`, any valid JavaScript expression works:

### String Operations

```
// Uppercase (Web Flow node renamed to "extract")
{{ steps.extract.outputs.extracts.nome.toUpperCase() }}  →  "JOÃO"

// Lowercase
{{ steps.extract.outputs.extracts.nome.toLowerCase() }}  →  "joão"

// First letter uppercase
{{ steps.extract.outputs.extracts.nome.charAt(0).toUpperCase() + steps.extract.outputs.extracts.nome.slice(1).toLowerCase() }}  →  "João"

// Substring
{{ steps.extract.outputs.extracts.texto.substring(0, 100) }}  →  "First 100 characters..."

// Slice (allows negative indices)
{{ steps.extract.outputs.extracts.texto.slice(0, 50) }}  →  "First 50 characters"
{{ steps.extract.outputs.extracts.texto.slice(-10) }}  →  "Last 10"

// Replacement
{{ steps.extract.outputs.extracts.url.replace("http:", "https:") }}
{{ steps.extract.outputs.extracts.texto.replaceAll("a", "A") }}  →  replaces all occurrences

// Concatenation (using default label with brackets)
{{ steps["postgres-query"].outputs.rows[0].firstName + " " + steps["postgres-query"].outputs.rows[0].lastName }}

// Template strings
{{ `${steps["postgres-query"].outputs.rows[0].firstName} ${steps["postgres-query"].outputs.rows[0].lastName}` }}

// Length
{{ steps.extract.outputs.extracts.texto.length }}  →  150

// Check if contains
{{ steps.extract.outputs.extracts.email.includes("@") }}  →  true
{{ steps.extract.outputs.extracts.url.startsWith("https://") }}  →  true
{{ steps.extract.outputs.extracts.arquivo.endsWith(".pdf") }}  →  true

// Find position
{{ steps.extract.outputs.extracts.email.indexOf("@") }}  →  5
{{ steps.extract.outputs.extracts.texto.lastIndexOf("palavra") }}  →  last occurrence

// Split string
{{ steps.extract.outputs.extracts.tags.split(",") }}  →  ["tag1", "tag2", "tag3"]
{{ steps.extract.outputs.extracts.nome.split("") }}  →  ["J", "o", "ã", "o"]

// Trim whitespace
{{ steps.extract.outputs.extracts.nome.trim() }}  →  removes leading and trailing spaces
{{ steps.extract.outputs.extracts.nome.trimStart() }}  →  removes leading spaces
{{ steps.extract.outputs.extracts.nome.trimEnd() }}  →  removes trailing spaces

// Repeat
{{ "abc".repeat(3) }}  →  "abcabcabc"

// Pad with zeros
{{ steps.api.outputs.json.id.toString().padStart(5, "0") }}  →  "00042"
{{ steps.api.outputs.json.code.padEnd(10, "-") }}  →  "ABC-------"

// Extract character
{{ steps.extract.outputs.extracts.nome.charAt(0) }}  →  "J"
{{ steps.extract.outputs.extracts.nome[0] }}  →  "J"
{{ steps.extract.outputs.extracts.nome.at(-1) }}  →  last character
```

### Number Operations

```
// Calculation (HTTP Request node renamed to "cart")
{{ steps.cart.outputs.json.subtotal * 1.1 }}  →  adds 10%
{{ steps.cart.outputs.json.price * steps.cart.outputs.json.quantity }}  →  total

// Rounding (node renamed to "api")
{{ Math.round(steps.api.outputs.json.score) }}  →  rounds to nearest integer
{{ Math.ceil(steps.api.outputs.json.score) }}  →  rounds up
{{ Math.floor(steps.api.outputs.json.score) }}  →  rounds down
{{ Math.trunc(steps.api.outputs.json.score) }}  →  removes decimals

// Decimal places
{{ Math.round(steps.api.outputs.json.score * 100) / 100 }}  →  2 decimal places
{{ steps.api.outputs.json.price.toFixed(2) }}  →  "10.50" (string)
{{ parseFloat(steps.api.outputs.json.price.toFixed(2)) }}  →  10.5 (number)

// Formatting (Postgres node renamed to "query")
{{ steps.query.outputs.rowCount.toString().padStart(5, "0") }}  →  "00042"

// Absolute value
{{ Math.abs(steps.api.outputs.json.balance) }}  →  always positive

// Maximum and minimum
{{ Math.max(10, 20, 30) }}  →  30
{{ Math.min(10, 20, 30) }}  →  10
{{ Math.max(...steps.query.outputs.rows.map(r => r.age)) }}  →  maximum age

// Power and root
{{ Math.pow(2, 3) }}  →  8 (2³)
{{ 2 ** 3 }}  →  8 (2³)
{{ Math.sqrt(16) }}  →  4 (square root)
{{ Math.cbrt(27) }}  →  3 (cube root)

// Random number
{{ Math.random() }}  →  0.0 to 1.0
{{ Math.floor(Math.random() * 100) }}  →  0 to 99
{{ Math.floor(Math.random() * 10) + 1 }}  →  1 to 10

// Sign
{{ Math.sign(steps.api.outputs.json.balance) }}  →  -1, 0, or 1

// Conversion
{{ parseInt("42") }}  →  42
{{ parseInt("42.7") }}  →  42
{{ parseFloat("42.7") }}  →  42.7
{{ Number("42.7") }}  →  42.7
{{ parseInt("ff", 16) }}  →  255 (hexadecimal)

// Verification
{{ Number.isNaN(steps.api.outputs.json.value) }}  →  true if NaN
{{ Number.isFinite(steps.api.outputs.json.value) }}  →  true if finite
{{ Number.isInteger(steps.api.outputs.json.value) }}  →  true if integer

// Constants
{{ Math.PI }}  →  3.141592653589793
{{ Math.E }}  →  2.718281828459045
```

### Array Operations

```
// Length
{{ steps["postgres-query"].outputs.rows.length }}  →  5

// First/Last
{{ steps["postgres-query"].outputs.rows[0].name }}  →  first
{{ steps["postgres-query"].outputs.rows[steps["postgres-query"].outputs.rows.length - 1].name }}  →  last
{{ steps["postgres-query"].outputs.rows.at(-1).name }}  →  last (simpler)
{{ steps["postgres-query"].outputs.rows.at(-2).name }}  →  second to last

// Check if contains
{{ steps["postgres-query"].outputs.rows.some(r => r.email === "joao@exemplo.com") }}  →  true if any
{{ steps["postgres-query"].outputs.rows.every(r => r.active) }}  →  true if all
{{ steps["postgres-query"].outputs.rows.includes("value") }}  →  true if contains exact value

// Search
{{ steps["postgres-query"].outputs.rows.find(r => r.id === 42) }}  →  first object that matches
{{ steps["postgres-query"].outputs.rows.findIndex(r => r.id === 42) }}  →  index of first match
{{ steps["postgres-query"].outputs.rows.indexOf("value") }}  →  index of primitive value

// Filter
{{ steps["postgres-query"].outputs.rows.filter(r => r.active) }}  →  active only
{{ steps["postgres-query"].outputs.rows.filter(r => r.age > 18) }}  →  over 18
{{ steps["postgres-query"].outputs.rows.filter(r => r.active).length }}  →  count active
{{ steps["postgres-query"].outputs.rows.filter(r => r.email.includes("@gmail.com")) }}  →  Gmail only

// Map (transform)
{{ steps["postgres-query"].outputs.rows.map(r => r.email) }}  →  ["email1", "email2", ...]
{{ steps["postgres-query"].outputs.rows.map(r => r.email).join(", ") }}  →  "email1, email2, ..."
{{ steps["postgres-query"].outputs.rows.map(r => r.name.toUpperCase()) }}  →  names in uppercase
{{ steps["postgres-query"].outputs.rows.map((r, i) => `${i + 1}. ${r.name}`) }}  →  numbered list

// Reduce (aggregate)
{{ steps["postgres-query"].outputs.rows.reduce((sum, r) => sum + r.price, 0) }}  →  sum of prices
{{ steps["postgres-query"].outputs.rows.reduce((max, r) => r.age > max ? r.age : max, 0) }}  →  maximum age
{{ steps["postgres-query"].outputs.rows.reduce((acc, r) => acc + r.name + ", ", "") }}  →  concatenate names

// Sort
{{ steps["postgres-query"].outputs.rows.sort((a, b) => a.name.localeCompare(b.name)) }}  →  alphabetical order
{{ steps["postgres-query"].outputs.rows.sort((a, b) => a.age - b.age) }}  →  ascending order by age
{{ steps["postgres-query"].outputs.rows.sort((a, b) => b.price - a.price) }}  →  descending order by price

// Reverse
{{ steps["postgres-query"].outputs.rows.reverse() }}  →  reverses order
{{ steps["postgres-query"].outputs.rows.slice().reverse() }}  →  reverses without modifying original

// Slice
{{ steps["postgres-query"].outputs.rows.slice(0, 10) }}  →  first 10
{{ steps["postgres-query"].outputs.rows.slice(-5) }}  →  last 5
{{ steps["postgres-query"].outputs.rows.slice(2, 5) }}  →  from index 2 to 4

// Concatenate
{{ steps.query1.outputs.rows.concat(steps.query2.outputs.rows) }}  →  joins two arrays
{{ [...steps.query1.outputs.rows, ...steps.query2.outputs.rows] }}  →  spread operator

// Flatten
{{ [[1, 2], [3, 4]].flat() }}  →  [1, 2, 3, 4]
{{ [[1, [2, [3]]]].flat(2) }}  →  [1, 2, 3] (2 levels)
{{ steps.api.outputs.json.data.flatMap(item => item.tags) }}  →  all tags from all items

// Join into string
{{ steps["postgres-query"].outputs.rows.map(r => r.name).join(", ") }}  →  "João, Maria, Pedro"
{{ steps["postgres-query"].outputs.rows.map(r => r.name).join(" | ") }}  →  "João | Maria | Pedro"

// Remove duplicates
{{ [...new Set(steps["postgres-query"].outputs.rows.map(r => r.city))] }}  →  unique cities

// Create array
{{ Array.from({length: 5}, (_, i) => i) }}  →  [0, 1, 2, 3, 4]
{{ Array.from({length: 5}, (_, i) => i + 1) }}  →  [1, 2, 3, 4, 5]
{{ Array(3).fill(0) }}  →  [0, 0, 0]
{{ Array(3).fill("x") }}  →  ["x", "x", "x"]

// Check if array
{{ Array.isArray(steps.api.outputs.json.data) }}  →  true
```

### Object Operations

```
// Deep access (node renamed to "api")
{{ steps.api.outputs.json.data.user.address.city }}

// Or using default label with brackets:
{{ steps["http-request"].outputs.json.data.user.address.city }}

// Optional chaining (avoids error if property does not exist)
{{ steps.api.outputs.json?.data?.user?.name }}  →  undefined if not exists
{{ steps.api.outputs.json?.data?.user?.name ?? "No name" }}  →  default value

// Keys
{{ Object.keys(steps.api.outputs.json) }}  →  ["id", "name", "email"]
{{ Object.keys(steps.api.outputs.json).length }}  →  number of properties
{{ Object.keys(steps.api.outputs.json).join(", ") }}  →  "id, name, email"

// Values
{{ Object.values(steps.api.outputs.json) }}  →  [42, "João", "joao@exemplo.com"]

// Entries
{{ Object.entries(steps.api.outputs.json) }}  →  [["id", 42], ["name", "João"], ...]
{{ Object.entries(steps.api.outputs.json).map(([k, v]) => `${k}: ${v}`).join(", ") }}

// Check property
{{ "email" in steps.api.outputs.json }}  →  true if property exists
{{ Object.hasOwn(steps.api.outputs.json, "email") }}  →  true if own property

// Merge objects
{{ Object.assign({}, steps.api.outputs.json, {newProp: "value"}) }}
{{ {...steps.api.outputs.json, newProp: "value"} }}  →  spread operator

// Stringify (convert to JSON string)
{{ JSON.stringify(steps.api.outputs.json) }}  →  '{"id":42,"name":"João"}'
{{ JSON.stringify(steps.api.outputs.json, null, 2) }}  →  formatted with indentation

// Parse (convert from JSON string)
{{ JSON.parse(steps.api.outputs.body) }}  →  JavaScript object
{{ JSON.parse(steps.api.outputs.body).data.user.name }}  →  access after parse

// Extract specific properties
{{ (({id, name}) => ({id, name}))(steps.api.outputs.json) }}  →  only id and name

// Count properties
{{ Object.keys(steps.api.outputs.json).length }}  →  number of fields

// Transform object into array
{{ Object.entries(steps.api.outputs.json).map(([k, v]) => ({key: k, value: v})) }}
```

### Conditionals (Ternary)

```
// Conditional value
{{ steps["http-request"].outputs.status === 200 ? "Success" : "Failure" }}

// Multiple conditions
{{ steps.api.outputs.json.age >= 18 ? "Adult" : "Minor" }}
{{ steps.api.outputs.json.score >= 90 ? "A" : steps.api.outputs.json.score >= 80 ? "B" : "C" }}

// With logical operators
{{ (steps.api.outputs.json.age >= 18 && steps.api.outputs.json.active) ? "Allowed" : "Denied" }}
{{ (steps.api.outputs.status === 200 || steps.api.outputs.status === 201) ? "OK" : "Error" }}

// Fallback for null/undefined
{{ steps["web-flow"].outputs.extracts.titulo || "No title" }}
{{ steps.api.outputs.json.name || "Anonymous" }}
{{ steps.api.outputs.json.config || {} }}  →  empty object if not exists

// Nullish coalescing (only null/undefined, not falsy values)
{{ steps.api.outputs.json.count ?? 0 }}  →  0 only if null/undefined
{{ steps.api.outputs.json.name ?? "No name" }}

// Existence check
{{ steps.api.outputs.json.email ? "Email: " + steps.api.outputs.json.email : "No email" }}

// Conditional formatting
{{ steps.api.outputs.json.price > 1000 ? "$ " + (steps.api.outputs.json.price / 1000).toFixed(1) + "k" : "$ " + steps.api.outputs.json.price }}
```

### Logical and Comparison Operations

```
// Comparison
{{ 5 > 3 }}  →  true
{{ 10 < 20 }}  →  true
{{ 5 >= 5 }}  →  true
{{ 3 <= 2 }}  →  false
{{ "abc" === "abc" }}  →  true (strict equality)
{{ 10 !== 5 }}  →  true (not equal)
{{ "5" == 5 }}  →  true (loose equality)
{{ "5" === 5 }}  →  false (different types)

// Logical operators
{{ true && false }}  →  false (AND)
{{ true || false }}  →  true (OR)
{{ !true }}  →  false (NOT)
{{ !!value }}  →  converts to boolean

// Comparisons with data
{{ steps.api.outputs.status === 200 }}  →  true/false
{{ steps.api.outputs.json.age > 18 }}  →  true/false
{{ steps["postgres-query"].outputs.rowCount > 0 }}  →  true/false
{{ steps.api.outputs.json.email.includes("@") }}  →  true/false

// Multiple conditions
{{ steps.api.outputs.status === 200 && steps.api.outputs.json.success }}
{{ steps.api.outputs.status >= 200 && steps.api.outputs.status < 300 }}
{{ steps.api.outputs.json.role === "admin" || steps.api.outputs.json.role === "moderator" }}

// Negation
{{ !steps.api.outputs.json.disabled }}  →  true if not disabled
{{ !(steps.api.outputs.status >= 400) }}  →  true if not an error
```

### Date and Time

```
// Current date
{{ new Date() }}  →  Date object
{{ Date.now() }}  →  timestamp in milliseconds
{{ new Date().toISOString() }}  →  "2024-01-15T10:30:00.000Z"
{{ new Date().toLocaleString("pt-BR") }}  →  "15/01/2024 10:30:00"
{{ new Date().toLocaleDateString("pt-BR") }}  →  "15/01/2024"
{{ new Date().toLocaleTimeString("pt-BR") }}  →  "10:30:00"

// Date components
{{ new Date().getFullYear() }}  →  2024
{{ new Date().getMonth() }}  →  0-11 (January = 0)
{{ new Date().getMonth() + 1 }}  →  1-12
{{ new Date().getDate() }}  →  1-31 (day of month)
{{ new Date().getDay() }}  →  0-6 (Sunday = 0)
{{ new Date().getHours() }}  →  0-23
{{ new Date().getMinutes() }}  →  0-59
{{ new Date().getSeconds() }}  →  0-59
{{ new Date().getMilliseconds() }}  →  0-999

// Create dates
{{ new Date("2024-01-15") }}  →  specific date
{{ new Date(2024, 0, 15) }}  →  January 15, 2024
{{ new Date(2024, 0, 15, 10, 30, 0) }}  →  with time
{{ new Date(Date.now() + 86400000) }}  →  tomorrow (+ 1 day in ms)

// Date calculations
{{ new Date().getTime() + (24 * 60 * 60 * 1000) }}  →  tomorrow's timestamp
{{ new Date().getTime() - (7 * 24 * 60 * 60 * 1000) }}  →  timestamp 7 days ago
{{ Math.floor((new Date("2024-12-31") - new Date()) / (1000 * 60 * 60 * 24)) }}  →  days until end of year

// Custom formatting
{{ `${new Date().getDate()}/${new Date().getMonth() + 1}/${new Date().getFullYear()}` }}  →  "15/1/2024"
{{ `${new Date().getHours()}:${new Date().getMinutes().toString().padStart(2, "0")}` }}  →  "10:05"

// Date comparison
{{ new Date(steps.api.outputs.json.expiresAt) > new Date() }}  →  true if not expired
{{ new Date(steps.api.outputs.json.createdAt).getTime() > Date.now() - 86400000 }}  →  created in the last 24h
```

### Type Conversion

```
// To String
{{ String(123) }}  →  "123"
{{ String(true) }}  →  "true"
{{ String(null) }}  →  "null"
{{ (123).toString() }}  →  "123"
{{ (123).toString(16) }}  →  "7b" (hexadecimal)

// To Number
{{ Number("123") }}  →  123
{{ Number("12.5") }}  →  12.5
{{ Number(true) }}  →  1
{{ Number(false) }}  →  0
{{ parseInt("123") }}  →  123
{{ parseInt("123.45") }}  →  123
{{ parseFloat("123.45") }}  →  123.45
{{ parseInt("ff", 16) }}  →  255 (hexadecimal)
{{ parseInt("1010", 2) }}  →  10 (binary)

// To Boolean
{{ Boolean(1) }}  →  true
{{ Boolean(0) }}  →  false
{{ Boolean("") }}  →  false
{{ Boolean("text") }}  →  true
{{ Boolean(null) }}  →  false
{{ Boolean(undefined) }}  →  false
{{ !!value }}  →  converts to boolean

// Type check
{{ typeof "text" }}  →  "string"
{{ typeof 123 }}  →  "number"
{{ typeof true }}  →  "boolean"
{{ typeof {} }}  →  "object"
{{ typeof [] }}  →  "object"
{{ typeof null }}  →  "object" (JS quirk)
{{ typeof undefined }}  →  "undefined"
{{ Array.isArray([1, 2]) }}  →  true
{{ Array.isArray("text") }}  →  false
```

### Regular Expressions (Regex)

```
// Test pattern
{{ /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(steps.api.outputs.json.email) }}  →  validates email
{{ /^\d{3}\.\d{3}\.\d{3}-\d{2}$/.test(steps.api.outputs.json.cpf) }}  →  validates formatted CPF
{{ /^[0-9]+$/.test(steps.api.outputs.json.code) }}  →  digits only
{{ /^[A-Z]/.test(steps.api.outputs.json.name) }}  →  starts with uppercase

// Extract with match
{{ steps.api.outputs.json.text.match(/\d+/g) }}  →  all numbers
{{ steps.api.outputs.json.url.match(/https?:\/\/([^\/]+)/)[1] }}  →  extract domain

// Replace with regex
{{ steps.api.outputs.json.phone.replace(/\D/g, "") }}  →  remove non-digits
{{ steps.api.outputs.json.cpf.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, "$1.$2.$3-$4") }}  →  format CPF
```

---

## Advanced Practical Examples

### Validations

```
// Valid email
{{ /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(variables.email) }}

// Formatted CPF
{{ /^\d{3}\.\d{3}\.\d{3}-\d{2}$/.test(variables.cpf) }}

// Brazilian phone number
{{ /^\(\d{2}\) \d{4,5}-\d{4}$/.test(variables.telefone) }}

// Postal code (CEP)
{{ /^\d{5}-\d{3}$/.test(variables.cep) }}

// Strong password (minimum 8 characters, uppercase, lowercase, number)
{{ variables.senha.length >= 8 && /[A-Z]/.test(variables.senha) && /[a-z]/.test(variables.senha) && /[0-9]/.test(variables.senha) }}

// Valid URL
{{ /^https?:\/\/.+/.test(variables.url) }}

// Letters only
{{ /^[A-Za-zÀ-ÿ\s]+$/.test(variables.nome) }}

// Numbers only
{{ /^[0-9]+$/.test(variables.codigo) }}
```

### Formatting

```
// CPF
{{ variables.cpf.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, "$1.$2.$3-$4") }}  →  "123.456.789-00"

// CNPJ
{{ variables.cnpj.replace(/(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})/, "$1.$2.$3/$4-$5") }}  →  "12.345.678/0001-90"

// Phone
{{ variables.telefone.replace(/(\d{2})(\d{5})(\d{4})/, "($1) $2-$3") }}  →  "(11) 98765-4321"

// Postal code (CEP)
{{ variables.cep.replace(/(\d{5})(\d{3})/, "$1-$2") }}  →  "12345-678"

// Currency
{{ "R$ " + variables.preco.toFixed(2).replace(".", ",") }}  →  "R$ 10,50"
{{ "R$ " + variables.preco.toLocaleString("pt-BR", {minimumFractionDigits: 2}) }}  →  "R$ 1.234,56"

// Percentage
{{ (variables.valor * 100).toFixed(1) + "%" }}  →  "85.5%"
{{ Math.round(variables.acertos / variables.total * 100) + "%" }}  →  "85%"

// Number with thousands separator
{{ variables.numero.toLocaleString("pt-BR") }}  →  "1.234.567"

// First letter uppercase
{{ variables.nome.charAt(0).toUpperCase() + variables.nome.slice(1).toLowerCase() }}  →  "João"

// Title case (each word capitalized)
{{ variables.titulo.split(" ").map(w => w.charAt(0).toUpperCase() + w.slice(1).toLowerCase()).join(" ") }}
```

### Complex Manipulations

```
// Extract email domain
{{ steps.api.outputs.json.email.split("@")[1] }}  →  "exemplo.com"

// Extract URL domain
{{ new URL(variables.url).hostname }}  →  "www.exemplo.com"

// Name initials
{{ variables.nome.split(" ").map(n => n[0]).join("") }}  →  "JS" (João Silva)

// Word count
{{ variables.texto.trim().split(/\s+/).length }}  →  number of words

// Character count without spaces
{{ variables.texto.replace(/\s/g, "").length }}

// Truncate text
{{ variables.texto.length > 50 ? variables.texto.substring(0, 50) + "..." : variables.texto }}

// Remove accents
{{ variables.texto.normalize("NFD").replace(/[\u0300-\u036f]/g, "") }}

// Slug (URL-friendly)
{{ variables.titulo.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "").replace(/[^a-z0-9]+/g, "-").replace(/^-|-$/g, "") }}

// Generate unique ID
{{ Date.now() + "-" + Math.random().toString(36).substr(2, 9) }}  →  "1705320000000-k7j3h2g1f"

// Simple hash (non-cryptographic)
{{ variables.texto.split("").reduce((hash, char) => ((hash << 5) - hash) + char.charCodeAt(0), 0) }}

// Reverse string
{{ variables.texto.split("").reverse().join("") }}

// Remove duplicate characters from string
{{ [...new Set(variables.texto.split(""))].join("") }}

// Capitalize each word
{{ variables.frase.replace(/\b\w/g, l => l.toUpperCase()) }}
```

### Calculations and Statistics

```
// Average
{{ variables.notas.reduce((a, b) => a + b, 0) / variables.notas.length }}

// Median
{{ variables.notas.sort((a, b) => a - b)[Math.floor(variables.notas.length / 2)] }}

// Sum
{{ variables.valores.reduce((a, b) => a + b, 0) }}

// Maximum
{{ Math.max(...variables.valores) }}

// Minimum
{{ Math.min(...variables.valores) }}

// Standard deviation (simplified)
{{ Math.sqrt(variables.valores.map(x => Math.pow(x - variables.valores.reduce((a,b) => a+b) / variables.valores.length, 2)).reduce((a,b) => a+b) / variables.valores.length) }}

// Growth percentage
{{ ((variables.valorNovo - variables.valorAntigo) / variables.valorAntigo * 100).toFixed(1) + "%" }}

// Rule of three
{{ (variables.valor * variables.proporcao / 100) }}

// Simple interest
{{ variables.capital * variables.taxa * variables.tempo / 100 }}

// Days between dates
{{ Math.floor((new Date(variables.dataFim) - new Date(variables.dataInicio)) / (1000 * 60 * 60 * 24)) }}

// Age from birth date
{{ Math.floor((new Date() - new Date(variables.dataNascimento)) / (1000 * 60 * 60 * 24 * 365.25)) }}

// BMI (Body Mass Index)
{{ (variables.peso / Math.pow(variables.altura, 2)).toFixed(1) }}
```

### Groupings and Transformations

```
// Group by property
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.categoria] = acc[item.categoria] || [];
  acc[item.categoria].push(item);
  return acc;
}, {}) }}

// Count occurrences
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.status] = (acc[item.status] || 0) + 1;
  return acc;
}, {}) }}

// Sum by group
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.categoria] = (acc[item.categoria] || 0) + item.valor;
  return acc;
}, {}) }}

// Remove duplicates
{{ [...new Set(steps.query.outputs.rows.map(r => r.email))] }}

// Sort objects by multiple fields
{{ steps.query.outputs.rows.sort((a, b) => a.categoria.localeCompare(b.categoria) || a.nome.localeCompare(b.nome)) }}

// Filter and map in chain
{{ steps.query.outputs.rows.filter(r => r.ativo).map(r => r.nome).sort().join(", ") }}

// Create lookup/index
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.id] = item;
  return acc;
}, {}) }}

// Flatten array of arrays
{{ steps.api.outputs.json.data.flatMap(item => item.tags) }}

// Partition array (even and odd indices)
{{ steps.query.outputs.rows.reduce((acc, item, i) => {
  acc[i % 2 === 0 ? "pares" : "impares"].push(item);
  return acc;
}, {pares: [], impares: []}) }}
```

---

## Usage in Nodes

### Navigation URL

```
{{ variables.BASE_URL }}/login
{{ variables.BASE_URL }}/users/{{ steps["http-request"].outputs.json.id }}
{{ variables.BASE_URL }}/products?category={{ steps.api.outputs.json.category }}&page={{ variables.page }}
```

### Text for Fill/Type

```
{{ variables.TEST_EMAIL }}
{{ steps["custom-js"].outputs.result.email }}
test-{{ Date.now() }}@exemplo.com
{{ variables.FIRST_NAME }} {{ variables.LAST_NAME }}
```

### SQL with Dynamic Data

```sql
SELECT * FROM orders WHERE user_id = '{{ steps["http-request"].outputs.json.userId }}'

SELECT * FROM products WHERE category = '{{ variables.CATEGORY }}' AND price > {{ variables.MIN_PRICE }}

INSERT INTO logs (user_id, action, timestamp)
VALUES ('{{ steps.login.outputs.json.userId }}', 'login', '{{ new Date().toISOString() }}')
```

> **Caution:** Use parameters ($1, ?) instead of direct interpolation to prevent SQL injection where possible.

### HTTP Request Body

```json
{
  "token": "{{ steps.login.outputs.json.token }}",
  "data": {
    "name": "{{ variables.TEST_NAME }}",
    "email": "{{ steps["custom-js"].outputs.result.email }}",
    "timestamp": "{{ new Date().toISOString() }}"
  }
}
```

> In the example above, `steps.login` is an HTTP Request node renamed to "login".

### HTTP Request Headers

```
Authorization: Bearer {{ steps.login.outputs.json.token }}
X-User-ID: {{ steps.api.outputs.json.userId }}
X-Request-ID: {{ Date.now() }}-{{ Math.random().toString(36).substr(2, 9) }}
```

### If Condition

```
{{ steps["http-request"].outputs.status === 200 }}
{{ steps["postgres-query"].outputs.rowCount > 0 }}
{{ steps["web-flow"].outputs.extracts.price < variables.MAX_PRICE }}
{{ steps.api.outputs.json.role === "admin" || steps.api.outputs.json.role === "moderator" }}
{{ steps.api.outputs.json.age >= 18 && steps.api.outputs.json.active }}
```

### Switch Expression

```
{{ steps.api.outputs.status }}
{{ steps.api.outputs.json.userType }}
{{ Math.floor(steps.api.outputs.json.score / 10) }}
```

### Loop Items (For Each)

```
{{ steps["postgres-query"].outputs.rows }}
{{ steps.api.outputs.json.users }}
{{ Array.from({length: 10}, (_, i) => i + 1) }}
{{ steps.api.outputs.json.data.filter(item => item.active) }}
```

### Loop Condition (While)

```
{{ variables.contador < 10 }}
{{ steps.api.outputs.json.hasMore }}
{{ steps["postgres-query"].outputs.rowCount > 0 }}
{{ variables.tentativas < variables.MAX_TENTATIVAS && !variables.sucesso }}
```

### Log Message

```
User {{ steps["postgres-query"].outputs.rows[0].name }} has {{ steps["postgres-query"].outputs.rows[0].orders }} orders

API responded with status {{ steps["http-request"].outputs.status }}: {{ steps["http-request"].outputs.json.message }}

Processed {{ variables._loopIndex + 1 }} of {{ steps.loop.outputs._loopCount }} items

Total: $ {{ steps.query.outputs.rows.reduce((sum, r) => sum + r.price, 0).toFixed(2) }}
```

### Set Variable

```
// Variable name: userId
// Value:
{{ steps.login.outputs.json.user.id }}

// Variable name: fullName
// Value:
{{ steps.api.outputs.json.firstName + " " + steps.api.outputs.json.lastName }}

// Variable name: processedData
// Value:
{{ steps.query.outputs.rows.map(r => ({id: r.id, name: r.name.toUpperCase()})) }}

// Variable name: timestamp
// Value:
{{ new Date().toISOString() }}
```

---

## Loops and Special Variables

When a node is inside a loop, you have access to special variables:

```
{{ variables._loopItem }}      // Current iteration item
{{ variables._loopIndex }}     // Current index (starts at 0)
```

You can also access them via the loop node's outputs:

```
{{ steps.loop.outputs._loopItem }}
{{ steps.loop.outputs._loopIndex }}
{{ steps.loop.outputs._loopCount }}  // Total iterations (available after loop)
```

### Examples in Loops

```
// Message with progress
{{ "Processing item " + (variables._loopIndex + 1) + " of " + steps.loop.outputs._loopCount }}

// Access property of current item
{{ variables._loopItem.nome }}
{{ variables._loopItem.preco * 1.1 }}

// Use index in calculations
{{ "Item #" + (variables._loopIndex + 1) + ": " + variables._loopItem.name }}

// Condition based on index
{{ variables._loopIndex === 0 ? "First" : variables._loopIndex === steps.loop.outputs._loopCount - 1 ? "Last" : "Middle" }}

// Use item in URL
{{ variables.BASE_URL }}/api/users/{{ variables._loopItem.id }}

// Use item in SQL
SELECT * FROM orders WHERE user_id = '{{ variables._loopItem.userId }}'
```

---

## Error Handling

If an expression fails (property does not exist, incorrect type, etc.), the result is replaced with an error marker:

```
[Error: Cannot read property 'name' of undefined]
```

To prevent errors, use the optional chaining operator or fallbacks:

```
// Optional chaining (node renamed to "api")
{{ steps.api.outputs.json?.user?.name }}

// Or with default label:
{{ steps["http-request"].outputs.json?.user?.name }}

// Fallback with OR
{{ steps["http-request"].outputs.json.name || "N/A" }}

// Fallback with nullish coalescing
{{ steps.api.outputs.json.count ?? 0 }}

// Check before accessing
{{ steps.api.outputs.json.user ? steps.api.outputs.json.user.name : "No user" }}

// Check for empty array
{{ steps.query.outputs.rows.length > 0 ? steps.query.outputs.rows[0].name : "No results" }}
```

> **Important:** If an expression references a node or variable that does not exist (e.g., a typo), the node will fail with a clear error message indicating which expression failed, rather than causing unexpected behavior.

---

## Security

The expression system runs in a **sandboxed environment** with restrictions:

- **No access** to `require`, `import`, `process`
- **No access** to the file system
- **No access** to the network (use the HTTP Request node for that)
- Only `steps` and `variables` are available in the context

---

## Tips and Best Practices

### 1. Rename Nodes with Simple Labels

```
// ❌ Hard to read
{{ steps["http-request"].outputs.json.token }}

// ✅ Cleaner (rename the node to "login")
{{ steps.login.outputs.json.token }}
```

### 2. Use Optional Chaining to Avoid Errors

```
// ❌ May throw error if user does not exist
{{ steps.api.outputs.json.user.name }}

// ✅ Returns undefined if not exists
{{ steps.api.outputs.json?.user?.name }}

// ✅ With default value
{{ steps.api.outputs.json?.user?.name ?? "No name" }}
```

### 3. Use Interpolation to Build URLs and Messages

```
// ✅ Dynamic URL
{{ variables.BASE_URL }}/api/users/{{ steps.api.outputs.json.id }}

// ✅ Formatted message
Status: {{ steps.api.outputs.status }} - {{ steps.api.outputs.json.message }}
```

### 4. Use Single Expression to Preserve Types

```
// ✅ Returns array (single expression)
{{ steps.query.outputs.rows }}

// ❌ Returns string (interpolation)
Rows: {{ steps.query.outputs.rows }}
```

### 5. Add a Log Node for Debugging

```
// Log to inspect values
{{ JSON.stringify(steps.api.outputs.json, null, 2) }}

// Log variables
Variables: {{ JSON.stringify(variables) }}

// Log loop item
Current item: {{ JSON.stringify(variables._loopItem) }}
```

### 6. Use Fallbacks for Optional Data

```
{{ steps.api.outputs.json.name || "Anonymous" }}
{{ steps.api.outputs.json.config || {} }}
{{ steps.api.outputs.json.items || [] }}
```

### 7. For Complex Logic, Use Custom JavaScript

If an expression is getting too complex, consider:
- Using a **Custom JavaScript** node to process the data
- Splitting the logic into multiple **Set Variable** nodes
- Using control nodes (If, Switch, Loop)

```
// ❌ Too complex expression
{{ steps.query.outputs.rows.filter(r => r.active && r.age > 18).map(r => ({...r, fullName: r.firstName + " " + r.lastName})).sort((a, b) => a.fullName.localeCompare(b.fullName)) }}

// ✅ Use Custom JavaScript
// In the Custom JavaScript node:
const activeAdults = steps.query.outputs.rows
  .filter(r => r.active && r.age > 18)
  .map(r => ({...r, fullName: `${r.firstName} ${r.lastName}`}))
  .sort((a, b) => a.fullName.localeCompare(b.fullName));

return { processedUsers: activeAdults };

// Then reference:
{{ steps.transform.outputs.result.processedUsers }}
```

### 8. Convert Types When Necessary

```
// ❌ Unintended concatenation
{{ "10" + "5" }}  // "105"

// ✅ Numeric sum
{{ Number("10") + Number("5") }}  // 15
{{ parseInt("10") + parseInt("5") }}  // 15
```

### 9. Validate Before Using

```
{{ Array.isArray(variables.lista) && variables.lista.length > 0 ? variables.lista[0] : "Empty list" }}

{{ typeof steps.api.outputs.json.data === "object" ? JSON.stringify(steps.api.outputs.json.data) : "Invalid data" }}
```

### 10. Use Template Strings for Readability

```
// ❌ Hard to read
{{ "Name: " + variables.nome + ", Age: " + variables.idade + ", Email: " + variables.email }}

// ✅ More readable
{{ `Name: ${variables.nome}, Age: ${variables.idade}, Email: ${variables.email}` }}
```

---

## Complete Examples

### Example 1: API Authentication Flow

```
Step 1 (http-request): Login
  URL: https://api.example.com/login
  Body: { "user": "admin", "pass": "secret" }

Step 2 (set-variable): Store Token
  Key: authToken
  Value: {{ steps.login.outputs.json.token }}

Step 3 (http-request): Get User Data
  URL: https://api.example.com/user
  Headers: { "Authorization": "Bearer {{ variables.authToken }}" }
```

### Example 2: Database Query with Conditional Logic

```
Step 1 (postgres-query): Get Users
  SQL: SELECT * FROM users WHERE active = true

Step 2 (if): Check if users exist
  Condition: {{ steps["postgres-query"].outputs.rowCount > 0 }}

Step 3 (loop): Process each user (if true branch)
  Array: {{ steps["postgres-query"].outputs.rows }}

Step 4 (http-request): Send notification (inside loop)
  URL: https://api.example.com/notify
  Body: { "email": "{{ variables._loopItem.email }}" }
```

### Example 3: Web Scraping with Data Extraction

```
Step 1 (web-flow): Extract product data
  Navigate: https://example.com/products
  Extract: productName (selector: .product-title)
  Extract: productPrice (selector: .price)

Step 2 (set-variable): Store product info
  Key: product
  Value: {{ { name: steps["web-flow"].outputs.extracts.productName, price: steps["web-flow"].outputs.extracts.productPrice } }}

Step 3 (postgres-query): Save to database
  SQL: INSERT INTO products (name, price) VALUES ($1, $2)
  Params: {{ [variables.product.name, variables.product.price] }}
```

---

## Limitations

- Expressions are evaluated in a secure sandbox environment
- Browser APIs cannot be accessed (window, document, etc.)
- HTTP requests cannot be made directly (use dedicated nodes)
- External modules cannot be imported
- Asynchronous functions (async/await) are not supported in expressions
- The global context or variables of other nodes cannot be modified

---

## Conclusion

QANode's expression system puts the full power of JavaScript at your fingertips for creating dynamic and intelligent flows. With `{{ }}` syntax, you can:

- Access data from previous nodes and variables
- Perform calculations and complex transformations
- Validate and format data
- Create conditional logic
- Manipulate strings, arrays, and objects
- Work with dates and numbers

Always remember to use `{{ }}` to delimit your expressions and leverage the full power of JavaScript to create robust and efficient tests!
