# Component Architecture

The frontend component architecture follows a domain-driven structure. Each business module owns its components, Redux slice, and service integrations. This keeps the codebase organized as it grows.

---

## Directory Structure

```
src/
  components/
    common/       # Shared components used across features
    front/        # Public-facing pages (login, signup, homepage)
  pages/
    dashboard/    # Authenticated dashboard features
      inventory/
      ppc/
    front/        # Public pages
  reducer/        # Redux slices
  services/       # Business logic and API helpers
  network/        # HTTP client and URL management
```

---

## Component Hierarchy

```
App
├── FrontendRoutes (public)
│   ├── IndexPage
│   ├── LoginPage
│   ├── SignupPage
│   ├── PricingPage
│   ├── AboutUsPage
│   └── ...
└── DashboardRoutes (authenticated)
    ├── Navbar
    │   ├── SearchBar
    │   ├── NotificationPanel
    │   └── UserMenu
    ├── Sidebar
    │   ├── MenuItems
    │   └── SyncLabel (per item)
    └── Main Content
        ├── DashboardAnalyticsPage
        ├── InventoryPage
        │   ├── InventoryTable
        │   ├── FuturisticCalendar
        │   └── SyncIndicator
        ├── PPC Campaigns
        │   ├── CampaignTable
        │   ├── StrategyEngine
        │   └── HistorySidebar
        └── ...
```

---

## Common Components

Shared components live in `components/common/` and are reused across features:

| Component             | Purpose                                 | Used In              |
| -------------------- | --------------------------------------- | -------------------- |
| CustomTable          | Sortable, resizable, paginated table    | Every list page      |
| CustomPagination     | Page navigation controls                | Every list page      |
| ColumnSelectorDropdown | Show/hide table columns               | Every table          |
| CurrencyFormatter    | Format monetary values                  | Finance displays     |
| Loader               | Full-page loading spinner               | Route transitions    |
| SectionLoader        | Section-level skeleton loading          | Content areas        |
| SyncingLoader        | Data sync progress indicator            | Dashboard            |
| ConfirmActionModal   | Confirmation dialog                     | Delete/update actions |
| SearchBar            | Text search with autocomplete           | List filters         |
| ScaleWrapper         | Zoom level scaling                      | Full app             |

---

## Page Components

Each page is a self-contained component that:
1. Reads initial data from Redux or fetches it via API
2. Renders the page layout with child components
3. Handles user interactions and dispatches Redux actions

Pages do not share state directly. They read from shared Redux slices.

---

## Component Patterns

**Container pattern.** Pages are containers that orchestrate data fetching and state. Child components are presentational and receive data through props. This makes child components reusable and testable.

**Conditional rendering.** Components check data availability before rendering. If data is still loading, skeleton placeholders are shown. If data is empty, a SearchNoResult component is displayed with appropriate messaging.

**Route-level data fetching.** The DashboardRoutes component checks auth status, sync status, and user plan on every route change. Individual pages fetch domain-specific data when they mount.

---

## Interview Talking Points

**On the domain-driven structure:** "Each feature module has its own folder with components, Redux slice, and service integrations. The PPC module does not reach into inventory code. This means a team can work on PPC while another works on inventory without merge conflicts. It also makes each module independently testable."

**On the container pattern:** "Pages handle data loading and state management. Child components receive data through props and handle presentation. This separation means I can change the data source without touching the UI, or redesign the UI without changing how data is loaded. Both concerns evolve independently."

## Related Documents

- [Frontend Architecture](../architecture/frontend-architecture.md) - Full frontend architecture
- [State Management](state-management.md) - Redux store design
- [UI Design System](ui-design-system.md) - Theming and responsive design
