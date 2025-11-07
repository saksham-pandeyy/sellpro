# Product Management

The product management module provides a searchable catalog of all ASINs a seller manages. It includes product details, images, sales history, cost configuration, and competitor pricing.

---

## What You Will Learn

- How the product list page is organized with search, filtering, and column customization
- How the single product page shows detailed performance data
- How product costs (COGS) are managed at the ASIN level
- How product images are uploaded and managed
- How the sales chart handles products with different sales velocities
- How product settings feed into demand forecasting and P&L calculations

---

## Product Data Model

Each product record contains:
- Identifiers (ASIN, SKU)
- Product name and category
- Current price
- Cost components (unit cost, shipping cost, prep cost)
- Lead time (days from order to delivery from supplier)
- Safety stock (minimum stock to maintain)
- Image URL
- Active status
- Marketplace

---

## Product List Page

The product list page shows all ASINs in a sortable, searchable, customizable table with 30+ possible columns.

### Table Columns

| Column | Data Type | Description |
| ------------------ | -------------- | ---------------------------------- |
| ASIN | string | Amazon product identifier (linked) |
| Product Name | string | Product title |
| SKU | string | Seller's stock keeping unit |
| Current Price | currency | Current selling price |
| Stock Level | number | Available FBA units |
| Days of Supply | number | Stock / daily sales rate |
| Total Sales | currency | Sales in current period |
| Profit Margin | percentage | (Price - Cost) / Price |
| Status | string | Active, inactive, discontinued |
| Category | string | Amazon product category |
| Last Updated | date | Last data sync timestamp |

### Search and Filter

The search bar supports multiple query modes with auto-detection:
- If the query starts with a pattern matching ASIN format → search by ASIN
- If the query contains dashes → search by SKU
- Otherwise → broad search across name, ASIN, and SKU

### Export

Users can export the visible product list to spreadsheet format. Only visible columns are included, and values are formatted according to their data type before export.

---

## Single Product Page

Clicking a product ASIN opens a detail page with multiple sections:

### 1. Product Overview

The header shows product name, ASIN, SKU (with copy-to-clipboard), main product image, current price, competitor price, stock level with color-coded status, category, and marketplace flags.

### 2. Sales History Chart

An interactive chart showing sales trends with:
- **Date range selection** — 7, 14, 30, 90 day presets + custom range
- **Period comparison** — Compares current period with same-length previous period
- **Dual Y-axes** — Revenue on one axis, units on another
- **Hover tooltips** — Show exact values
- **Skeleton loading** — Chart-shaped placeholder while data loads

### 3. Cost Configuration

A settings modal allows updating product-level financial data:
- Unit cost (COGS)
- Shipping cost per unit
- Prep cost (labeling, preparation)
- Other costs
- Lead time (days)
- Safety stock

Total cost is auto-calculated from all components. Validation ensures positive numbers. Changes feed into profit margin, P&L report, and demand forecasting.

### 4. Product Images

- **Main Image** from Amazon — Displayed at the top, click to enlarge
- **Additional Images** — Loaded from product catalog, displayed as horizontal gallery
- **Custom Upload** — Users can upload replacement images with cropping and resizing

---

## Product Sales and Profit Calculations

### Profit Margin

Profit margin is calculated as (price - total cost) / price. Total cost includes unit cost, shipping cost, prep cost, and other costs. If price or cost data is missing, the margin is not shown.

### Days of Supply

Days of supply is calculated as available stock divided by average daily sales. Average daily sales uses a configurable lookback period (default 30 days) to smooth out daily fluctuations.

---

## Interview Talking Points

**On COGS management:** "Accurate profit calculation depends on accurate COGS. We made COGS editable at the product level with multiple cost components. The total is auto-calculated and feeds into profit margin, P&L report, and demand forecasting modules."

**On search optimization:** "The search bar auto-detects what the user is searching for. If the query looks like an ASIN or SKU, it searches by that field specifically. Otherwise it does a broad search. This gives fast, relevant results without requiring the user to select a search mode."

---

## Related Documents

- [Inventory Module Overview](overview.md) - Module architecture
- [Inventory Health](inventory-health.md) - Stock monitoring and days of supply
- [Profit and Loss](../analytics/profit-and-loss.md) - COGS impact on P&L
- [Demand Forecasting](futuristic-analytics.md) - Forecasting with lead times and safety stock
- [Production Orders](production-orders.md) - Manufacturing lifecycle
