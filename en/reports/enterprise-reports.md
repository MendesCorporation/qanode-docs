# Reports

QANode automatically generates **PDF reports** after each execution and allows generating consolidated reports by period, with export and email delivery options.

---

## Execution Reports

After each scenario or suite execution, a **PDF report** is automatically generated with:

- **Summary**: Status, duration, execution date
- **Details per Node**: Individual status, logs, outputs, duration
- **Screenshots**: Screen captures (when configured in web nodes)
- **Errors**: Details of the failures encountered

The report is available in the execution list and can be downloaded at any time.

---

## Consolidated Reports

Generate reports covering multiple executions over a period:

1. In the side menu, click **Reports**
2. Select the **period** (start and end date)
3. (Optional) Filter by **project**
4. Click **Generate Report**

<!-- ![Generate report](../../assets/images/gerar-relatorio.png) -->
*Image: Reports screen with period selection, project filter, and generate button*

### Report Metrics

| Metric | Description |
|--------|-------------|
| **Success Rate** | Percentage of successful executions |
| **Total Executions** | Total number of executions in the period |
| **Successful Executions** | Number of passed executions |
| **Failed Executions** | Number of failed executions |
| **Average Duration** | Average execution time |
| **Top Failures by Scenario** | Scenarios that failed most often |
| **Top Failures by Node Type** | Node types that failed most often |

### Executions per Day Chart

The report includes a bar chart showing the distribution of successful and failed executions across the days of the selected period.

### Metrics per Project

When a specific project is selected, additional metrics are displayed:

| Metric | Description |
|--------|-------------|
| **Total Scenarios/Suites** | Count in the project |
| **Completed Scenarios** | Scenarios that passed |
| **Pending Scenarios** | Scenarios not yet executed |
| **Failed Scenarios** | Scenarios that failed |
| **Progress** | % of time elapsed vs % scenarios executed |

---

## Export

### PDF Download

1. Generate the report
2. Click **Export PDF**
3. The file will be downloaded with the name: `report_{project}_{start}_{end}.pdf`

### Email Delivery

1. Generate the report
2. Click **Send by Email**
3. Select recipients:
   - **Registered users** — select from the list
   - **Additional emails** — add custom addresses
4. Click **Send**

The PDF report will be sent as an **email attachment**.

> **Requirement:** SMTP must be configured in Settings for email delivery. See [Administration](../administration/enterprise-administration.md).

---

## Report Template

The PDF report layout is configurable. Go to **Settings** → **Report Template** to customize:

| Setting | Description |
|---------|-------------|
| **Background Image** | Background image for report pages |
| **Logo** | Logo displayed in the header |
| **Show Logo** | Enable/disable logo |
| **Include Screenshots** | Include screen captures |
| **Screenshot Size** | Image dimensions in the report |
| **Include Logs** | Include logs from each node |
| **Maximum Logs** | Log line limit per node |
| **Include Outputs** | Include node output data |
| **Include Duration** | Show duration of each node |
| **Include Node ID** | Show technical identifier |
| **Step Numbering** | Number steps sequentially |
| **Summary Section** | Include summary at the beginning |
| **Error Details** | Include complete error details |
| **Metadata** | Date, execution ID, project/flow name |

<!-- ![Report template](../../assets/images/template-relatorio.png) -->
*Image: Template configuration screen with options preview*

---

## Tips

- **Configure the template** before generating reports — especially logo and background image
- **Include screenshots** for visual evidence — essential for audits
- **Use project filter** for focused reports
- **Schedule deliveries** — use scheduled suites + alarms for automatic reports
- **Limit logs** — reports with too many logs become lengthy; use 10-20 lines per node
