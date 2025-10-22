# Campaign Management

This document covers how the platform manages advertising campaigns across all three Amazon ad formats: Sponsored Products, Sponsored Brands, and Sponsored Display.

---

## What You Will Learn

- How campaigns are structured across the three ad formats
- How the frontend manages campaign data with domain-separated state
- How the table system handles campaign lists with different schemas
- How budget management works
- How campaign status transitions are handled
- How bulk editing works for multiple campaigns

---

## Campaign Data Model

Each ad format has its own campaign structure, but they share common fields:

| Field | SP | SB | SD | Description |
| ---------------- | ----- | ----- | ----- | ------------------------------------ |
| Campaign ID | Yes | Yes | Yes | Unique identifier from external API |
| Name | Yes | Yes | Yes | User-defined campaign name |
| Budget | Yes | Yes | Yes | Daily or lifetime budget |
| Status | Yes | Yes | Yes | Enabled, Paused, Archived |
| Targeting Type | Yes | No | No | Automatic or Manual |
| Bidding Strategy | Yes | Yes | Yes | Dynamic, Down Only, Fixed |
| Start Date | Yes | Yes | Yes | Campaign start |

The state store has separate slices for each format. Loading campaigns for Sponsored Products goes into one slice, Sponsored Brands campaigns into another. This separation prevents data collisions.

---

## Campaign List View

Each ad type has its own campaign list page. The pages look similar but the data comes from different API endpoints and goes into different state slices.

Each campaign list component handles:
- Fetching campaigns with date range filters
- Displaying campaigns in a sortable, filterable table
- Providing bulk selection via checkboxes
- Showing campaign performance metrics (impressions, clicks, spend, sales, ACOS)
- Linking to detail views for each campaign

### Column Customization

The campaign table has configurable columns. Users can show or hide columns, resize them, and reorder them. These preferences are saved per table key so the layout persists between visits.

Typical columns for campaign lists:
- Campaign Name (with link to detail view)
- Status (with status badge)
- Budget (with spend percentage)
- Impressions, Clicks, Spend, Sales
- ACOS, ROAS
- Impressions Share
- Start Date

---

## Campaign Detail View

Clicking a campaign name opens a detail view with multiple sub-sections:

### For Sponsored Products

The detail view has tabs for each management level:
1. **Campaign Settings** — Budget, targeting type, bidding strategy, status
2. **Ad Groups** — List of ad groups within the campaign
3. **Targeting** — Keywords and product targets
4. **Search Terms** — Actual customer search queries
5. **Placements** — Top of Search, Product Pages, Rest of Search
6. **Negative Keywords** — Keywords to exclude

### For Sponsored Brands

1. **Campaign Settings** — Budget, bidding strategy, status
2. **Ad Groups** — Keywords and products
3. **Targeting** — Keywords
4. **Search Terms** — Customer search queries
5. **Negatives** — Negative keywords

### For Sponsored Display

1. **Campaign Settings** — Budget, bidding strategy, status
2. **Ad Groups** — Targeting groups
3. **Targeting** — Audience and contextual targets

---

## Campaign Creation

Creating a new campaign is a multi-step wizard:

1. Choose campaign type (SP/SB/SD)
2. Set campaign settings (name, budget, dates)
3. Choose targeting type (auto/manual)

**If manual targeting:**
4. Add keywords or product targets
5. Set bids per keyword

**If automatic targeting:**
4. Set default bid
5. Choose targeting categories

6. Review and confirm

The wizard preserves state across steps. If the user goes back, their previous selections are restored.

---

## Budget Management

Campaign budgets can be updated individually or in bulk. The budget display shows:
- **Budget amount** — The daily or lifetime budget
- **Spend to date** — How much has been spent in the current period
- **Budget utilization** — Progress bar showing spend vs. budget
- **Estimated overspend** — Projection of whether the campaign will exceed budget

### Budget Edit Flow

1. User clicks the budget field or selects campaigns for bulk update
2. A modal opens with the current budget value
3. User enters the new budget
4. Frontend validates the value (positive number, within limits)
5. API call updates the budget on the external service
6. Table refreshes with the new value

---

## Campaign Status Management

Campaigns can have these statuses:
- **Enabled** — Campaign is active and spending
- **Paused** — Campaign is not spending but retains settings
- **Archived** — Campaign is permanently stopped (cannot be reactivated)

Status changes go through a confirmation flow to prevent accidental changes.

---

## Engineering Decisions Worth Talking About in Interviews

### Why separate state per ad format

The three ad formats look similar but have different data shapes, different API endpoints, and different sub-resources. A single state slice would become a mess of conditional logic. Separate slices keep each format's code clean. The cost is some duplication, but the benefit is that changing Sponsored Products logic never breaks Sponsored Brands.

### Why the table is a shared component

Every campaign list needs sorting, filtering, pagination, column customization, and checkbox selection. Building these into a shared table component means every list gets them for free. Adding a new feature benefits all 30+ table instances.

### Why campaign creation is a wizard

Campaign creation has multiple steps with dependencies between them (you cannot add keywords before choosing targeting type). A wizard enforces the correct order and validates each step before proceeding.

---

## Related Documents

- [PPC Module Overview](overview.md) - Module-level architecture
- [Strategy Automation](strategy-automation.md) - Automated campaign optimization
- [Keywords and Targeting](keyword-and-targeting.md) - Keyword and targeting management
- [Bulk Operations](bulk-operations.md) - Bulk campaign editing
- [Performance Analytics](performance-analytics.md) - Campaign performance metrics
