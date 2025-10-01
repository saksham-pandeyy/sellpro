# Initial Data Synchronization

When a new user authorizes their Amazon account, the system begins an initial data sync that pulls historical data from Amazon's SP-API and Ads API. This process runs as a background job and can take several minutes depending on data volume.

---

## What You Will Learn

- How the initial sync works
- What data is synced and in what order
- How sync status is tracked and displayed
- How the system handles sync errors and retries

---

## Sync Architecture

The data sync is triggered after the user authorizes their Amazon account. The backend dispatches a series of sync jobs, each responsible for pulling a specific data type from Amazon's APIs.

| Sync Step        | Data Pulled                         | Source   | Typical Duration |
| ---------------- | ----------------------------------- | -------- | ---------------- |
| inventory_list   | Current stock levels, inbound shipments | SP-API | 2-5 min          |
| product_list     | Product details, categories, ASINs  | SP-API   | 3-8 min          |
| sales            | Sales history, revenue, units sold  | SP-API   | 5-15 min         |
| shipment_list    | FBA shipment tracking info          | SP-API   | 2-5 min          |
| three_pl_awd     | 3PL warehouse inventory             | SP-API   | 2-5 min          |
| orders           | Customer orders, returns            | SP-API   | 5-15 min         |
| reports          | P&L data, fee reports               | SP-API   | 5-10 min         |
| campaigns        | PPC campaign structure, performance | Ads API  | 3-10 min         |

Steps run sequentially to avoid overwhelming the Amazon API rate limits.

---

## Sync Status Tracking

The sync status is tracked through the `amazon_data_sync_logs` state in the auth reducer. Each sync step has a status: `pending`, `running`, `completed`, or `failed`.

The `amazonDataSyncStatus` function checks whether a specific route's data is still syncing:

Each dashboard route maps to a specific sync step. The inventory page checks the `inventory_list` step, the products page checks `product_list`, and so on. Sync indicators only appear for data relevant to each page.

This function supports two API response shapes:
- **New format:** `{ session: { status, steps: [] }, summary: {} }`
- **Legacy format:** `[ { name, first_synced_at, status }, ... ]`

---

## Sync Indicator in the UI

The sidebar shows a spinning sync indicator next to menu items whose data is still loading:

Sync labels in the sidebar use the route-to-step mapping to show animated indicators only for steps still in progress.

The DashboardAnalyticsPage shows a banner when any sync is in progress:

> "The data syncing process is ongoing! Please wait for the analytics to update."

---

## Error Handling

If a sync step fails, it is retried automatically. After a configurable number of failures, the step is marked as failed and the user is notified through the notification system. The user can manually trigger a re-sync from the settings page.

---

## Interview Talking Points

**On the route-to-step mapping:** "Each route knows which sync step it depends on. The inventory page checks inventory_list, the products page checks product_list. Sync indicators only appear for data the user actually cares about on each page. This was a UX decision to avoid showing a spinning icon for every menu item when only one data type is syncing."

**On the two API shapes:** "The sync function handles both new and legacy API response formats. The new format wraps sync steps in a session object with summary data. The legacy format returns a flat array. Supporting both allowed the backend team to migrate incrementally without breaking the frontend."

## Related Documents

- [Onboarding Overview](overview.md) - Onboarding funnel
- [Amazon Authorization](../authentication/amazon-authorization.md) - Amazon OAuth flow
- [Data Flow](../architecture/data-flow.md) - Data movement through the system
