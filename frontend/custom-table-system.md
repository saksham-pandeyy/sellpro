# Custom Table System

The custom table is the most reused component in the platform. There are 30+ table instances across the application, and they all use the same underlying component. This document explains how it works and why it was built this way.

---

## What You Will Learn

- Why a custom table was built instead of using a library
- How the table handles sorting, resizing, and column customization
- How column preferences are persisted with checksum validation
- How sticky headers work with the scroll system
- How the custom horizontal scrollbar works
- How checkbox selection and bulk actions are managed

---

## Why Build a Custom Table?

Most React projects use a table library like React Table, Material-UI Table, or AG Grid. This project uses a custom table for three reasons:

1. **Every table needs the same features.** Sorting, column resizing, column visibility toggling, checkbox selection, pagination. If all 30+ tables need these, building them once in a shared component is more efficient than configuring a library 30 times.

2. **Full control over rendering.** The table renders inside a Bootstrap layout with custom CSS. Libraries often fight with custom styling. A custom table renders exactly what we want.

3. **Column persistence is core to the UX.** Users expect their column layout to be remembered between visits. The persistence logic is tightly coupled with the table rendering. A library would add abstraction where we need direct control.

---

## Table Architecture

The CustomTable component accepts props that define its behavior:

The CustomTable component accepts props for column definitions, visible columns, sort state, pagination configuration, checkbox selection, and bulk actions. This abstraction allows 30+ table instances to share the same rendering logic.

---

## Column Definitions

Each table defines its columns through a structure array:

Each column definition includes a field key, display title, data type (for sort behavior), visibility flag, current width, and minimum width. Data types like `string`, `number`, `date`, `dollar_number`, and `percentage` each have their own sort behavior.

---

## Column Visibility

Users can show or hide columns through a dropdown menu. The dropdown lists all available columns with checkboxes next to each one.

Users can show or hide columns through a dropdown menu with checkboxes next to each available column. Toggling a column updates its `is_active` flag in the Redux store.

The column visibility state is stored in Redux and persisted to localStorage. When a column is hidden, the table re-renders with the remaining visible columns and recalculates the total table width.

---

## Column Resizing

Users can resize columns by dragging the right edge of the column header. A resize handle is rendered at the edge of each `th` element.

### How Resize Works

1. User mousedowns on the resize handle
2. A mousemove listener tracks the cursor position
3. The column width is updated based on the delta
4. The new width is stored in a local state object
5. On mouseup, the widths are dispatched to Redux and persisted

### Width Constraints

- Width cannot go below the column's `minWidth`
- Width cannot exceed a reasonable maximum (not explicitly constrained, but the container width limits expansion)
- When the total table width exceeds the container width, a horizontal scrollbar appears

---

## Sorting

Sorting works asynchronously to prevent UI jank on large datasets.

Sorting uses a brief delay to show a loading indicator before applying the sort. This prevents UI jank on large datasets. The sort cycles through ascending, descending, and no sort on each click.

### Sort Types

| Type            | Behavior                                         |
| --------------- | ------------------------------------------------ |
| string          | Alphabetical (A-Z, Z-A)                          |
| number          | Numeric (low-high, high-low)                     |
| dollar_number   | Strips $ and commas, then numeric sort           |
| date            | Chronological (old-new, new-old)                 |
| percentage      | Strips % sign, then numeric sort                 |

### Sort Indicator

The sorted column shows an arrow indicator in the header. Clicking the header cycles through asc, desc, and no sort.

---

## Sticky Headers

When the user scrolls down the page, the table header sticks to the top of the viewport so column labels remain visible.

### Implementation

When the table header scrolls out of view, a duplicate fixed-position header appears at the top of the viewport. This clone stays in sync with the table body's horizontal scroll position.

A duplicate header is rendered as a fixed-position element that appears when the original header scrolls out of view. The sticky header stays in sync with the table body scroll position.

---

## Custom Horizontal Scrollbar

When a table is wider than its container, a custom horizontal scrollbar appears at the bottom of the viewport. This is a UX improvement over the default browser scrollbar which would be at the bottom of the page (potentially thousands of pixels down).

### Implementation

The custom scrollbar is positioned as a fixed element at the bottom of the viewport:

The custom scrollbar is positioned as a fixed element at the bottom of the viewport and syncs its scroll position with the table body. It only appears when the table width exceeds its container.

It syncs its scroll position with the table body:

```javascript
const handleHorizontalScroll = () => {
    stickyHeaderRef.current.scrollLeft = tableBodyRef.current.scrollLeft;
};
```

The scrollbar is only shown when:
1. The table width exceeds the container width
2. The table is within the viewport (not scrolled past)

---

## Checkbox Selection

Users can select rows using checkboxes. A master checkbox in the header selects or deselects all visible rows.

A master checkbox in the header selects or deselects all visible rows. The selected count is shown in a bulk action toolbar.

Selected rows display a visual indicator. The selected count is shown in the bulk action toolbar.

---

## Pagination

The custom pagination component handles large datasets with page navigation.

Features:
- First page, previous page, page numbers, next page, last page
- Ellipsis for large page counts (e.g., "1, 2, ..., 99, 100")
- Responsive: Shows fewer page numbers on mobile
- Page size selector (10, 20, 50, 100 entries per page)
- "Showing X - Y of Z" label

Pagination calculates the page count, current range display (e.g., "Showing 1-20 of 100"), and supports configurable page sizes (10, 20, 50, 100).

---

## Column Persistence with Checksums

Column preferences are saved to localStorage with a checksum to detect schema changes.

Each table instance uses a unique key for persisting column configurations. There are 30+ such keys across the application.

### Save Flow

When saving, the column definitions are serialized with a checksum hash (concatenation of field names). On load, if the saved checksum matches the current schema, the user's preferences are restored; otherwise, defaults are used — automatically handling schema changes.

When a new column is added to a table or an existing column is removed, the checksum changes. All existing user preferences for that table are ignored and the default layout is used. This prevents users from having broken table layouts after an update.

---

## Interview Talking Points

**On building vs buying:** "I chose to build a custom table instead of using a library because we have 30+ table instances that all need the same features. Building it once in a shared component gave us full control over rendering and tight integration with our persistence layer. That said, if I were doing it again, I would consider using a virtualized table library like TanStack React Table for the base functionality and wrapping it with our persistence layer. The custom approach gave us full control, but maintaining scroll sync, resize handles, and sticky headers across browsers took more effort than expected. The core idea was right (share one component across 30 tables), but I would lean on a battle-tested library for the rendering and focus our custom code on the persistence and UX layer."

**On performance with large data:** "The current implementation is not optimized for tables with thousands of rows. The custom table renders all rows in the DOM. For tables with 50 or 100 rows this is fine, but for tables with 1000+ rows it causes performance issues. In a future iteration, I would add windowed rendering (virtualization) so only the visible rows are in the DOM. This is the main tradeoff of the current approach."

**On the checksum approach:** "The checksum system is a detail I am proud of. Users spend time configuring their column layouts. When we add a new column or remove one, those saved layouts become invalid. The checksum detects the schema change automatically and resets to the default layout. Users never see broken tables or missing columns."

**On the sticky header implementation:** "We use a duplicate header approach for sticky headers. The original header scrolls with the page. When it goes out of view, a fixed-position clone appears. The clone stays in sync with the table body scroll position. This avoids the complexity of position: sticky with horizontal scrolling."

---

## Related Documents

- [Frontend Architecture](../architecture/frontend-architecture.md) - Component structure
- [State Management](../frontend/state-management.md) - Redux and persistence
- [UI Design System](../frontend/ui-design-system.md) - Styling approach
- [Data Fetching Strategies](../frontend/data-fetching-strategies.md) - Loading state patterns
