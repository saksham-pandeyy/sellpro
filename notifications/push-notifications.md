# Push Notifications

Push notifications are delivered through Firebase Cloud Messaging (FCM). They work even when the browser tab is in the background or closed. This document covers the full FCM integration architecture, service worker lifecycle, device token registration, foreground and background message handling, and graceful degradation.

---

## What You Will Learn

- How FCM is configured on the frontend with environment variable validation
- How the service worker handles background messages independently of the React app
- How device tokens are registered, stored, and managed for push delivery
- How foreground messages are handled differently from background messages
- How the Firebase configuration check enables graceful degradation
- How the FCM token cleanup works on logout and force logout
- How the app integrates with Firebase Authentication using custom tokens

---

## Firebase Setup and Configuration

### Environment Variable Validation

Firebase configuration is loaded from environment variables and validated before initialization. If any required key is missing or is a placeholder value, Firebase initialization is skipped entirely and the notification system is gracefully disabled — no crashes, no errors, no blank screens — users simply do not get push notifications.

---

## Service Worker Architecture

### Service Worker Registration

The service worker is registered only when Firebase is properly configured and the browser supports service workers. Registration is wrapped in a conditional check to avoid errors in unsupported environments.

### Service Worker Code

The service worker script initializes Firebase independently from the main app and handles two key events: `onBackgroundMessage` for incoming push notifications while the tab is backgrounded, and `notificationclick` for when the user clicks the browser notification. The click handler checks for an existing tab before opening a new one.

### Service Worker Lifecycle

- **Install**: Service worker is downloaded and installed
- **Waiting**: If there is an existing service worker, the new one waits for activation
- **Activate**: The service worker starts controlling pages
- **Running**: The service worker listens for push events and notification clicks

The `notificationclick` event handler is critical. Without it, clicking a browser notification would do nothing. The handler:
1. Closes the notification
2. Checks if an existing tab is open for the application
3. If yes, focuses it and navigates to the relevant page
4. If no, opens a new tab at the relevant page

---

## Device Token Registration

### Requesting Permission

Notification permission is requested from the browser. If granted, an FCM device token is obtained using the VAPID key from environment configuration. If permission is denied, push notifications are silently disabled.

### Sending Token to Backend

The FCM device token is sent to the backend for storage. Token registration failure is non-critical — the app continues to work, just without push notifications.

### Token Refresh Handling

FCM tokens can be refreshed by the browser over time. Firebase provides a callback:

FCM tokens can be refreshed by the browser over time. Firebase provides a callback to detect refresh events and re-register the new token with the backend.

### Token Cleanup on Logout

When a user logs out, the FCM token is removed from the backend to prevent push notifications from continuing to a logged-out session. This cleanup is non-critical — the app continues to work without it.

---

## Foreground Message Handling

When the app is in the foreground, messages are handled through the `onMessage` callback:

When the app is in the foreground, the `onMessage` callback handles incoming notifications differently depending on their type: report completions trigger a toast with a "View Report" button, alerts update the in-app notification panel and badge count, and other messages display as browser notifications.

---

## Firebase Authentication Integration

The platform uses Firebase Authentication alongside its own JWT-based auth. When a user logs in, the backend generates a custom Firebase auth token signed with the service account key. This custom token allows the backend to control which users can access Firebase resources (like Firestore documents). The token is short-lived and signed by the backend service account.

---

## Graceful Degradation

If Firebase is not configured or any Firebase feature fails, the system degrades gracefully:

| Scenario                        | Behavior                                                |
| ------------------------------- | ------------------------------------------------------- |
| Firebase not configured         | No feature initialization. No errors. App works normally |
| Notification permission denied  | Push notifications disabled. In-app notifications work  |
| FCM token registration fails    | Push notifications disabled. Other Firebase features work |
| Firestore listener fails        | Badge count updates via REST API polling instead         |
| Firebase auth fails             | Firestore access limited. Push notifications still work |
| Service worker registration fails | Foreground push handling still works                 |

**No crashes. No console errors. No blank screens.**

---

## Interview Talking Points

**On the service worker design:** "The service worker runs independently of the React app. When a push notification arrives while the tab is in the background, the service worker handles it, shows a browser notification, and stores the notification data. When the user clicks the notification, the service worker navigates them to the relevant page. This works even if the React app has not finished loading yet."

**On the notification click handling:** "The notificationclick event handler is the most important part of the service worker. Without it, clicking a browser notification does nothing. We implemented it to check for an existing tab first and focus it before navigating, rather than always opening a new tab. This prevents the user from getting multiple tabs from multiple notification clicks."

**On the graceful degradation philosophy:** "Firebase requires 6 environment variables to be configured. If any one is missing, we skip Firebase initialization entirely. The app works perfectly without push notifications -- users just do not get browser notifications. This is better than throwing an error or showing a broken notification UI."

**On the custom Firebase auth token:** "The platform uses its own JWT-based authentication. To enable Firestore access, the backend generates a custom Firebase auth token that identifies the user to Firebase. This token is short-lived and signed with the Firebase service account key. It allows us to control Firestore access from the backend instead of exposing Firebase Admin SDK credentials to the frontend."

---

## Related Documents

- [Notification Architecture](notification-architecture.md) - Multi-channel delivery system
- [Real-Time Signals](real-time-signals.md) - Firestore integration for badge sync
- [Notification Panel](notification-panel.md) - Slide-out notification UI
- [Firebase Integration](../integrations/external-integrations.md) - Firebase configuration
- [Authentication Flow](../authentication/authentication-flow.md) - Custom token integration
