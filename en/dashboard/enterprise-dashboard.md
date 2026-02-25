# Dashboard

The **Dashboard** provides a consolidated view of your test status through configurable **widgets** — metric cards, charts, and tables.

---

## Overview

[Main dashboard](../../assets/images/dashboard.mp4)
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

---

## Widgets

### Widget Types

| Type | Description | Typical Use |
|------|-------------|-------------|
| **Metric Card** | Single numeric value with conditional formatting | Total executions, success rate |
| **Bar Chart** | Vertical bars with one or more series | Executions per day, failures per project |
| **Line Chart** | Trend lines | Success rate evolution |
| **Area Chart** | Filled area | Cumulative executions |
| **Pie Chart** | Proportional distribution | Success/failure ratio |
| **Table** | Tabular data | List of recent executions |

### Creating a Widget

The widget creation wizard has 3 steps:

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

For more complex queries, use SQL mode with the Monaco editor:

```sql
SELECT
  DATE(started_at) as dia,
  status,
  COUNT(*) as total
FROM runs
WHERE started_at >= NOW() - INTERVAL '30 days'
GROUP BY dia, status
ORDER BY dia
```

> SQL mode requires the `dashboard.sql` permission.

#### Step 2: Visualization

Choose the chart type and map the fields:

| Setting | Description |
|---------|-------------|
| **Chart Type** | Card, Bar, Line, Area, Pie, Table |
| **X Axis** | Field for the horizontal axis |
| **Y Axis** | Field for the vertical axis (numeric value) |
| **Series** | Field for multiple series (when using pivot) |
| **Legend** | Show/hide legend |
| **Tooltip** | Show values on mouse hover |

#### Step 3: Appearance

| Setting | Description |
|---------|-------------|
| **Title** | Name displayed on the widget |
| **Colors** | Custom colors per series/category |
| **Conditional Formatting** | Color rules based on values |

**Conditional Formatting** (for cards and tables):

| Operator | Example |
|----------|---------|
| `>` | If value > 90 → green |
| `<` | If value < 50 → red |
| `=` | If value = "failed" → red |
| `contains` | If contains "error" → yellow |

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

- **Drag** a widget to reposition it
- **Resize** by dragging the bottom-right corner
- The grid adjusts automatically to prevent overlaps

---

## Widget Examples

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

---

## Tips

- **Start simple** — a metric card with total executions + a daily bar chart
- Use **conditional formatting** to highlight issues (rate below 80% = red)
- **Team dashboards** — create dashboards with role-based visibility
- Use **pivot mode** to create charts with multiple series without SQL
- **Limit your data** — large queries can impact performance
