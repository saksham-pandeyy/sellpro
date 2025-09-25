# Data Fetching Strategies

The application uses several data fetching patterns depending on the use case: prefetching, polling, lazy loading, and direct API calls. Each pattern addresses a specific performance or UX requirement.

---

## What You Will Learn

- How the prefetching system works
- How report polling is implemented
- How lazy data loading reduces initial load time
- How skeleton loading improves perceived performance

---

## Prefetching

The `ApiDataAutoComponent` prefetches dashboard data in the background. When a user navigates to a page that needs data, the system checks if the data was already loaded:

Data is only fetched if it's not already present in Redux. This prevents redundant API calls when navigating back to a previously visited page.

This prefetching is conditional. It only fetches data that has not been loaded yet, preventing redundant API calls when navigating back to a previously visited page.

---

## Report Polling

Long-running operations (like generating futuristic data reports) use a polling pattern. The `GlobalReportPolling` service checks for report completion at regular intervals:

The `GlobalReportPolling` service checks for report completion at 30-second intervals, up to 5 attempts (2.5 minutes total). After the maximum attempts, it shows an informational message and stops silently.

The poller is implemented as a singleton. Starting a second poll while one is already running reuses the existing poller instance. This prevents multiple polling loops from running simultaneously.

When the report is ready, the system shows a toast notification with a "View Report" button that navigates to the report page.

---

## Lazy Loading

Large datasets (campaign lists, search terms, product lists) are loaded through pagination. Each page request fetches only the data needed for the current view:

- **API pagination** - The backend returns paginated results with total count
- **Client-side caching** - Loaded pages are cached in Redux
- **Infinite scroll** - Notification panel loads more items as the user scrolls

---

## Skeleton Loading

When data is being fetched, the UI shows skeleton placeholders that match the layout of the actual content:

| Component               | Purpose                              |
| ----------------------- | ------------------------------------ |
| SkeletonTable           | Table row placeholders               |
| SkeletonCard            | Card layout placeholders             |
| SkeletonBarChart        | Chart area placeholders              |
| SkeletonLineChart       | Line chart placeholders              |
| SkeletonGauge           | Gauge widget placeholders            |
| SkeletonCalender        | Calendar grid placeholders           |
| SkeletonCircleChart     | Donut chart placeholders             |
| SkeletonHistory         | History timeline placeholders        |
| SkeletonFuturisticTable | Futuristic data table placeholders   |
| SkeletonContainer       | Generic container skeleton           |

Skeletons are shown immediately while the API request is in flight, making the app feel faster than showing a spinner.

---

## Direct API Calls

For user-initiated actions (saving a form, updating settings, triggering a sync), components make direct API calls through the `requests.js` service:

Components use a centralized request service that handles Bearer token injection, global error handling, and response parsing automatically.

The request service handles:
- Bearer token injection via `authHeader()`
- Global error handling via `handleException()`
- Response parsing

---

## Interview Talking Points

**On the singleton poller:** "There is only one polling loop for the entire application. If a user starts a second report, it does not create a second poller. It just adds the new report ID to the existing loop. This prevents the app from creating dozens of polling timers if the user triggers multiple reports."

**On skeleton loading:** "Each skeleton component matches the exact layout of the content it replaces. A table skeleton has the same number of rows and columns as the actual table. This makes the transition from loading to loaded seamless because the layout does not jump when data arrives."

## Related Documents

- [Frontend Architecture](../architecture/frontend-architecture.md)
- [Report Polling](../notifications/report-polling.md) - Polling architecture and implementation
