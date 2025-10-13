# Notification Panel

The notification panel is a slide-out UI component in the top navbar. It displays all user notifications in a grouped, scrollable list with actions for managing them.

---

## What You Will Learn

- How the panel is structured with header, list, and footer
- How infinite scrolling loads more notifications
- How notifications are grouped and expanded
- How read/unread state is managed
- How clicking a notification navigates to the relevant page

---

## Panel Structure

The panel slides out from the right side of the viewport:

```mermaid
graph TB
    subgraph "Notification Panel"
        A[Header]
        B[Scrollable List]
        C[Load More Sentinel]
    end

    subgraph "Header"
        D[Title: "Notifications"]
        E["Mark All Read" Button]
        F[Close Button]
    end

    subgraph "Notification Item"
        G[Icon]
        H[Title & Message]
        I[Timestamp]
        J[Expand Chevron]
    end

    A --> D
    A --> E
    A --> F
    B --> G
    B --> H
    B --> I
    B --> J
```

---

## Header

The header shows:
- "Notifications" title
- "Mark all read" button (only visible when there are unread notifications)
- Close button (or Escape key to close)

---

## Notification List

Each notification item shows:

- **Icon** - Different icon per type (alert bell, tip lightbulb, info circle)
- **Title** - Description of the notification
- **Message** - Additional detail (if available)
- **Timestamp** - Relative time (from moment: "2 hours ago", "yesterday")
- **Unread indicator** - Styled background for unread items
- **Expand chevron** - For grouped notifications with multiple items

### Read/Unread Styling

- Unread notifications have a colored left border and bold title
- Read notifications have a muted background and normal weight title
- The icon background color also changes based on read status

---

## Grouped Notifications

When multiple related notifications exist (like 5 low stock alerts), they are grouped into a single item:

- Shows count: "5 inventory alerts"
- Clicking the chevron expands to show individual items
- Individual items can be clicked separately
- The expanded view shows a nested list with sub-items

---

## Infinite Scroll

The notification list uses an IntersectionObserver to detect when the user scrolls near the bottom:

An IntersectionObserver watches a sentinel element near the bottom of the notification list. When it enters the viewport, the next page of notifications is fetched and appended. A 200px root margin provides a buffer so the next page starts loading before the user reaches the bottom.

This loads more notifications as the user scrolls down, 20 items at a time.

---

## Click Actions

Clicking a notification navigates to the relevant page:

| Notification Type       | Navigates To                           |
| ---------------------- | -------------------------------------- |
| Alert                  | Smart alerts page with alert ID or IDs |
| Tip                    | Tips page with tip ID                  |
| Price checker experiment | Price checker experiment detail      |
| Futuristic report      | Futuristic data calendar               |

---

## Mark as Read

When a notification is clicked, it is marked as read:

When a notification is clicked, the UI marks it as read immediately (optimistic update) while the backend call happens in the background. If the API call fails, the read state is reverted.

The "Mark All Read" button sends a bulk request:

The "Mark All Read" button sends a bulk request and refreshes the notification list on success.

---

## Badge Counter

The navbar bell icon shows a badge with the unread count. The badge is updated in real-time through Firestore signals.

---

## Interview Talking Points

**On the optimistic update pattern:** "When a user marks a notification as read, the UI updates immediately (optimistic update) and the API call happens in the background. If the API call fails, the read state is reverted. This makes the panel feel instant while maintaining data consistency."

**On the infinite scroll with IntersectionObserver:** "We use IntersectionObserver instead of scroll events for the notification list. It is more performant because the browser fires the callback only when the sentinel element enters the viewport, not on every pixel scroll. The 200px root margin gives us a buffer so the next page starts loading before the user reaches the bottom."

---

## Related Documents

- [Notification Architecture](notification-architecture.md) - System overview
- [Push Notifications](push-notifications.md) - FCM integration
- [Real-Time Signals](real-time-signals.md) - Firestore integration
- [Report Polling](report-polling.md) - Async report notifications
