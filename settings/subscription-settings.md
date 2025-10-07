# Subscription Settings

The Subscription Settings tab displays the user's current plan, manages PPC automation, adjusts ASIN tracking capacity, and provides subscription lifecycle controls. It is one of the most feature-dense settings pages.

---

## What You Will Learn

- How subscription details are fetched and displayed
- How the PPC Automation addon is toggled
- How ASIN tracking capacity is managed (Enterprise plans)
- How the cancel subscription flow works
- How the retry payment flow works for past-due subscriptions
- How the subscription polling mechanism works

---

## Subscription Details

The page fetches subscription data from the backend on mount. It normalizes the response into a consistent format:

The subscription data from the API is normalized into a consistent format that the UI can rely on. Fields like `billing_interval`, `status`, `price`, and dates are mapped from their various possible source field names so the display logic remains simple and predictable.

The plan details section shows:

| Field          | Description                              |
| -------------- | ---------------------------------------- |
| Current Plan   | Plan name (e.g., Scale, Growth, Enterprise) |
| Billing Cycle  | Monthly or Yearly with start/end dates   |
| Plan Status    | Color-coded status badge                 |

Status badges are color-coded:
- **green/paid** for active subscriptions
- **red/cancelled** for expired, past_due, unpaid, canceled, or incomplete_expired

---

## Manage Your Plan

Below the plan details, a "Manage Your Plan" section provides action cards.

### Select ASINs to Track

Users can open the ASIN selection modal to choose which ASINs the platform monitors. The button shows current usage:

```
Manage ASINs (12 / 50 used)
```

The ASIN limit depends on the plan:
- **Scale**: 50 ASINs
- **Growth**: 20 ASINs
- **Other**: 10 ASINs

The `AsinSelectionModal` component handles the selection UI and saves changes via the backend.

### PPC Automation Toggle

A toggle switch enables or disables the PPC Automation addon. When toggling on:

1. A confirmation modal appears showing the PPC addon price (if available from pricing data)
2. On confirm, the subscription is upgraded via `POST /api/v1/subscriptions/upgrade`
3. If the payment requires additional verification (SCA), the user is redirected to the upgrade checkout page
4. On success, the subscription and user data are updated in Redux

When toggling off, a simpler confirmation flow disables the addon without additional payment steps.

### Additional ASINs (Enterprise Only)

For Enterprise plan users, an additional card allows updating ASIN tracking capacity:

1. User enters a number in an input field
2. Clicking "Update Capacity" shows a confirmation with the per-ASIN price
3. On confirm, the subscription is upgraded with the new ASIN count
4. If payment verification is needed, the user is redirected to the upgrade checkout

The ASIN addon price is derived from the pricing data:

The ASIN addon price is looked up from the cached pricing data by matching the current plan's billing interval and plan name. This ensures the correct price is shown based on the user's subscription tier.

---

## Cancel Subscription

Active subscriptions (non-trial) show a "Cancel Subscription" button. Clicking it opens a confirmation modal explaining that the plan remains active until the end of the current billing period. On confirm, a `POST /api/v1/subscriptions/cancel` request is sent.

---

## Payment Retry

When a subscription has a `past_due`, `unpaid`, or `incomplete` status, a warning banner appears at the top:

> "Your last payment failed. Please retry to avoid service interruption."

A "Retry Payment" button navigates to `/pay-invoice` where the `RetryPaymentComponent` handles payment retry.

---

## Subscription Polling

The subscription settings page implements a polling mechanism for subscriptions in `past_due` or `plan_due` status:

The subscription poller checks the subscription status endpoint every 10 seconds for up to 9 attempts (90 seconds total). If the status changes to active during this window (after a successful payment retry), the UI updates automatically. The poll count is tracked in sessionStorage to prevent re-triggering on page refresh.

The poller checks every 10 seconds for up to 9 attempts (90 seconds total). If the subscription becomes active during this window, the UI updates automatically. The polling state is tracked in `sessionStorage` so it is not re-triggered across page refreshes.

---

## Related Data Fetching

The subscription settings page fetches several data sets on mount:

| Data              | Endpoint                        | Cached In              |
| ----------------- | ------------------------------- | ---------------------- |
| Subscription      | `/subscriptions/details`        | auth.subscription      |
| Pricing           | `/pricing`                      | auth.pricing           |
| Payment Methods   | `/payment-methods`              | auth.payment_methods   |
| ASIN Selections   | `/asin-selections`              | Local component state |

Each fetch is conditional: if the data already exists in Redux, it is not re-fetched.

---

## Interview Talking Points

**On the subscription normalization:** "The backend returns subscription data in different shapes depending on the endpoint and subscription status. I normalize the response into a consistent format that the UI components can rely on. Fields like `billing_interval` and `status` are mapped from their various possible source field names so the display logic stays simple."

**On the polling mechanism:** "Subscriptions in `past_due` status may become active again after a payment retry. Rather than making the user manually refresh, I implemented a polling loop that checks every 10 seconds for up to 90 seconds. If the status changes to active, the UI updates automatically. The poll count is tracked in sessionStorage so a page refresh does not restart the timer."

## Related Documents

- [Settings Overview](overview.md) - Module structure
- [Account Settings](account-settings.md) - PPC automation from accounts tab
- [Payment and Invoices](payment-and-invoices.md) - Payment methods and retry
- [External Integrations](../integrations/external-integrations.md) - Stripe payment processing
- [External Integrations](../integrations/external-integrations.md) - Stripe integration
