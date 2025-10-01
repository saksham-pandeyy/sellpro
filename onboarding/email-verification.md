# Email Verification

Email verification is a required step after registration. Users must verify their email address before they can access the dashboard or connect their Amazon account.

---

## What You Will Learn

- How the email verification flow works
- How the verify email page handles tokens
- How users can request a new verification email
- How the system enforces verification

---

## Verification Flow

1. User registers with email and password
2. Backend sends a verification email with a signed token
3. User clicks the link and lands on the verify email page
4. The page validates the token and marks the email as verified
5. User is redirected to the next onboarding step

---

## Verify Email Page

The VerifyEmailPage handles two scenarios:

**Direct verification** - User clicks the link in their email and lands on the page with a valid token. The page calls the verification API endpoint, and on success, updates the Redux store and redirects to the onboarding flow.

**Manual resend** - Users who did not receive the email can request a new one. The page calls the resend endpoint, which triggers another verification email from the backend.

---

## Enforcement

The LoginPage checks the user's email verification status after login:

```javascript
if (result.email_verified_at) {
    // Proceed to dashboard or plan selection
} else {
    navigate('/verify-your-email');
}
```

Users who try to access dashboard routes without a verified email are redirected back to the verification page.

---

## Interview Talking Points

**On the verification flow:** "Email verification is enforced at the route level. The login redirect checks `email_verified_at` and sends unverified users to the verification page. This keeps the check simple and prevents users from getting into the app without a verified account."

## Related Documents

- [Authentication Flow](../authentication/authentication-flow.md) - Full registration and login flow
- [Onboarding Overview](overview.md) - Onboarding funnel
