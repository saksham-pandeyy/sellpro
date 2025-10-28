# Performance Analytics

This document covers how the PPC module displays performance data through dashboards, charts, and statistics tables. The analytics layer helps users understand campaign performance and make data-driven decisions.

---

## What You Will Learn

- How the PPC dashboard aggregates performance data across all ad types
- How the statistics table works with sortable columns
- How charts visualize trends over time
- How date range management works in the PPC context
- How product-level analytics break down performance by ASIN

---

## PPC Dashboard

The PPC dashboard is the first thing users see when they enter the PPC module. It shows aggregate performance across all campaigns.

### Dashboard Sections

1. **Summary cards** — Total spend, total sales, total orders, ACOS, ROAS for the selected period
2. **Performance charts** — Sales, spend, impressions, and clicks over time
3. **Statistics table** — Breakdown by campaign or product
4. **Product modal** — Deep dive into individual ASIN performance

---

## Period Selector

The period selector lets users choose the time range for all dashboard data:
- **Preset ranges** — Last 7 days, Last 14 days, Last 30 days, Last 90 days, Last year
- **Custom date range** — User picks start and end dates from a date picker
- **Comparison period** — Some views support comparing two periods side by side

The selected date range is persisted to localStorage so it is restored when the user comes back.

---

## Summary Cards

Key metrics displayed at the top of the dashboard:

| Metric | Description | Format |
| ------------- | ------------------------------------------------ | ---------- |
| Total Spend | Total ad spend for the period | Currency |
| Total Sales | Total attributed sales | Currency |
| Total Orders | Number of orders attributed to ads | Number |
| ACOS | Advertising Cost of Sale (spend/sales) | Percentage |
| ROAS | Return on Ad Spend (sales/spend) | Decimal |

---

## Charts

The dashboard has multiple chart visualizations:

**Sales Chart** — Line chart showing attributed sales over time. Users can see trends, spikes, and patterns in ad-attributed revenue.

**Spend Chart** — Line or bar chart showing ad spend over the same period. Often shown alongside the sales chart so users can see the relationship between spend and revenue.

**Impressions Chart** — Line chart showing how often ads were shown.

**Clicks Chart** — Line chart showing click volume. Comparing clicks to impressions gives the click-through rate (CTR).

### Chart Controls

Each chart supports:
- Hover tooltips showing exact values
- Legend toggling for multi-series charts
- Export as image

---

## Statistics Table

Below the charts, a detailed table breaks down performance by campaign or product.

### Campaign Statistics

Columns include campaign name, impressions, clicks, CTR, spend, sales, ACOS, ROAS, orders, CPC, and conversion rate. Each column is sortable. Users can customize which columns appear.

### Product Statistics

A separate table breaks down performance by ASIN, showing ad spend, attributed sales, organic sales, total sales, ACOS, orders, and revenue.

---

## Product-Level Analytics

Clicking a product from the statistics table opens a modal with detailed performance data:
- **Overview** — Total spend, sales, orders for this product
- **Trend chart** — Performance over time
- **Campaign breakdown** — Which campaigns drove sales for this product
- **Keyword performance** — Which keywords generated sales

---

## Date Range Persistence

The PPC module has its own date range keys that are persisted separately from other modules. When a user sets a date range on the PPC dashboard and navigates to Inventory, the PPC date range is saved. When they return to PPC, their previous date range is restored.

---

## Interview Talking Points

**On data aggregation:** "The dashboard aggregates data from multiple API endpoints. Summary cards come from one endpoint, chart data from another, and the statistics table from a product statistics endpoint. Each endpoint has different aggregation levels and time granularity."

**On performance with large datasets:** "An Amazon seller might have thousands of campaigns and millions of search terms. The statistics table is paginated, charts show aggregated data points, and product data fetches on demand. The date range filter limits the data scope."

**On chart customization:** "Charts need custom plugins for certain features — for example, correcting mouse coordinates at non-default zoom levels so tooltips work correctly."

---

## Related Documents

- [PPC Module Overview](overview.md) - Module-level architecture
- [Campaign Management](campaign-management.md) - Campaign management
- [Strategy Automation](strategy-automation.md) - Automated optimization
- [Dashboard Analytics](../analytics/dashboard-analytics.md) - General dashboard system
