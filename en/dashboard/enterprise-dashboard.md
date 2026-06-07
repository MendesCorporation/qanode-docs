# Dashboard

The **Dashboard** provides a consolidated view of your test status through configurable **widgets** — metric cards, charts, and tables.

---

## Overview

[Main dashboard](../../assets/images/dashboard-visao-geral.mp4)

*Image: Dashboard with multiple widgets: metric cards, bar chart, and table*

---

## Multiple Dashboards

QANode supports multiple dashboards with different visibility settings:

| Type | Visibility | Description |
|------|------------|-------------|
| **Private** | Creator only | Personal dashboard |
| **Public** | All users | Shared dashboard |
| **By Role** | Users with specific roles | Team/role dashboard |
| **System** | All users (read-only) | QANode default dashboards |

[Dashboards with different visibility settings](../../assets/images/dashboard-visibilidade-painel.mp4)

---

## Widgets

### Widget Types

| Type | Description | Typical Use |
|------|-------------|-------------|
| **Metric Card** | Single numeric value with conditional formatting | Total executions, success rate |
| **Gauge** | Semicircular indicator with minimum, maximum, and current value | Success rate, SLA, progress |
| **Bar Chart** | Vertical bars with one or more series | Executions per day, failures per project |
| **Stacked Bars** | Vertical bars with multiple stacked series | Cumulative comparison between categories |
| **Horizontal Bars** | Horizontal bars with one or more series | Rankings, categorical comparisons |
| **Line Chart** | Trend lines | Success rate evolution |
| **Stacked Line** | Multiple lines with accumulated series | Evolution by status or team |
| **Area Chart** | Filled area | Cumulative executions |
| **Stacked Area** | Areas accumulated by series | Volume by status over time |
| **Pie Chart** | Proportional distribution | Success/failure ratio |
| **Donut** | Proportional distribution with a free center for totals | Status distribution with central value |
| **Funnel** | Funnel-shaped process stages | Conversion, quality, or process stages |
| **Calendar Heatmap** | Daily intensity in a calendar view | Failures per day, runs per day |
| **Scatter** | Points mapped by X/Y axes | Duration by number of steps |
| **Bubble** | Scatter chart with variable point size | Duration, steps, and severity |
| **Table** | Tabular data | List of recent executions |

### Creating a Widget

The widget creation wizard has 3 steps:

[Creating a Widget](../../assets/images/dashboard-criando-widget.mp4)

#### Step 1: Data Source

Define where the data will come from:

**Query Builder (Recommended):**

| Field | Description |
|-------|-------------|
| **Table** | Data table (runs, projects, flows, etc.) |
| **Columns** | Fields to select |
| **Filters** | Conditions (equal to, contains, greater than, etc.) |
| **Grouping** | Group by field (with date formatting) |
| **Aggregation** | COUNT, SUM, AVG, MIN, MAX |
| **Sorting** | Field and direction (ASC/DESC) |
| **Pivot** | Create multiple series from a field |
| **Limit** | Maximum number of records (up to 1000) |

**Filter Logic:** Combine multiple filters using **AND** or **OR** operators.

**Date Formatting in Grouping:**

| Format | Result |
|--------|--------|
| Date Only | `2024-01-15` |
| Time Only | `14:30` |
| Date and Time | `2024-01-15 14:30` |
| Month/Year | `2024-01` |
| Year | `2024` |

**Direct SQL:**

For more complex queries, use SQL mode with the Monaco editor. Prefer Query Builder for simple widgets and use SQL when you need CTEs, joins, conditional aggregations, or calculations that would be hard to express in the builder:

```sql
SELECT
  DATE("startedAt") AS day,
  status,
  COUNT(*)::int AS total
FROM "Run"
WHERE "isSandbox" = false
  AND "startedAt" >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY day, status
ORDER BY day;
```

> SQL mode requires the `dashboard.sql` permission.

> SQL also supports CTEs (Common Table Expressions) with the `WITH` clause. Queries that modify data (INSERT, UPDATE, DELETE, DROP, etc.) remain blocked.

#### Step 2: Visualization

Choose the chart type and map the fields:

| Setting | Description |
|---------|-------------|
| **Chart Type** | Metric, Gauge, Bar, Stacked Bar, Horizontal Bar, Line, Area, Pie, Donut, Funnel, Heatmap, Scatter, Bubble, or Table |
| **X Axis** | Field for the horizontal axis |
| **Y Axis** | Field for the vertical axis (numeric value) |
| **Series** | Field for multiple series (when using pivot) |
| **Label** | Label field, used by pie, donut, funnel, scatter, bubble, and tables |
| **Size** | Numeric field used by Bubble |
| **Legend** | Show/hide legend |
| **Tooltip** | Show values on mouse hover |

#### Step 3: Appearance

| Setting | Description |
|---------|-------------|
| **Title** | Name displayed on the widget |
| **Colors** | Custom colors per series/category |
| **Conditional Formatting** | Color rules based on values |
| **Progress Columns** | Displays numeric table columns as progress bars |
| **Threshold / Target** | Visual references for gauges and indicators |

**Conditional Formatting** (for cards and tables):

| Operator | Example |
|----------|---------|
| `>` | If value > 90 → green |
| `<` | If value < 50 → red |
| `=` | If value = "failed" → red |
| `contains` | If contains "error" → yellow |

### Tables With Progress

In table widgets, numeric columns can be shown as progress bars. This is useful for tracking:

- project progress;
- success percentage;
- scenario coverage;
- SLA consumption;
- pipeline progress.

Each progress column can have its own color and minimum/maximum values. Text fields, IDs, and dates are not recommended for progress.

### Conditional Formatting In Tables

Conditional formatting rules let you highlight rows or cells based on values returned by the query.

Examples:

| Rule | Result |
|------|--------|
| `status = failed` | Red row |
| `success_rate < 80` | Attention indicator |
| `failures > 0` | Highlight scenarios with failures |

Use conditional formatting to draw attention to issues, but avoid applying too many colors at the same time.

---

## Dashboard Layout

The dashboard uses a responsive **12-column grid**:

| Property | Description |
|----------|-------------|
| **Width** | 1 to 12 columns |
| **Height** | 1 to 10 rows |
| **X Position** | Starting column (0-11) |
| **Y Position** | Starting row |

### Rearranging Widgets

[Rearranging Widgets](../../assets/images/dashboard-reorganizar-widgets.mp4)

- **Drag** a widget to reposition it
- **Resize** by dragging the bottom-right corner
- The grid adjusts automatically to prevent overlaps
- Default widget sizes are optimized by type: metrics are smaller, while charts and tables start larger.

---

## Widget Examples

[Widget Examples](../../assets/images/dashboard-tipos-de-widgets.mp4)

### Card: Success Rate

```
Table: runs
Aggregation: COUNT(*)
Filter: status = "success" AND started_at >= today - 30 days
```

### Bar Chart: Executions per Day

```
Table: runs
Grouping: started_at (date only)
Aggregation: COUNT(*)
Pivot: status
Type: Bar
```

Result: Stacked bars with different colors for success/failure per day.

### Table: Latest Failures

```
Table: runs
Filter: status = "failed"
Columns: flow name, date, duration, error
Sorting: started_at DESC
Limit: 10
```

---

## Auto-Refresh

Widgets can be configured to refresh automatically at defined intervals, keeping data always up to date.

[Auto-Refresh](../../assets/images/dashboard-auto-refresh.mp4)

---

## Tips

- **Start simple** — a metric card with total executions + a daily bar chart
- Use **conditional formatting** to highlight issues (rate below 80% = red)
- **Team dashboards** — create dashboards with role-based visibility
- Use **pivot mode** to create charts with multiple series without SQL
- **Limit your data** — large queries can impact performance
