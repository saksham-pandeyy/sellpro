# Inventory Health

The inventory health module gives sellers a real-time view of their stock levels across all marketplaces. It answers the most important question: "Am I going to run out of stock?"

---

## What You Will Learn

- How inventory data is pulled and displayed
- How days of supply is calculated and what it means
- How stock alerts work (low stock, out of stock, excess inventory)
- How the inventory table is organized with sorting and filtering
- How the sync indicator shows data freshness

---

## Inventory List View

The main inventory page shows a table of all products with their current stock status.

**Key columns in the table:**

| Column | Description |
| --------------- | ------------------------------------------------------------ |
| ASIN | Amazon product identifier |
| SKU | Seller's internal stock keeping unit |
| Product Name | Product title |
| Available Stock | Units available in FBA |
| Inbound Stock | Units in transit to FBA |
| Total Stock | Available + inbound |
| Days of Supply | Estimated days until stockout at current sales rate |
| Status | Health indicator (healthy, low, critical, out of stock) |
| Last Updated | When the external service last reported this data |

---

## Days of Supply Calculation

Days of Supply estimates how many days the current stock will last based on recent sales velocity.

```
Days of Supply = Available Stock / Average Daily Sales
```

Average daily sales is calculated over a configurable lookback period (default 30 days). This smooths out daily fluctuations and gives a more stable estimate.

**Thresholds for stock status:**

| Status | Days of Supply | Color |
| -------------- | -------------------- | ------ |
| Healthy | More than 60 days | Green |
| Adequate | 30-60 days | Blue |
| Low | 15-30 days | Yellow |
| Critical | Less than 15 days | Orange |
| Out of Stock | 0 days | Red |

---

## Stock Alerts

The system monitors stock levels and generates alerts when certain conditions are met:

| Alert Type | Trigger | Action |
| ---------------- | ----------------------------------------------- | ------------------------------------- |
| Low Stock | Stock drops below 28 days of supply | Alert created, notification sent |
| Out of Stock | Stock reaches 0 | Alert created, notification sent |
| Excess Inventory | Stock exceeds 90 days of supply | Alert created, suggests promotion |
| High Return Rate | Return rate exceeds threshold | Alert created |
| Price Change | Product price changes significantly | Alert created |

Alerts include the product name, ASIN, current stock level, and a link to take action.

---

## Data Sync Integration

The inventory health data comes from an external API. A sync process runs periodically to keep the data fresh. When syncing is in progress, the sidebar shows an animated sync icon next to the Inventory menu item. The icon disappears when the sync completes.

Each route only shows sync status for its own data. Only the inventory_list sync step affects the Inventory page indicator.

---

## Table Features

The inventory table supports:
- **Sorting** by any column (stock level, days of supply, product name)
- **Searching** by ASIN, SKU, or product name
- **Column customization** — show/hide columns, resize widths
- **Export** for offline analysis
- **Row highlighting** based on stock status

---

## Interview Talking Points

**On the days of supply calculation:** "Days of Supply sounds simple — stock divided by daily sales. But the daily sales rate needs to be calculated carefully. A 7-day window is too noisy. A 90-day window is too slow. We settled on a 30-day rolling average, which balances responsiveness with stability."

**On sync architecture:** "Inventory data is never real-time. The external service updates it on a schedule. We run a background sync and show users when their data was last updated. The sidebar icons give immediate visual feedback about data freshness. This manages expectations — users know the data might be a few hours old."

---

## Related Documents

- [Inventory Module Overview](overview.md) - Module architecture
- [Product Management](product-management.md) - Product catalog
- [3PL and AWD](3pl-and-awd.md) - Multi-location stock
- [Demand Forecasting](futuristic-analytics.md) - Demand forecasting
- [Alert Categories](../alerts/alert-categories.md) - Inventory alert types
