# Custom Date Ranges

Date range management is a cross-cutting concern that affects every data-driven page in the platform. Users set date ranges frequently, and the system remembers their preferences across sessions and modules.

---

## What You Will Learn

- How date ranges are persisted with unique keys
- How the date picker component works
- How different modules have their own date range contexts
- How the date range key system prevents conflicts

---

## Date Range Persistence

Each module that uses date ranges has its own unique key. When a user sets a date range on the PPC dashboard, it is saved with the PPC key. When they navigate to Inventory, the Inventory date range is loaded from its own key.

There are 40+ date range keys in the system. Each module and sub-view has its own key so that date ranges are independent per context.

**Why separate keys?** Users work differently in each module. On the PPC dashboard, they might look at the last 7 days. On the P&L report, they might look at the last quarter. Separate keys let them set different ranges for different modules without manual switching.

---

## Date Picker

The date picker component supports:
- **Preset ranges** — Last 7 days, Last 14 days, Last 30 days, Last 90 days, Last year
- **Custom range** — User selects start and end dates from a calendar
- **Quick select** — Common ranges as one-click buttons

The date picker is used across the dashboard, PPC, reports, inventory, and review modules.

---

## Period Date Range Selector

Some views support period comparison (current period vs. previous period). The selector lets users choose:
- **Period type** — Days, weeks, months, years
- **Period value** — How many periods to show
- **Comparison** — Compare with previous period or same period last year

---

## Interview Talking Points

**On why 40+ date range keys:** "This was a UX decision from observing how users actually work. They set a 7-day range on the PPC dashboard, then switch to the P&L report and set a quarterly range. If the date range was global, they would have to change it every time they switched pages. Separate keys per module let users maintain different time contexts simultaneously."

---

## Related Documents

- [Dashboard Analytics](dashboard-analytics.md) - Dashboard date ranges
- [PPC Performance Analytics](../ppc/performance-analytics.md) - PPC date ranges
- [Profit and Loss](profit-and-loss.md) - Financial period selection
- [Frontend Architecture](../architecture/frontend-architecture.md) - State persistence
