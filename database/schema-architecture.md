# Database Design

The database uses a multi-tenant design where each marketplace represents a separate tenant. Data is isolated at the marketplace level to prevent cross-tenant data leakage. This document covers the schema architecture approach, migration patterns, query optimization considerations, caching strategy, and the tradeoffs behind key design decisions.

---

## What You Will Learn

- How multi-tenant data isolation works at the database level
- How the schema is organized by business domain
- How indexing strategies handle common query patterns
- How data retention and archival are managed
- How the caching layer integrates with the database
- How schema migrations are handled in production

---

## Multi-Tenant Approach

Each marketplace (US, UK, DE, FR, IT, ES, CA, MX, BR) is treated as a separate tenant. When a user switches marketplaces, the API scopes all queries to that marketplace.

### Why Marketplace-Level Tenancy?

Most multi-tenant SaaS platforms tenant at the account level. This approach tenants at the marketplace level because Amazon sellers operate differently in each marketplace. A seller might have the same ASIN listed on Amazon.com and Amazon.co.uk, but the inventory levels, pricing, PPC campaigns, and sales data are completely independent.

### How Isolation Is Enforced

Marketplace IDs are included in every query as a filter condition. The backend service layer is responsible for always including this filter:

Every database query includes the marketplace ID as a filter condition, enforced by the backend service layer.

No data is shared between marketplaces — not even lookup tables or reference data. This is stricter than necessary for most cases, but it eliminates an entire class of bugs.

### Tradeoff: Single vs. Separate Databases

| Approach | Pros | Cons |
| -------- | ---- | ---- |
| Single database with marketplace_id filter | Simple to manage, one connection pool | Accidental cross-tenant data leaks possible |
| Separate database per marketplace | Hard isolation, independent scaling | Complex migrations, connection pool per tenant |

A single-database approach with strict query-level filtering was chosen for operational simplicity. The risk of cross-tenant leaks is mitigated by:
- Enforcing marketplace_id in the data access layer
- Regular automated audits verifying isolation
- Read-only replicas that also enforce marketplace isolation

---

## Schema Organization

The schema is organized by business domain, mirroring backend service boundaries. Each domain team owns its tables with no cross-domain foreign key constraints.

### Inventory Domain

The inventory domain manages:

- **Products Catalog** — Core product information: identifiers (ASIN, SKU), pricing, cost settings (COGS), lead times, safety stock levels, and status flags. Each product is uniquely identified by the combination of marketplace, user, and ASIN.
- **Inventory Snapshots** — Historical stock records written during each sync cycle. Stores available, inbound, and reserved quantities per ASIN per day. A generated column calculates total quantity from available + inbound.
- **Shipment Records** — FBA inbound/outbound shipment tracking with status lifecycle (draft → submitted → in-transit → delivered → closed), destination, carrier, tracking, and timing information.
- **3PL Warehouses** — Third-party logistics warehouse configurations with contact information and active status.
- **Production Orders** — Manufacturing lifecycle tracking from purchase order creation through receipt, including supplier, costs, and status transitions.

### PPC Domain

The PPC domain manages:

- **Campaigns** — Records for all ad formats (Sponsored Products, Sponsored Brands, Sponsored Display) with settings for budget, targeting type, bidding strategy, and schedule.
- **Ad Groups** — Groupings within campaigns, each with its own default bid and status.
- **Keywords** — Keyword targeting with match type (exact, phrase, broad) and bid amounts, linked to campaigns and ad groups.
- **Search Terms** — Customer search query performance data with impression, click, spend, and sales metrics. ACOS is calculated as a generated column. Data is retained for a rolling 90-day window.
- **Strategy Configurations** — Strategy engine settings: target ACOS, bid adjustment limits, cooldown periods, and linked campaigns.
- **Strategy Actions** — Append-only audit log of every automated change with before/after snapshots and reasoning.

### Financial Domain

The financial domain manages:

- **Transactions** — Normalized financial records from Amazon reports (orders, refunds, fees, advertising costs, adjustments). Each transaction has a type, category, amount, currency, and date.
- **Reimbursement Claims** — Identified potential claims for lost/damaged inventory or fee errors, with estimated amounts and status tracking.
- **P&L Report Snapshots** — Materialized period-based profitability reports with total revenue, fees, advertising costs, COGS, and net profit (computed via generated column). Full breakdowns stored as JSONB.

### Orders Domain

The orders domain manages:

- **Orders** — Customer order records with status, amounts, and shipping information.
- **Order Items** — Line items within orders linking to ASINs with quantities and pricing.
- **Review Requests** — Customer review tracking: request timing, rating, text, and sentiment classification.

---

## Indexing Considerations

Indexes are designed around actual query patterns:

| Table | Example Indexed Columns | Query Pattern Served |
| ------------- | -------------------------------------------- | --------------------------------------------- |
| campaigns | (marketplace_id, user_id, status, type) | Campaign list queries with filters |
| campaigns | (marketplace_id, status) | Dashboard aggregation queries |
| products | (marketplace_id, user_id, asin) | Product lookup by ASIN |
| products | (marketplace_id, user_id, sku) | Product lookup by SKU |
| keywords | (campaign_id, ad_group_id, match_type) | Keyword management within a campaign |
| search_terms | (campaign_id, date) | Search term reports filtered by date |
| search_terms | (campaign_id, acos) | Finding low-performing terms for negation |
| orders | (marketplace_id, order_date) | Order history queries |
| orders | (user_id, order_date) | Seller-facing order list |
| transactions | (marketplace_id, user_id, transaction_date) | Financial reports by date range |
| strategy_actions | (strategy_id, created_at) | Audit history timeline |
| strategy_actions | (campaign_id, action_type) | Filtering history by campaign |

### Composite Index Design

The most impactful composite index is on `(marketplace_id, user_id, status, type)` for campaigns. Every campaign list query filters by marketplace and user, and most also filter by status and ad type. A composite index with these columns in order means the database can satisfy the entire WHERE clause from the index without touching the table.

For search terms, an index on `(campaign_id, acos)` supports the strategy engine's keyword harvesting queries — finding search terms with low ACOS that are candidate keywords.

### Covering Indexes

Some queries benefit from covering indexes that include all selected columns, avoiding table lookups entirely:

Covering indexes with `INCLUDE` clauses add payload columns without making them part of the index key, keeping the index narrow while serving queries entirely from the index.

The `INCLUDE` clause adds payload columns without making them part of the index key, keeping the index narrow while still serving the query entirely from the index.

---

## Data Retention and Archival

| Data Type | Retention Period | Archival Strategy |
| -------------------- | ---------------- | ------------------------------------------------------ |
| Sales data | Indefinite | Partitioned by month. Old partitions compressed |
| PPC campaign data | Lifetime | Retained with campaign record |
| Search terms | 90 days | Auto-deleted after 90 days (Amazon API limitation) |
| Inventory snapshots | 2 years | Aggregated to weekly averages after 6 months |
| Sync logs | 30 days | Rotated daily |
| Notifications | 90 days | Auto-deleted after 90 days |
| Strategy actions | Indefinite | Append-only audit log. Partitioned by year |
| P&L reports | Indefinite | Stored as materialized JSONB snapshots |

### Partitioning Strategy

Large tables use date-based partitioning. For example, search terms are partitioned by month so the 90-day retention policy is enforced by dropping entire partitions instead of running DELETE statements, which would bloat the table:

```
search_terms_2026_01 (January data)
search_terms_2026_02 (February data)
search_terms_2026_03 (March data)
-- Partition older than 90 days is dropped
```

---

## Caching Strategy

A Redis cache layer reduces read load on frequently accessed data:

| Data | Cache Pattern | TTL | Invalidation |
| ----------------------------- | ------------------------------------ | -------- | ------------------------------- |
| Amazon API access tokens | Cache-aside | 55 min | Expiry (tokens last 60 min) |
| Pricing/plan data | Cache-aside | 5 min | Time-based |
| Campaign list (aggregated) | Cache-aside | 2 min | On campaign update |
| Dashboard summary | Cache-aside | 5 min | New sync data arrives |
| Sync status | Cache-aside | 30 sec | Sync job progresses |
| Rate limit counters | Sliding window in Redis | Automatic | Automatic |

### Cache-Aside Pattern

The application uses a cache-aside pattern:
1. Check Redis for cached data
2. On hit, return immediately
3. On miss, query the database
4. Store result in Redis with TTL
5. Return result

### Cache Invalidation

Stale data is worse than no data in a financial application. Caches are invalidated aggressively:
- **Write-through**: When a campaign is updated, the campaign cache is invalidated in the same transaction
- **Time-based**: Dashboard data has a short TTL because new sync data arrives frequently
- **Manual**: Users can force-refresh from the UI, bypassing the cache

---

## Migration Strategy

Schema migrations follow a strict process for zero-downtime deployments:

1. **Backward-compatible changes only**: Add columns as nullable with defaults. Never drop columns in the same deployment that removes code references.
2. **Expand-contract pattern**: For column renames, add the new column first, deploy code that writes to both, verify, then drop the old column.
3. **Zero-downtime migrations**: Use `CREATE INDEX CONCURRENTLY` for new indexes. Validate constraints after data is consistent.
4. **Rollback scripts**: Every migration has a corresponding rollback that is tested before deployment.

---

## The N+1 Query Problem

A common performance issue in list views is the N+1 query pattern. For example, the campaign list page needs to show performance metrics alongside campaign metadata. An unoptimized approach would:
1. Query campaigns (1 query)
2. For each campaign, query aggregated metrics (N queries)

This is solved with a single query using JOIN and GROUP BY:

The N+1 query problem (e.g., fetching campaign metrics one by one) is solved with a single JOIN and GROUP BY query that fetches all data at once.

---

## Interview Talking Points

**On marketplace-level tenancy:** "Each marketplace is a tenant. When a user switches from US to UK, all queries include the marketplace ID as a filter. This keeps data isolated without requiring separate databases per tenant. We chose this over separate databases because managing many connection pools would add operational complexity without proportional benefit."

**On the indexing strategy:** "Campaign queries filter by marketplace and user on almost every request. A composite index on these columns covers the most common query pattern. Indexes on search term ACOS specifically support the strategy engine's keyword harvesting queries."

**On partitioning for data retention:** "Search terms are partitioned by month. When a partition is older than 90 days, we drop it instead of running DELETE statements. This prevents table bloat and makes retention trivial to enforce."

**On cache invalidation:** "Write operations invalidate the cache in the same transaction. Dashboard data has a short TTL because Amazon data changes throughout the day. Rate limit counters use a sliding window in Redis for sub-second precision."

**On the N+1 prevention:** "The campaign list page would show N+1 query behavior if each campaign's metrics were fetched separately. Using a LEFT JOIN with GROUP BY fetches everything in one query."

---

## Related Documents

- [Multi-Tenant Design](../architecture/multi-tenant-design.md) - Tenant isolation architecture
- [Cloud Infrastructure](../infrastructure/cloud-architecture.md) - Infrastructure and RDS configuration
- [Data Flow](../architecture/data-flow.md) - Data movement through the system
- [Backend Service Architecture](../backend/service-architecture.md) - Service boundaries
