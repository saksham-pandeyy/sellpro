# PPC Advertising Module

This is the most feature-rich part of the platform. It lets sellers manage all three Amazon ad types (Sponsored Products, Sponsored Brands, Sponsored Display) from one interface, automate bid optimization through strategies, and track every change with a full audit history.

---

## Why This Module Matters for Portfolio

The PPC module demonstrates several important engineering capabilities:
- Managing a complete lifecycle for three different ad formats, each with their own data models
- Building a rule-based strategy engine that evaluates campaigns and makes automated bid adjustments
- Implementing an audit system that tracks every automated change with before/after snapshots
- Handling bulk operations across hundreds of campaigns at once
- Integrating with external APIs that have strict rate limits and pagination

---

## Module Structure

The PPC module is organized by ad format, with shared components for cross-cutting features:

```
ppc/
  campaigns/                  # Cross-campaign management
  dashboard/                  # PPC analytics overview
  history-components/         # Audit history (shared across all ad types)
  sponsored-brand/            # Sponsored Brands management
  sponsored-display/          # Sponsored Display management
  sponsored-products/         # Sponsored Products management
  strategy/                   # Automation engine
```

---

## The Three Ad Formats

### Sponsored Products

These promote individual product listings in search results and product detail pages. Management includes:
- Campaign settings (budget, targeting type, bidding strategy)
- Ad groups within each campaign
- Keyword targeting (manual and automatic)
- Product targeting (competing ASINs and categories)
- Search terms (actual customer queries)
- Placements (Top of Search, Product Pages, Rest of Search)
- Negative keywords and negative product targets

**Engineering challenge:** A single campaign can have hundreds of keywords with individual bids, and thousands of search terms. The management interface handles this data efficiently with sorting, filtering, and bulk operations.

### Sponsored Brands

These are header ads showing a brand logo, headline, and multiple products. Management includes:
- Campaign settings
- Ad groups
- Keyword targeting
- Search terms
- Negative keywords

The interface is similar to Sponsored Products but the data structure differs enough that separate components are needed. Shared patterns (tables, history, bulk actions) are reused through common components.

### Sponsored Display

These are retargeting ads for customers who have viewed similar products. Management includes:
- Campaign settings
- Ad groups
- Targeting (audience and contextual)

Display has fewer management levels than Products or Brands, but the targeting logic is more complex (audience segments vs. keywords).

---

## Key Architecture Decisions

### Shared Components Across Ad Types

Each ad type has its own management interface, but they all use the same underlying table component for rendering data, and the same date range persistence for filtering. Adding a new feature to the shared table (like column resize) automatically benefits all three ad types.

### State Separation

Separate state slices for each ad type and for cross-cutting PPC concerns prevent data from one ad type overwriting another in the store. This means fetching Sponsored Products data never affects Sponsored Brands data.

### Table Customization

Every PPC table allows users to:
- Show/hide columns from a dropdown
- Resize column widths by dragging
- Sort by any column
- Save column preferences per table
- Select rows with checkboxes for bulk actions

Column preferences are unique per table key. A user's Sponsored Products campaign table layout is separate from their Sponsored Brands campaign table layout.

---

## What Each PPC Document Covers

| Document | What it covers | Best for portfolios |
|----------|---------------|-------------------|
| [Campaign Management](campaign-management.md) | CRUD for all three ad types, budget management, status tracking | Shows understanding of multi-format data modeling |
| [Strategy Automation](strategy-automation.md) | Rule engine, bid optimization, keyword harvesting | Shows ability to build rules engines and automated systems |
| [Keywords and Targeting](keyword-and-targeting.md) | Keyword management, negative targeting, placement optimization | Shows attention to detail in complex domain logic |
| [Performance Analytics](performance-analytics.md) | Dashboards, charts, statistics tables, date range comparison | Shows data aggregation and visualization skills |
| [Audit History](audit-history.md) | Change tracking, history sidebar, before/after snapshots | Shows understanding of auditing and compliance |
| [Bulk Operations](bulk-operations.md) | Bulk bid updates, campaign edits, targeting changes | Shows handling of large-scale operations |

---

## Related Documents

- [Campaign Management](campaign-management.md) - Deep dive into campaign CRUD
- [Strategy Automation](strategy-automation.md) - The automated bid optimization engine
- [Performance Analytics](performance-analytics.md) - Charts, dashboards, and data aggregation
- [Backend Service Architecture](../backend/service-architecture.md) - How the PPC service works
