# Security Architecture

Security is implemented at multiple layers: authentication, session management, API security, data encryption, and input validation. Each layer addresses a specific threat vector.

---

## What You Will Learn

- How authentication tokens are protected at rest
- How sessions are managed and timed out
- How API requests are authenticated and validated
- How sensitive data is encrypted
- How multi-tenant isolation prevents data leakage
- How error handling prevents information disclosure

---

## Authentication Tokens

JWT tokens are encrypted with AES-256 before being stored in localStorage:

JWT tokens are encrypted with AES-256 before being stored in localStorage alongside an expiry timestamp.

Token expiry is checked in two layers:
1. **Client-side expiry** - The `getAccessToken()` function checks the stored expiry timestamp before returning the token
2. **Server-side validation** - The backend validates the JWT signature and expiry on every request

If either check fails, the token is removed from storage and the user is redirected to login.

---

## Session Management

Three layers of session protection run simultaneously:

**Layer 1 - Activity Extender.** On every page navigation, the token's expiry timestamp is prolonged. This keeps the session alive as long as the user is actively navigating.

**Layer 2 - Inactivity Timeout.** A listener watches for mouse moves, key presses, and scroll events. If there is no activity for the configured timeout period (default 30 minutes), the user is logged out.

**Layer 3 - Validity Checker.** A periodic check (every 60 seconds) calls the backend to verify the token is still valid. If the backend reports an invalid token, the user is logged out immediately.

All three layers handle cross-tab synchronization. If a user logs out in one tab, other tabs detect the change through localStorage events and redirect to login.

---

## API Security

Every API request requires a Bearer token in the Authorization header:

Every API request requires a Bearer token in the Authorization header. Requests without a valid token receive a 401 response, triggering a redirect to login.

Requests without a valid token receive a 401 response, which triggers a global redirect to the login page.

CORS is configured to only allow requests from known origins. This prevents other websites from making API calls using the user's credentials.

---

## Data Encryption

| Data                     | Encryption Method        |
| ------------------------ | ------------------------ |
| JWT tokens (at rest)     | AES-256 via CryptoJS     |
| API communication        | TLS/HTTPS                |
| Amazon credentials       | Server-side encryption   |
| Payment data             | Stripe handles encryption |

The encryption key (`REACT_APP_STRING_ENCRYPTION_KEY`) is set as an environment variable and never hardcoded in the source code.

---

## Multi-Tenant Isolation

Data isolation is enforced at the marketplace level:

1. **API queries** include the marketplace ID as a filter condition
2. **Redux slices** separate data by domain but not by marketplace (only one marketplace is active at a time)
3. **Sidebar navigation** changes the active marketplace and triggers a data reload

This prevents a user's US marketplace data from being visible when they switch to their UK marketplace.

---

## Input Validation

Validation happens at two levels:

**Client-side validation** provides immediate feedback using React Hook Form:

Form validation occurs both client-side (for immediate feedback) and server-side (to enforce business rules). Client errors are displayed inline using React Hook Form patterns.

**Server-side validation** enforces business rules and prevents malformed data from being stored. Server errors are returned in a structured format and displayed in the UI:

Server-side validation errors are returned in a structured format and displayed in the UI alongside client-side validation.

---

## Error Handling

The global error handler (`handleException`) intercepts API errors:

| HTTP Code          | How it is Handled                                              |
| ------------------ | -------------------------------------------------------------- |
| 401               | Redirect to login. Clear all stored state                      |
| 403               | Show deactivated account page with support contact             |
| 422               | Display field-level validation error messages                  |
| 429               | Show retry-after message                                       |
| 500               | Show generic error toast. Log details for debugging            |

---

## Force Logout

Admins can force-logout a user's session. The `forceLogout.js` service handles this:

1. Backend invalidates the user's token
2. On the next API request, the user receives a 401 response
3. The `handleException` function detects this and redirects to a session expired page
4. The page displays a message explaining the admin-initiated logout

---

## Interview Talking Points

**On the two-layer token expiry:** "The token has two expiration mechanisms. The JWT itself has an expiration claim that the server validates. Separately, the client stores an expiry timestamp and checks it before making API calls. This prevents expired tokens from being sent to the server unnecessarily. If the client check fails, the user is redirected before any API call is made."

**On cross-tab session sync:** "If a user logs out in one tab, the other tabs detect the change immediately. The session timeout and validity checker both use localStorage events to synchronize across tabs. This prevents the confusing scenario where a user logs out in one tab but is still logged in on another."

## Related Documents

- [Authentication Flow](../authentication/authentication-flow.md) - Registration and login
- [Session Management](../authentication/session-management.md) - Session handling deep dive
- [Amazon Authorization](../authentication/amazon-authorization.md) - Amazon OAuth flow
