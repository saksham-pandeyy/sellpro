# Session Management

This document explains how user sessions are managed after authentication. It covers JWT handling, session timeout detection, cross-tab synchronization, and how the system protects against session hijacking.

---

## What You Will Learn

- How JWT tokens are stored, encrypted, and validated
- How the three-layer session protection system works
- How sessions stay in sync across multiple browser tabs
- How admin force-logout works
- How the system handles session expiry gracefully

---

## Session Architecture

The session management system has three layers that work together to protect user sessions:

1. **Token Expiration Handler** - Prolongs token validity during active navigation
2. **Session Timeout Handler** - Detects user inactivity and logs out
3. **Token Validity Checker** - Periodically verifies the token with the backend

Each layer runs independently. If any layer detects an issue, the user is redirected to the session expired page.

---

## Layer 1: Token Expiration Handler

This handler runs on every page navigation. It checks whether the stored JWT token is close to expiring and, if so, extends its local expiration timestamp.

On every route change, the handler checks if the stored JWT token is close to expiring and extends its local expiration timestamp. This is a client-side mechanism that prolongs the local expiry, not the actual JWT expiration — keeping the session alive during active navigation.

This is strictly a client-side mechanism. It extends the local expiry timestamp, not the actual JWT expiration. As long as the user is actively navigating, their session stays alive. If they stop navigating and the token's local expiry passes, the next layer takes over.

---

## Layer 2: Session Timeout Handler

This handler monitors user activity and enforces an inactivity timeout. It runs in a useEffect that sets up event listeners and a periodic check interval.

### How It Works

1. When the dashboard loads, the handler records the current time as the last activity time
2. It listens for user activity events: mouse movement, mouse clicks, key presses, scrolling, and touch events
3. These events are throttled (updated at most once every few seconds) to avoid excessive writes
4. Every few seconds, a check interval runs and compares the current time against the last activity time
5. If the idle time exceeds the configured timeout, the user is logged out

### Event Monitoring

The handler listens for user activity events (mouse movement, clicks, key presses, scrolling, touch) using passive listeners to avoid affecting page performance.

The `{ passive: true }` option is important. It tells the browser not to wait for the event handler before scrolling or performing the default action. This prevents the activity monitoring from affecting page performance.

### Cross-Tab Synchronization

The handler also listens for storage events from other tabs:

When one tab detects a session timeout, it writes to localStorage. The `storage` event fires in all other tabs from the same origin, causing them to also redirect to the session expired page.

When one tab detects a session timeout, it writes to localStorage. The storage event fires in all other tabs belonging to the same origin, so they detect the change and redirect to the session expired page.

### Focus and Visibility Changes

The handler also checks the session when the tab regains focus or becomes visible:

The session is also checked when the tab regains focus or becomes visible, catching cases where a user returns after hours away.

This catches cases where the user switches to another tab for hours and comes back. The session check runs immediately on return, rather than waiting for the next interval tick.

---

## Layer 3: Token Validity Checker

This handler periodically makes an authenticated API call to verify the token is still valid on the server side. It runs once per hour.

This handler periodically (once per hour) makes an authenticated API call to verify the token is still valid server-side. If a 401 response is received, it triggers a force logout — catching cases where an admin disabled the account or the password was changed elsewhere.

This layer catches cases where the server has invalidated the token (for example, an admin disabled the user's account or the password was changed from another location).

---

## Logout Flows

### User-Initiated Logout

When a user clicks Logout:

1. Frontend sends a logout request to the API
2. All stored tokens and user data are cleared from localStorage
3. The Redux store is reset to initial state
4. User is redirected to the login page

### Session Expired (Forced Logout)

When any of the three layers detects an issue:

1. `sessionExpired` flag is written to localStorage and sessionStorage
2. After a brief delay, the user is redirected to `/session-expired`
3. The session expired page shows a message explaining what happened
4. The user can click "Log in again" to return to the login page

### Admin Force-Logout

An admin can force-logout a user's session:

1. Admin triggers the action from an admin panel
2. When the user's next request hits the API, it returns a 401
3. The frontend's error interceptor catches the 401
4. Instead of the normal session expired message, it checks the `sessionExpiredByAdmin` flag
5. The session expired page shows a different message indicating an admin logged them out

### Session Expired by Admin

In this case, the frontend also sends a request to clear the user's Firebase Cloud Messaging token, preventing push notifications from continuing to reach the logged-out session.

---

## Force Logout Utility

The core force logout function:

The force logout function writes a `sessionExpired` flag to both localStorage and sessionStorage (for cross-tab detection), optionally sets an admin flag, and redirects after a brief 75ms delay to allow in-progress operations to complete.

The brief delay allows any in-progress operations to complete before the redirect happens. The `sessionExpired` flag in both `localStorage` and `sessionStorage` ensures that:
- Other tabs detect the change via the storage event
- If the user manually navigates back, the session expired page still shows

---

## State Cleanup

When a session ends (logout or expiry), the system clears:

- All access tokens from localStorage
- All cached API data from Redux
- All user preferences (except theme and zoom settings)
- Any polling intervals
- Any notification subscriptions

The theme and zoom preferences are preserved because those are UI settings, not session data. The user should not have to reconfigure their display preferences after logging back in.

---

## Related Documents

- [Authentication Flow](authentication-flow.md) - Login, registration, MFA
- [Amazon Authorization](amazon-authorization.md) - Amazon API authorization
- [Security Architecture](../security/authentication-security.md) - Security best practices
- [Frontend Architecture](../architecture/frontend-architecture.md) - Redux state management
