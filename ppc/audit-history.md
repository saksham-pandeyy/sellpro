# Audit History

The audit history module tracks every change made by the strategy automation engine and manual user actions. It provides a complete timeline of what happened, when, and why.

---

## What You Will Learn

- How the history module captures and stores change events
- How the history sidebar is organized by event type
- How the history detail modal shows before/after snapshots
- How the history is rendered with type-specific components
- How the history can be filtered by campaign, date, or action type

---

## Why Audit History Matters

Automated systems that make financial decisions need complete transparency. If the strategy engine changes a bid, the user needs to know:
- Which campaign was affected
- What the bid was before and after
- Why the engine decided to make the change
- When it happened

Without this audit trail, users would not trust the automation.

---

## History Architecture

Event sources (strategy engine, manual user actions) feed into a history API and database. The frontend reads from this to render:
- A history list page
- A sidebar for quick access
- Individual event display components
- A details modal for full information

---

## History Event Types

The history system handles multiple event types, each rendered by a dedicated display component:

### Campaign Events
- Campaign Created
- Budget Updated
- Campaign Status Changed
- Bidding Strategy Changed

### Keyword Events
- Keyword Added
- Keyword Bid Changed
- Keyword Status Changed
- Negative Keyword Added

### Targeting Events
- Target Added
- Target Bid Changed
- Target Status Changed
- Negative Target Added
- Target Archived

### Placement Events
- Placement Updated
- Bid Adjustment Percentage Changed

### Search Term Events
- Search Term Promoted to Campaign Keyword

---

## History Item Rendering

Each event type has a dedicated component that knows how to render that event's data. A central mapper links event types to their components. This means adding a new event type only requires creating a new component and adding one mapping entry — the rest of the infrastructure stays unchanged.

---

## History Sidebar

The history sidebar provides a quick view of recent changes without leaving the current page. It appears as a slide-out panel.

### Sidebar Features
- **Event list** — Chronological list of recent history items
- **Type filtering** — Filter by event type
- **Date filtering** — Filter by date range
- **Campaign filtering** — Filter by specific campaign
- **Infinite scroll** — Loads more items as the user scrolls

### Sidebar Item Display

Each history item shows:
- Event type icon
- Event description
- Timestamp (relative: "2 hours ago")
- Campaign or resource name

Clicking an item opens the details modal.

---

## History Details Modal

The details modal shows the full information for a history event:

**For Bid Changes:**
```
Campaign: "Summer Sale Campaign - SP"
Keyword: "coffee mug" (Exact Match)
Previous bid: $0.85
New bid: $0.68
Change: -20%
Reason: ACOS exceeded target threshold (35% vs 25% target)
Source: Strategy "Profit Target 25%"
Timestamp: June 4, 2026, 14:32:15 UTC
```

**For Keyword Additions:**
```
Campaign: "Summer Sale Campaign - SP"
Added keyword: "ceramic coffee mug" (Exact Match)
Initial bid: $0.75
Source: Harvested from search term analysis
Original search term performance: 1,450 impressions, 89 clicks, $66.75 spend, $320.00 sales, 20.9% ACOS
Timestamp: June 4, 2026, 14:30:00 UTC
```

---

## History Page

The dedicated history page shows the full history log with advanced filtering:
- Date range selector
- Campaign filter
- Event type filter
- Source filter (automated vs. manual changes)
- Search by campaign name or keyword

Results are paginated.

---

## Interview Talking Points

**On the event-driven architecture:** "Every automated change goes through a single logging function that captures the event type, resource identifiers, before/after values, and the reason for the change. This keeps logging consistent regardless of what triggered the change."

**On the component-per-event pattern:** "Each event type has its own rendering component. This means the team can add a new event type without touching existing event renderers. It is a simple example of the Open/Closed Principle in practice."

**On why this matters for users:** "Without an audit trail, users would not trust the automation. The strategy engine could be making the best decisions in the world, but if users cannot see what it did and why, they would disable it. The history module is the trust layer that makes automation acceptable."

---

## Related Documents

- [PPC Module Overview](overview.md) - Module-level architecture
- [Strategy Automation](strategy-automation.md) - How automated changes are generated
- [Campaign Management](campaign-management.md) - Campaign CRUD
- [Bulk Operations](bulk-operations.md) - Bulk campaign editing
