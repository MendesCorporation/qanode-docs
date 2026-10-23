# Load Test Node

The **Load Test** node runs load and performance tests against HTTP endpoints, simulating multiple virtual users (VUs) in parallel. It generates PNG visual evidence with detailed metrics and supports automatic pass/fail criteria via thresholds.

---

## Overview

[Overview](../../assets/images/nos-load-test-visao-geral.mp4)

| Property | Value |
|----------|-------|
| **Type** | `load-test` |
| **Category** | Performance |
| **Color** | 🟫 Amber (#92400E) |
| **Input** | `in` |
| **Output** | `out` |

---

## Test Types

Each test type automatically generates a different load profile from the **VUs** and **Duration** fields.

| Type | Description | When to Use |
|------|-------------|-------------|
| **Smoke** | 1 VU, configured duration | Validate that the endpoint responds before running larger tests |
| **Load** | Ramp up → sustain → ramp down | Verify behavior under expected normal load |
| **Stress** | Aggressive ramp to the limit | Identify the point where the system starts to degrade |
| **Spike** | Sudden VU spike | Test reaction to sudden traffic spikes (e.g., flash sales) |
| **Soak** | Sustained load over long duration | Detect memory leaks and gradual degradation |
| **Breakpoint** | Progressive VU staircase | Find the exact breaking point of the system |

---

## Configuration

### Request

| Field | Type | Description |
|-------|------|-------------|
| **Credential** | `string` | Saved HTTP credential (auto-fills URL and auth) |
| **Method** | `string` | `GET`, `QUERY`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS` |
| **URL** | `string` | Target endpoint (supports `{{ }}`) |
| **Auth** | `string` | Manual authentication type (if not using a credential) |
| **Headers** | `object` | Additional request headers |
| **Query Parameters** | `array` | Key-value pairs added to the URL |
| **Body Type** | `string` | `None`, `JSON`, `Raw text`, `x-www-form-urlencoded`, `multipart/form-data`, or `Binary file` |
| **Body / Fields / File** | `any` | Content sent according to the selected body type |

### Request Body

The Load Test request builder follows the same model as the [HTTP Request](../api/http-request.md) node:

| Type | When to use |
|------|-------------|
| **None** | Requests without a body |
| **JSON** | REST APIs that receive JSON objects or arrays |
| **Raw text** | XML, SOAP, textual payloads, or custom formats |
| **x-www-form-urlencoded** | URL-encoded forms |
| **multipart/form-data** | Forms with text fields and files |
| **Binary file** | Direct upload of a `fileRef` as the request body |

Header and body fields accept `{{ }}` expressions. To send a file, use a `fileRef` produced by another node:

```
File: {{ steps["file-generate"].outputs.fileRef }}
```

You can also import a `curl` command; QANode tries to convert the method, URL, headers, query, body, and form fields to the matching visual mode.

### Dynamic Data per Request

The **Dynamic data** section generates fresh values immediately before every request made by each virtual user. Use it when an endpoint requires a unique email, external identifier, sequence number, future date, or any value that must not be repeated during the test.

Create a value under **Dynamic data** and reference it with:

```text
{{ load.valueName }}
```

Dynamic values can be used in:

- the URL and query parameters;
- header names and values;
- JSON or raw-text bodies;
- `x-www-form-urlencoded` keys and values;
- names and values of text fields in `multipart/form-data`.

Binary file contents are not changed. In multipart requests, only the field name and text fields accept dynamic data.

#### Built-in values

These values are available automatically and do not need to be created:

| Expression | Value |
|------------|-------|
| `{{ load.runId }}` | QANode execution ID |
| `{{ load.requestId }}` | Global request sequence, starting at 1 and unique across all VUs |
| `{{ load.vuId }}` | Virtual user identifier |
| `{{ load.iteration }}` | Iteration number within that VU |
| `{{ load.timestamp }}` | ISO timestamp generated at request time |

The same value is reused everywhere within one request. Generated values are refreshed for the next request.

Example:

```text
URL: https://api.example.com/orders/{{ load.requestId }}
Header X-Request-Id: {{ load.requestId }}
Body: { "requestId": "{{ load.requestId }}" }
```

The URL, header, and body receive the same `requestId` for that call.

#### Available types

| Type | Configuration | Result |
|------|---------------|--------|
| **UUID** | Name only | New UUID v4 for every request |
| **Email** | Prefix and domain | Unique email containing the execution token and `requestId` |
| **Sequence** | Start and increment | `start + (requestId - 1) × increment` |
| **Random number** | Minimum, maximum, and decimal places | Random number within the range |
| **List** | One value per line; random or sequential selection | One list item per request |
| **Timestamp** | Format, offset, and unit | Current, past, or future date |
| **Template** | Text with other `load` expressions | Value composed after the other dynamic values are generated |

##### UUID

```text
Name: externalId
Type: UUID
Use: {{ load.externalId }}
```

##### Email

```text
Name: customerEmail
Type: Email
Prefix: performance
Domain: example.com
Use: {{ load.customerEmail }}
```

The result follows this pattern and does not repeat within the run:

```text
performance_executionToken_142@example.com
```

Use a reserved domain or a domain controlled by your company. Do not use real addresses in a test that may send email.

##### Sequence

```text
Name: customerNumber
Start: 1000
Increment: 5
```

The first requests produce `1000`, `1005`, `1010`, and so on, regardless of which VU executes them.

##### Random number

```text
Name: amount
Minimum: 10
Maximum: 100
Decimal places: 2
```

From 0 to 10 decimal places are supported. The maximum must be greater than or equal to the minimum.

##### List

```text
Name: region
Values:
  south
  north
  east
Selection: Sequential
```

In **Sequential** mode, items cycle according to `requestId`, which helps distribute load evenly. In **Random** mode, any item can be selected for each request.

##### Timestamp

Available formats:

| Format | Example |
|--------|---------|
| **ISO** | `2026-07-21T18:30:00.000Z` |
| **Unix** | `1784658600` in seconds |
| **Unix ms** | `1784658600000` in milliseconds |

The offset supports seconds, minutes, hours, or days. Use a positive number for the future and a negative number for the past.

```text
Name: expiresAt
Format: ISO
Offset: 2
Unit: Hours
```

##### Template

Templates are evaluated after the other dynamic data and can combine their values:

```text
Name: customerKey
Template: customer-{{ load.customerNumber }}-{{ load.externalId }}
Use: {{ load.customerKey }}
```

#### Complete example — unique user creation

Create these values:

| Name | Type | Configuration |
|------|------|---------------|
| `email` | Email | prefix `performance`, domain `example.com` |
| `externalId` | UUID | — |
| `plan` | List | `basic`, `pro`, sequential selection |
| `sequence` | Sequence | start 1, increment 1 |

Use them in the body:

```json
{
  "email": "{{ load.email }}",
  "externalId": "{{ load.externalId }}",
  "plan": "{{ load.plan }}",
  "sequence": "{{ load.sequence }}",
  "requestId": "{{ load.requestId }}",
  "createdAt": "{{ load.timestamp }}"
}
```

Every call creates a different payload, even when multiple VUs run concurrently.

#### Naming rules

- names must start with a letter;
- they may contain letters, numbers, `_`, or `$`;
- names must be unique;
- reserved names cannot be used: `runId`, `requestId`, `vuId`, `iteration`, and `timestamp`.

QANode validates unknown references before starting the load. If the body uses `{{ load.customer }}` without a `customer` definition, the execution fails with a configuration message instead of sending incorrect requests.

#### Best practices

- use a sequence when distribution must be predictable;
- use a sequential list to balance regions, plans, or categories;
- use UUID or email when the API requires uniqueness;
- plan cleanup for records created by the test;
- remember that dynamic data changes inputs but does not replace thresholds and metric analysis;
- run a Smoke Test before the full load to validate the generated format.

### Load Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **VUs** | `number` | `10` | Number of concurrent virtual users |
| **Duration (s)** | `number` | `30` | Total test duration in seconds |
| **Think Time (ms)** | `number` | `0` | Pause between requests per VU |
| **Timeout (ms)** | `number` or expression | `30000` | Maximum wait time for each request's response |

> For **Smoke**, the VUs field is fixed at 1 and not displayed.  
> For **Soak**, the default duration is 1800s (30 min).  
> For **Breakpoint**, VUs represents the **maximum** VUs to be reached.

**Timeout (ms)** accepts **1 to 86,400,000 ms (24 hours)** per request. It does not change the configured total test duration. Numeric load settings can also use expressions such as `{{ variables.LOAD_VUS }}` or `{{ variables.REQUEST_TIMEOUT_MS }}`; the result must be numeric and stay within each field's limits.

#### How Breakpoint works

The test divides the total duration into ~30s steps, progressively increasing VUs:

```
VUs: 10 | Duration: 60s  →  2 steps of 30s
  Step 1: 0s → 30s  →  5 VUs
  Step 2: 30s → 60s →  10 VUs
```

### Custom Stages

Enable **Custom** in the Stages section to manually define the load profile:

| Field | Description |
|-------|-------------|
| **Duration (s)** | Duration of this stage in seconds |
| **Target VUs** | Number of VUs at the end of this stage |

Example stages for a manual stress test:

| Duration | Target VUs | Description |
|----------|-----------|-------------|
| 30s | 10 | Initial ramp up |
| 60s | 50 | Sustained load |
| 30s | 100 | Stress peak |
| 15s | 0 | Ramp down |

---

## Authentication

### Using Saved Credentials

Select an **HTTP/API** credential. The base URL and authentication data are applied automatically:

1. Select the credential in the **Credential** field
2. The base URL is automatically filled in the **URL** field
3. Complete with the endpoint path: `/api/checkout`

### Manual Authentication

| Type | Fields | Result |
|------|--------|--------|
| **Bearer Token** | Token | Header `Authorization: Bearer {token}` |
| **Basic Auth** | Username + Password | Header `Authorization: Basic {base64}` |
| **API Key** | Header Name + Token | Custom header with the token |

---

## Thresholds

Thresholds define automatic pass/fail criteria. If any threshold is not met, the node is marked as **FAILED**.

| Metric | Description |
|--------|-------------|
| `p50` | 50th percentile latency (ms) |
| `p95` | 95th percentile latency (ms) |
| `p99` | 99th percentile latency (ms) |
| `avgDuration` | Average latency (ms) |
| `errorRate` | Error rate (%) |
| `rps` | Requests per second |

| Operator | Meaning |
|----------|---------|
| `<` | Less than |
| `≤` | Less than or equal |
| `>` | Greater than |
| `≥` | Greater than or equal |

**Common threshold examples:**

| Threshold | Meaning |
|-----------|---------|
| `p95 < 500` | 95% of responses under 500ms |
| `errorRate < 1` | Error rate below 1% |
| `rps > 10` | Minimum 10 requests per second |
| `p99 < 2000` | 99% of responses under 2s |

> Without configured thresholds, the node always passes (as long as the endpoint responds).

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `passed` | `boolean` | `true` if all thresholds were met |
| `testType` | `string` | Test type executed |
| `metrics` | `object` | Consolidated test metrics |
| `thresholds` | `array` | Result of each configured threshold |
| `stages` | `array` | Executed stages (auto or custom) |

### `metrics` object structure

```json
{
  "totalRequests": 1141,
  "errorCount": 0,
  "errorRate": 0.0,
  "rps": 34.49,
  "avgDuration": 212,
  "minDuration": 98,
  "maxDuration": 668,
  "p50": 182,
  "p90": 332,
  "p95": 451,
  "p99": 579
}
```

### Accessing Outputs

```
// Check if passed
{{ steps["load-test"].outputs.passed }}  →  true

// Total requests
{{ steps["load-test"].outputs.metrics.totalRequests }}  →  1141

// p95 latency
{{ steps["load-test"].outputs.metrics.p95 }}  →  451

// Error rate
{{ steps["load-test"].outputs.metrics.errorRate }}  →  0.0

// RPS
{{ steps["load-test"].outputs.metrics.rps }}  →  34.49
```

---

## Generated Evidence

The node automatically generates **two PNG charts** as execution evidence:

### 1. Summary Report (`load-test-report.png`)

Consolidated view with:
- Metric cards (p50, p95, p99, Avg, RPS, Requests, Errors, Error Rate)
- Horizontal bar chart with latency distribution
- Throughput over time chart (req/s)
- Status pills for each configured threshold
- Footer with executed stages

### 2. RPS × Latency Timeline (`load-test-timeline.png`)

Dual-axis chart with:
- **Blue bars** (left axis): RPS over time
- **Orange line** (right axis): Average latency over time
- **Purple dashed line** (right axis): p95 latency over time

> The timeline chart is especially useful for **Breakpoint** and **Stress** tests, where you can visually identify the exact moment latency starts rising in response to increased load.

---

## Practical Examples

### Smoke test — Basic validation

```
Type: Smoke
URL: https://api.example.com/health
Method: GET
Duration: 10s
```

Ideal to run at the start of a test flow — ensures the environment is up.

### Load test — Normal load with thresholds

```
Type: Load
URL: https://api.example.com/products
Method: GET
VUs: 20
Duration: 60s
Thresholds:
  - p95 < 500
  - errorRate < 1
```

### Stress test — System limits

```
Type: Stress
URL: https://api.example.com/checkout
Method: POST
VUs: 100
Duration: 120s
Body: { "productId": "123", "quantity": 1 }
Auth: Bearer → {{ variables.API_TOKEN }}
Thresholds:
  - p99 < 2000
  - errorRate < 5
```

### Breakpoint — Finding the breaking point

```
Type: Breakpoint
URL: https://api.example.com/search
Method: GET
Max VUs: 200
Duration: 300s
Thresholds:
  - p95 < 1000
  - errorRate < 2
```

The system will progressively increase from 1 to 200 VUs in ~30s steps. When p95 exceeds 1000ms or errors surpass 2%, the node marks as failed — indicating the breaking point.

### Chaining with other nodes

```
[Load Test: Smoke]
    │ outputs.passed = true
    ▼
[If: {{ steps.smoke.outputs.passed }}]
    │ true  → [Load Test: Full load]
    │ false → [Log: "Smoke failed — environment unavailable"]
```

---

## Queue Isolation

The Load Test node runs in a separate queue (`qanode-load-tests`) to avoid interfering with other running flows.

To configure a dedicated Load Test worker:

```bash
WORKER_QUEUES=load-tests node dist/start.js
```

For a worker that processes both queues:

```bash
WORKER_QUEUES=executions,load-tests node dist/start.js
```

---

## Tips

- **Start with Smoke** before running load tests — ensures the endpoint is responding correctly
- **Configure thresholds** so the test fails automatically when the system degrades, without having to analyze numbers manually
- **Use the timeline chart** to identify the exact moment of degradation in Breakpoint and Stress tests
- **Think Time** simulates human behavior — useful for Soak tests where you want continuous but realistic load
- **Saved credentials** make it easy to run in different environments (staging, production) without changing the flow
- For reliable tests, **up to 100 VUs** per worker instance. Above that, consider a dedicated worker
