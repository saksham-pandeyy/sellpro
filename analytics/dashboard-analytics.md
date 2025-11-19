# Dashboard Analytics

The analytics dashboard is the first thing users see when they log in. It provides a real-time snapshot of business performance through summary cards, interactive charts, and detailed statistics tables.

---

## What You Will Learn

- How the dashboard is structured with summary cards and charts
- How sales statistics are calculated and compared year-over-year
- How the graph section visualizes different metrics with charts
- How the dashboard prefetches data in the background for instant navigation
- How date ranges are persisted across sessions per view
- How edge cases are handled (zero values, missing data, partial periods)

---

## Dashboard Layout

The dashboard is organized into three main sections that load progressively:

1. **Summary cards** — Four color-coded cards showing key metrics (load first)
2. **Statistics widgets** — Smaller cards showing breakdowns (load second)
3. **Graph section** — Four interactive charts (load last, most data-heavy)

---

## Summary Cards

Four colored cards at the top show the most important metrics for the selected period:

| Card | Metric | Data Type | Comparison |
| ----------- | ---------------- | ----------- | --------------------------- |
| Sales | Total revenue | Currency | Same period last year |
| Units | Total units sold | Number | Same period last year |
| Net Profit | Revenue - costs | Currency | Same period last year |
| Net Margin | Profit / Revenue | Percentage | Same period last year |

### Percentage Change Calculation

The percentage change calculation handles several edge cases:
- Normal case: `((current - previous) / current) × 100`
- New metric with no history: shows 100% increase
- Dropped from positive to zero: shows -100%
- Both zero: shows 0% (no change)

These edge cases prevent division by zero and misleading indicators.

---

## Statistics Widgets

Below the summary cards, smaller cards break down specific areas:
- **Refunds** — Total refund count and dollar amount
- **Sales Breakdown** — Organic sales vs. PPC-attributed sales
- **Tips and Alerts** — Count of unread tips and alerts
- **Review Summary** — Total orders, review requests, reviews received

---

## Graph Section

Four interactive charts provide visual analysis:

### 1. Sales and Unit Graph

A combined chart with dual Y-axes showing revenue (line) and units sold (bar) on the same timeline.

### 2. Cost Breakdown Graph

Shows cost categories: Amazon fees, advertising costs, COGS, shipping costs, other costs.

### 3. Profit Over Time Graph

A line chart showing net profit trend — the most important chart answering "am I actually making money?"

### 4. PPC Summary Graph

Shows advertising performance: ad spend, attributed sales, ACOS trend.

### Chart Features

Each chart supports:
- Hover tooltips showing exact values
- Legend toggle to show/hide series
- Correct interaction at all zoom levels
- Export as image

---

## Data Prefetching Architecture

A background component runs when the dashboard loads and prefetches data for other modules:
- Checks which data has not been loaded into the state store yet
- Prefetches common datasets (inventory, products, shipments, sales)
- By the time the user navigates to other pages, data is often already available

**Prefetch logic:**
- Data not in store → Fetch from API + store
- Data in store but stale (> 5 min) → Fetch + update
- Data in store and fresh → Skip fetch, use cached

When a user clicks "Products" in the sidebar, the product list data is often already in the store. The page renders instantly with cached data and refreshes in the background.

---

## Year-Over-Year Comparison

The dashboard compares the current period with the same period last year:
- Current period: e.g., Mar 1 - Mar 31, 2026
- Previous period: Mar 1 - Mar 31, 2025 (same dates, previous year)
- If current period is partial, previous period uses same relative dates

---

## Date Range Persistence

The dashboard saves its selected date range to localStorage so it persists between sessions. The date range is applied to all dashboard components. Changing it triggers a full data refresh across all cards and charts.

---

## Performance Optimization

| Optimization | Implementation | Impact |
| ----------------------------- | ------------------------------------------------------------- | --------------------------- |
| Progressive loading | Cards load first, charts load last | User sees data 2x faster |
| Chart data caching | Chart data cached, refreshed on date range change | Reduces API calls |
| Conditional chart rendering | Charts only render when container is visible | Saves memory on hidden tabs |

---

## Interview Talking Points

**On the progressive loading strategy:** "The summary cards come from a lightweight endpoint that returns aggregated data. The charts come from a heavier endpoint with time-series data. Loading cards first means users see critical information (sales, units, profit, margin) immediately."

**On the Chart.js zoom handling:** "At non-default zoom levels, chart tooltips break because the browser reports mouse coordinates differently. A custom plugin divides coordinates by the zoom factor before Chart.js processes them. Three lines of code that fix tooltip positioning at every zoom level."

**On edge case handling:** "The percentage change calculation handles several edge cases. If last year there were no sales (new product), the change shows 100%. If both are zero, the change shows 0%. These prevent NaN values from appearing in the UI."

---

## Related Documents

- [Profit and Loss](profit-and-loss.md) - Detailed financial reporting
- [Custom Date Ranges](custom-date-ranges.md) - Date range persistence architecture
- [PPC Performance Analytics](../ppc/performance-analytics.md) - PPC-specific charts
- [Data Fetching Strategies](../frontend/data-fetching-strategies.md) - Prefetching patterns
- [UI Design System](../frontend/ui-design-system.md) - Chart theming and zoom handling
- [Report Polling](../notifications/report-polling.md) - Async data loading
