# State Management

The application uses Redux Toolkit with 20+ domain-specific slices. Each business domain has its own slice, and a global app slice handles cross-cutting concerns like loading states and theme preferences.

---

## What You Will Learn

- How the Redux store is structured
- How slices are organized and why
- How the root reducer handles logout resets
- How selectors access state efficiently
- How middleware handles side effects

---

## Store Structure

The store is configured in `src/store.js` using Redux Toolkit's `configureStore`. Each business domain gets its own slice:

Each business domain gets its own slice registered with `configureStore`. Slices are organized by domain to mirror the backend service structure, keeping the mental model consistent across the stack.

---

## Slice Organization

Slices are organized by business domain, mirroring the backend service structure:

| Slice          | State                    | Key Data                                               |
| -------------- | ------------------------ | ------------------------------------------------------ |
| auth           | User, tokens, sync status | `auth_user`, `access_token`, `amazon_data_sync_logs`  |
| app            | Global UI state          | `loader`, `theme`, `zoom_level`                       |
| campaign       | PPC campaign data        | Campaign lists, performance metrics                   |
| dashboard      | Dashboard analytics      | Summary cards, chart data                              |
| inventory      | Stock levels, health     | Inventory list, stock status                           |
| product        | Product details          | ASIN data, costs, images                               |
| sales          | Sales history            | Revenue, units, comparisons                            |
| orders         | Customer orders          | Order list, review status                              |
| shipments      | FBA shipments            | Shipment tracking, status                              |
| notifications  | In-app notifications     | Notification list, unread count                        |

---

## Logout Reset Pattern

Every slice resets to its initial state when the user logs out. This is implemented through a root reducer that intercepts the logout action:

When a logout action is dispatched, the root reducer passes `state = undefined` to every combined reducer, causing each slice to return its initial state. This pattern avoids manually resetting each slice.

When `state = undefined`, each slice reducer returns its own `initialState`, effectively clearing all cached data. This prevents stale data from persisting when a new user logs in on the same browser.

---

## Data Flow Pattern

Data flows through the application in a consistent pattern:

1. **Component mounts** - Checks if data exists in Redux
2. **Fetch if needed** - Dispatches a thunk that calls the API
3. **Store response** - API response is saved to the Redux slice
4. **Render from state** - Components read from Redux via `useSelector`
5. **Update on action** - User actions dispatch slice reducers that update state

---

## Selector Usage

Components use `useSelector` to read from specific slices:

Components read from Redux using `useSelector` selectors that drill into the relevant slice. Selectors are kept simple — computed transformations are handled at the component level.

Selectors are kept simple. For computed data, the component handles the transformation rather than creating memoized selectors. This keeps the pattern consistent and easy to follow.

---

## Interview Talking Points

**On the logout reset pattern:** "When a user logs out, Redux sets state to undefined, which causes every slice reducer to return its initial state. This clears all cached data and prevents the next user from seeing stale info. The alternative is manually resetting each slice, which is error-prone when adding new slices."

**On slice boundaries:** "Each business domain has its own slice. Campaign data lives in the campaign slice, orders in the orders slice. This separation lets me work on the PPC module without worrying about breaking inventory state. The domain boundaries match the backend service boundaries, making the mental model consistent across the stack."

## Related Documents

- [Frontend Architecture](../architecture/frontend-architecture.md)
- [Component Architecture](component-architecture.md)
- [Data Flow](../architecture/data-flow.md)
