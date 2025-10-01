# Onboarding Module

The onboarding module guides new users from signup to a fully connected Amazon account with synced data. It handles account creation, Amazon authorization, and the initial data synchronization process.

---

## What You Will Learn

- How the signup and login flow works
- How Amazon account authorization is handled
- How the initial data sync works
- How signup intents preserve plan selections across redirects
- How users progress through the onboarding funnel

---

## Onboarding Funnel

```
Signup / Login --> Email Verification --> Choose Plan --> Authorize APIs --> Data Sync --> Dashboard
```

Each step must be completed before the user can access the dashboard. The system enforces this funnel:

1. **Signup** - Create an account with email/password or Google OAuth
2. **Email Verification** - Verify email address before proceeding
3. **Plan Selection** - Choose a subscription plan (or free trial)
4. **Amazon Authorization** - Grant SP-API and Ads API permissions
5. **Data Sync** - Initial data pull from Amazon
6. **Dashboard** - Full platform access

---

## Signup Flow

The signup page supports two registration methods:

**Email Registration** - Users enter their full name, email, and password. The form validates all fields client-side before submitting to the backend. On success, the user receives an access token and is redirected to email verification.

**Google OAuth** - Users can sign up with their Google account for a faster registration flow. The GoogleAuthUI component handles the OAuth redirect and callback.

Both flows support signup intents, which preserve plan selections made on the pricing page before registration.

---

## Signup Intents

Signup intents solve a specific UX problem: users select a plan on the pricing page, then need to create an account. The intent token preserves their plan choice through the registration flow.

The intent token captures the selected plan ID, name, type, billing interval, and any addons (PPC, additional ASIN capacity). It is created on the pricing page, stored in localStorage, and attached to the signup request. After registration, the intent is cleared.

The intent is created on the pricing page, stored in localStorage, and attached to the signup request. After registration, the intent is cleared.

---

## Amazon Authorization

After signup and plan selection, users must authorize the platform to access their Amazon seller data. This is handled through two API integrations:

**SP-API Authorization** - Grants access to sales, inventory, orders, and fulfillment data. The user is redirected to Amazon's Login with Amazon (LWA) page, then back to the platform with an authorization code.

**Ads API Authorization** - Grants access to advertising data (PPC campaigns, keywords, search terms). Separate OAuth flow through Amazon's advertising console.

The AuthorizeAPIs page handles both flows and checks the authorization status before allowing users to proceed.

---

## Data Synchronization

Once authorized, the system begins the initial data sync. The sync runs as a background process that pulls data from Amazon's APIs:

| Sync Step        | Data                              | Source  |
| ---------------- | --------------------------------- | ------- |
| inventory_list   | Stock levels, inbound quantities  | SP-API  |
| product_list     | Product details, categories       | SP-API  |
| sales            | Sales history, revenue            | SP-API  |
| shipment_list    | FBA shipment tracking             | SP-API  |
| three_pl_awd     | 3PL warehouse data                | SP-API  |
| orders           | Customer orders, returns          | SP-API  |
| reports          | Profit and loss data              | SP-API  |
| campaigns        | PPC campaign data                 | Ads API |

The `amazonDataSyncStatus` function checks which steps are still running and shows sync indicators in the sidebar. Users can see which data is still being loaded and which is ready to use.

---

## Route Protection

The onboarding funnel is enforced through route-level checks in DashboardRoutes.js:

Route-level guards check the authorization and sync status. Users who have not completed all onboarding steps are redirected to the appropriate page, preventing access to features before prerequisites are met.

Users who have not completed all onboarding steps are redirected to the appropriate page. This prevents access to dashboard features before the required setup is complete.

---

## Interview Talking Points

**On the signup intent pattern:** "Signup intents preserve plan selections across the registration redirect. When a user picks a plan on the pricing page and then signs up, the intent token carries their plan choice through the OAuth flow. This prevents losing the selection when the user is redirected to Google or Amazon for authorization."

**On the funnel enforcement:** "Each onboarding step has a guard in the route logic. The system checks email verification, plan status, and API authorization before granting dashboard access. This prevents users from landing on pages where data is not available because prerequisites are missing."

**On data sync status:** "The sync status function maps routes to sync steps. The inventory page checks the inventory_list step, the products page checks product_list. This way, sync indicators only show for data the user actually cares about on each page."

## Related Documents

- [Amazon Authorization](../authentication/amazon-authorization.md) - Amazon API OAuth flow
- [Authentication Flow](../authentication/authentication-flow.md) - Registration and login
- [Session Management](../authentication/session-management.md) - Session handling
- [Data Flow](../architecture/data-flow.md) - How data moves through the system
