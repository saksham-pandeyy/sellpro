# Account Settings

The Account Settings tab manages Amazon seller account connections. Users can view their connected Amazon accounts, authorize Selling Partner API (SP-API) and Advertising API access, and manage marketplace configurations within each account.

---

## What You Will Learn

- How Amazon accounts are fetched and displayed
- How SP-API and Ads API authorization works from settings
- How PPC Automation is enabled through account settings
- How marketplaces are organized under each account
- How account names can be edited

---

## Amazon Accounts List

The Accounts page fetches connected Amazon accounts from the backend on mount. Each account is displayed as a card with:

- Account name (with edit button)
- SP-API authorization status
- Ads API authorization status
- Marketplace list (expandable accordion)

### Data Fetching

Amazon accounts are fetched from the backend on mount and cached in the Redux store. The component only fetches if accounts are not already loaded, preventing redundant API calls.

Accounts are cached in the `auth` Redux slice under `amazon_accounts`. The component only fetches if accounts are not already loaded.

---

## API Authorization Cards

Each account displays two authorization cards side by side:

### Selling Partner API (SP-API)

The SP-API card shows a button to authorize access to inventory, orders, shipments, and financial data. If already authorized, the button is disabled and shows "Authorized".

The authorization redirects the user to Amazon's Login with Amazon (LWA) page:

The SP-API authorization redirects the user to Amazon's Login with Amazon (LWA) page with the application ID and redirect URL configured in environment variables.

### Advertising API (Ads API)

The Ads API card provides PPC campaign access. Its behavior depends on the user's subscription:

- **If PPC is included** (user has PPC addon or free trial): A direct "Authorize Ads API" button redirects to Amazon's advertising OAuth page
- **If PPC is not included**: The button says "Update Plan" (for free tier users) or "Enable PPC Automation" (for paid users without PPC)

The Ads API authorization button has three states depending on the user's subscription. If PPC access is available, it redirects to Amazon's advertising OAuth page. Otherwise, it triggers an upgrade flow.

---

## Enabling PPC Automation

When a user without PPC clicks the Ads API button, they go through an upgrade flow:

1. A confirmation modal shows the PPC addon price (fetched from pricing data)
2. If confirmed, an API call upgrades the subscription with the PPC addon
3. If the upgrade requires additional payment action, the user is redirected to the upgrade checkout page
4. On success, the user data and subscription are updated in Redux

The pricing is derived from the user's current plan:

The PPC addon price is looked up from cached pricing data by matching the user's billing interval and plan type.

---

## Marketplace Accordion

Each account has an expandable accordion showing its connected marketplaces. Each marketplace displays:

- Country flag icon (USA, Canada, Mexico, Brazil)
- Marketplace name
- A checkmark indicating it is connected

The icon mapping maps marketplace IDs to flag images:

Marketplace IDs are mapped to flag icons using a static lookup object. Only supported marketplaces have flag icons; unsupported ones simply omit the flag.

---

## Empty State

When no Amazon accounts are connected, the page shows an onboarding prompt with:

- Amazon logo image
- "Authorize Your APIs & Unlock your data now!" heading
- Explanation text
- "Proceed to Authorize your APIs" button linking to the `/authorize-apis` page

---

## Interview Talking Points

**On the conditional Ads API button:** "The Ads API button has three states depending on the user's subscription. If they have PPC access, it shows a direct authorize button. If they are on a free plan, it says 'Update Plan'. If they have a paid plan without PPC, it says 'Enable PPC Automation'. Each state leads to a different flow. The pricing and addon information comes from the subscription and pricing data cached in Redux, matching the current plan's billing interval."

**On the marketplace icons approach:** "Marketplace IDs are mapped to flag icons using a static lookup object. Only a subset of marketplaces have flag icons. For unsupported marketplaces, the flag is omitted. This is a simple approach that works for the current set of supported countries."

## Related Documents

- [Settings Overview](overview.md) - Module structure
- [Subscription Settings](subscription-settings.md) - Plan and addon management
- [Amazon Authorization](../authentication/amazon-authorization.md) - Full OAuth flow
- [Onboarding Overview](../onboarding/overview.md) - Initial API authorization
