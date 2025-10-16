# Real-Time Signals

Real-time signals use Firebase Firestore to deliver instant notification updates to the frontend. When a new notification arrives, the badge count updates without the user needing to refresh.

---

## What You Will Learn

- How Firestore is used for real-time notification signals
- How the onSnapshot listener works
- How the notification signal is structured
- How the frontend handles signal updates

---

## Why Real-Time Signals Matter

Push notifications through FCM are one-way. The backend sends a message, the device shows a notification. But the in-app notification panel needs to update in real-time. When a new alert arrives, the badge count should update immediately. Firestore provides this capability.

---

## Firestore Document Structure

Each user has a Firestore document that tracks notification summary data:

```
Collection: users
  Document: {userId}
    Collection: notifications
      Document: summary
        Fields:
          - unread_count: number
          - last_updated: timestamp
```

The document is updated by the backend whenever a new notification is created or a notification is marked as read.

---

## Frontend Listener

The frontend uses Firestore's `onSnapshot` method to subscribe to real-time updates:

The frontend subscribes to a Firestore document using `onSnapshot`. The first snapshot is skipped because initial data comes from the REST API. Only subsequent updates trigger badge count changes.

The first snapshot is skipped because the notification data is initially loaded through the REST API. Only subsequent updates (which represent new notifications) trigger badge count changes.

---

## Signal Data

The signal object received from Firestore:

The signal object contains the unread notification count and can be extended with additional metadata fields as needed.

When the signal is received, the frontend updates the Redux store with the new unread count:

When a new signal is received, the Redux store is updated with the new unread count, which automatically propagates to the navbar badge counter.

---

## Error Handling

The Firestore listener includes error handling to prevent crashes:

The Firestore listener includes error handling that silently ignores expected permission errors (like `admin-restricted-operation` in test environments) while logging unexpected errors.

The `admin-restricted-operation` error is expected in some test environments and is handled silently.

---

## Interview Talking Points

**On why Firestore and not just FCM:** "FCM push notifications are one-way. The backend sends a message, but the frontend has no way to know a new notification arrived unless it polls. Firestore's onSnapshot gives us real-time updates without polling. When the backend writes to Firestore, every open browser tab gets the update instantly."

**On skipping the first snapshot:** "The onSnapshot fires immediately with the current document state. We skip that first event because the initial data was already loaded via the REST API. Only subsequent updates (new notifications) should trigger badge changes. This prevents a flash of wrong data on page load."

## Related Documents

- [Notification Architecture](notification-architecture.md) - System overview
- [Push Notifications](push-notifications.md) - FCM integration
- [Notification Panel](notification-panel.md) - UI component
- [Firebase Integration](../integrations/firebase-services.md) - Firebase setup
