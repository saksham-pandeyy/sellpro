# Profit and Loss Report

The Profit and Loss (P&L) report calculates true profitability by subtracting all fees, advertising costs, and COGS from revenue.

---

## What You Will Learn

- How the P&L report aggregates data from multiple sources
- How fees are categorized and reconciled
- How COGS is managed at the ASIN level
- How the async report generation pipeline works
- How the report handles edge cases (refunds, adjustments, missing data)

---

## Why P&L Is Challenging

Transaction data is scattered across multiple reports with different formats, timestamps, and identifiers:

| Data Source | What It Contains | Update Frequency |
| -------------------- | --------------------------------------- | ---------------- |
| Sales data | Revenue, units sold, returns | Daily |
| Fee data | Referral fees, FBA fees, storage fees | Daily |
| Advertising data | Ad spend by campaign | Every 6 hours |
| COGS (manual input) | Cost of goods sold per ASIN | On edit |

Aggregating these into a single profit number requires:
1. **Normalizing timestamps** across sources using different time zones
2. **Matching transactions** across different identifier systems
3. **Handling partial data** when one source is available but another is not
4. **Currency conversion** for multi-marketplace sellers

---

## Data Aggregation Pipeline

```
External APIs → Sync Workers → Database (transactions table) → P&L Report Engine → Materialized report → Frontend
```

### Step 1: Data Collection

Sync workers pull data from external APIs and normalize it into a unified transactions table. Each transaction gets a type and category: order (revenue), fee, adjustment, etc.

### Step 2: Fee Categorization

Fees are categorized into meaningful groups:

| Fee Category | Amazon Fee Types Included | Typical % of Revenue |
| ------------------- | ------------------------------------------------------- | -------------------- |
| Referral Fees | Referral fee, variable closing fee | 15% |
| FBA Fees | Fulfillment fee, pick & pack, weight handling | 20-30% |
| Storage Fees | Monthly storage, long-term storage | 2-5% |
| Advertising Costs | Sponsored Products, Brands, Display ad spend | 10-30% |
| COGS | Unit cost, shipping cost, prep cost | 20-40% |
| Other | Removal fees, return processing fees, adjustment fees | 1-3% |

### Step 3: COGS Integration

COGS comes from two sources:
1. **Manual entry** — Sellers enter costs per product
2. **Production orders** — When inventory arrives from a supplier, the order records the unit cost

The report uses this hierarchy:
1. Production order cost from most recent receipt (most accurate)
2. Fall back to manually entered unit cost
3. If neither is available, mark product as having missing COGS

### Step 4: Profit Calculation

Profit is calculated at multiple levels:

```
Per ASIN: Net Profit = Revenue - Fees - Ad Spend - COGS
Per Period: Total Revenue - Total Fees - Total Ad Spend - Total COGS = Net Profit
```

---

## Async Report Generation

P&L reports can take significant time to generate for sellers with thousands of products. Rather than blocking the user, the report is generated asynchronously.

### Generation Flow

1. User requests a P&L report
2. API enqueues a report generation job, returns job ID
3. Frontend starts polling for completion (every 30 seconds, up to 5 attempts)
4. Report engine processes data and stores the completed report
5. When complete, a notification appears with a "View" button

### Report Caching

Completed reports are stored as materialized snapshots. The snapshot includes the full per-ASIN breakdown as structured data. This allows the report to be displayed without additional queries. When a user requests the same date range again, the cached report is returned if it was generated after the last data sync.

---

## Report Structure

### 1. Report Statistics (Summary Cards)

Six summary cards show key financial metrics:
- Total Revenue, Amazon Fees, Advertising Cost, COGS, Net Profit, Profit Margin

### 2. Cost Breakdown

A detailed view of all costs categorized by type. Each category can be expanded:
- **Amazon Fees** — Referral fees, FBA fees, storage fees, removal fees
- **Advertising** — Sponsored Products, Sponsored Brands, Sponsored Display
- **COGS** — Per-product cost of goods
- **Other** — Shipping, prep, returns, adjustments

### 3. Product Breakdown Table

A sortable, customizable table showing profitability per product: ASIN, product name, units sold, revenue, COGS, fees, ad spend, net profit, margin.

### 4. Cost Breakdown Modal

Clicking on a cost category opens a modal with detailed line items: fee type, transaction date, amount, related ASIN, description.

### 5. Profit and Sales Chart

A dual-series line chart showing revenue and profit trends. The profit line color changes — green if all periods are profitable, red if any period shows a loss.

---

## Edge Cases and Error Handling

### Missing COGS Data

Products with no COGS entered appear with a warning. The summary card shows a warning banner: "X products have missing COGS data. Net profit may be inaccurate."

### Partial Period Data

When data has not finished syncing for the requested period, the report includes all available data with a notice showing the last sync timestamp.

### Refund Adjustments

Refunds and returns are included as negative transactions. A refund reduces revenue for the period in which the refund occurred (not the original sale period).

---

## Interview Talking Points

**On the data aggregation challenge:** "The hardest part is aggregating data from different sources. Fees come from one API, ad costs from another, and COGS is entered manually. Each source has different timestamps, identifiers, and formats. The report engine normalizes everything into a unified transactions table first, then aggregates from there."

**On the async generation pattern:** "Reports for sellers with thousands of products can take 30 seconds or more. Process asynchronously with polling so the user is not blocked. Completed reports are cached so subsequent requests for the same date range return instantly."

**On COGS accuracy:** "The P&L is only as accurate as the COGS data. We make COGS editable at the product level, pull unit costs from production orders when available, and show warnings for products with missing data."

---

## Related Documents

- [Dashboard Analytics](dashboard-analytics.md) - Dashboard overview
- [Reimbursement Audit](reimbursement-audit.md) - Lost inventory recovery
- [Custom Date Ranges](custom-date-ranges.md) - Date range selection
- [Data Synchronization](../onboarding/data-synchronization.md) - Amazon data sync
- [Product Management](../inventory/product-management.md) - COGS management
- [Report Polling](../notifications/report-polling.md) - Async report generation
