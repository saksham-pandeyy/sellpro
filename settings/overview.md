# Settings Module

The settings module provides a centralized interface for users to manage their account, subscription, payment methods, and application preferences. It is organized as a sidebar-navigated multi-tab page within the authenticated dashboard.

---

## What You Will Learn

- How the settings page is structured with a sidebar and nested routes
- What each settings tab covers
- How the sidebar navigation works on mobile vs. desktop
- How settings changes persist and take effect

---

## Settings Navigation

The settings page uses a sidebar layout with nested routing. Each tab is a separate route under `/settings/`:

```
/settings/general         Profile, password, theme, zoom, 2FA
/settings/shipments       Shipping method defaults
/settings/accounts        Amazon account connections and API authorization
/settings/subscription    Subscription plan, PPC toggle, ASIN capacity
/settings/payment-methods Saved credit cards and payment defaults
/settings/invoices        Invoice history and receipt downloads
```

### Sidebar Structure

The `SettingsSidebar` component renders the navigation menu with SVG icons for each tab. On desktop, the sidebar is always visible. On mobile, it is hidden by default and toggled via a hamburger menu button (`SettingMenu` component) that uses a `SettingMenuContext` to share state.

### Routing Setup

The settings routes are configured in `App.js`:

```javascript
<Route element={<SettingMenuProvider><SettingsPage /></SettingMenuProvider>} path="/settings/">
    <Route element={<GeneralSettings />} path="general" />
    <Route element={<ShipmentSettings />} path="shipments" />
    <Route element={<AccountSettings />} path="accounts" />
    <Route element={<SubscriptionSettings />} path="subscription" />
    <Route element={<PaymentSettings />} path="payment-methods" />
    <Route element={<InvoicesPage />} path="invoices" />
    <Route element={<InvoiceReceipt />} path="invoces/:invoiceId" />
    <Route element={<PaymentAddSettingsPage />} path="payment-methods/add" />
</Route>
```

The `SettingMenuProvider` wraps the entire settings section and provides the mobile menu toggle state through React context. The `SettingsPage` component renders the sidebar and an `<Outlet />` for the active tab content.

---

## Tab Overview

| Tab            | Route                          | Purpose                                                   |
| -------------- | ------------------------------ | --------------------------------------------------------- |
| General        | `/settings/general`             | Profile name, password, 2FA toggle, theme, UI zoom        |
| Shipments      | `/settings/shipments`           | Default shipping method configurations                    |
| Accounts       | `/settings/accounts`            | Amazon account connections, SP-API and Ads API auth       |
| Subscription   | `/settings/subscription`        | Plan details, PPC addon toggle, ASIN capacity, cancel     |
| Payment        | `/settings/payment-methods`     | Saved credit cards, set primary, add/delete cards         |
| Invoices       | `/settings/invoices`            | Invoice history, PDF download, individual receipt view    |

---

## Interview Talking Points

**On the sidebar routing pattern:** "The settings page uses nested React Router routes with a shared sidebar layout. The `SettingsPage` component renders the sidebar and an `<Outlet />` for the active tab. This means each tab is a self-contained component that gets mounted only when navigated to. The `SettingMenuProvider` context handles the mobile menu toggle state so clicking a menu item on mobile automatically closes the sidebar."

**On data-driven settings:** "Several settings tabs fetch data from the backend when they mount. The Accounts tab loads Amazon accounts and pricing data. The Subscription tab fetches subscription details, pricing, payment methods, and ASIN selections. Each tab manages its own loading state and caches data in Redux so navigating between tabs does not re-fetch unnecessarily."

## Related Documents

- [General Settings](general-settings.md) - Profile, theme, zoom, 2FA
- [Account Settings](account-settings.md) - Amazon accounts and API authorization
- [Subscription Settings](subscription-settings.md) - Plan management and addons
- [Payment and Invoices](payment-and-invoices.md) - Payment methods and invoice history
- [Frontend Architecture](../architecture/frontend-architecture.md) - Routing and state management
