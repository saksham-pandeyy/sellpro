# Payment and Invoices

The Payment methods and Invoices tabs manage billing data. Users can view saved credit cards, set a primary payment method, browse invoice history, and download individual receipts.

---

## What You Will Learn

- How payment methods are listed and managed
- How a primary payment method is set
- How new payment methods are added via Stripe
- How invoices are listed with sorting and pagination
- How individual invoice receipts are viewed and downloaded

---

## Payment Methods

The Payment Methods tab (`/settings/payment-methods`) displays saved credit cards fetched from the backend.

### Card List

Each card is displayed with:

- Card icon
- Masked card number (`XXXX-XXXX-XXXX-1234`)
- Expiration date
- "Primary" label (if default)
- Delete button

### Set as Primary

Users can mark any card as the primary payment method:

Setting a primary card updates the local state optimistically (marking the selected card and unmarking others) while the backend request completes. If the API call fails, the optimistic update is reverted.

The primary card is used as the default for subscription payments and addon purchases. The update is optimistic: the local card list is updated immediately before the API response.

### Delete Card

A delete button opens a confirmation modal. On confirm, a `DELETE /api/v1/payment-methods/{id}` request removes the card from the list.

### Add Payment Method

Navigating to `/settings/payment-methods/add` opens the `PaymentAddSettingsPage`, which integrates Stripe Elements:

The Payment Add page integrates Elements from the payment gateway's SDK. The card input is rendered in a secure iframe so the platform never touches raw card numbers, staying out of PCI compliance scope.

The user enters card details in Stripe's secure iframe. The addon handles card validation and saves the new payment method to the user's account.

---

## Invoices

The Invoices tab (`/settings/invoices`) displays all past invoices in a sortable, paginated table using the `CustomTable` component.

### Invoice Table

| Column         | Description                    |
| -------------- | ------------------------------ |
| Invoice ID     | Unique invoice identifier      |
| Date           | Invoice date                   |
| Payment Method | Card or other method used      |
| Net Amount     | Amount before adjustments      |
| Discount       | Any discount applied           |
| Total Amount   | Final charged amount           |
| Status         | Paid, failed, or void          |
| Actions        | View and download buttons      |

Statuses are color-coded:
- **Paid** (green)
- **Failed** (red)
- **Void** (yellow)

The table supports sorting by any column and pagination with configurable page sizes. Invoice data is cached in the `auth` Redux slice under `invoices`.

### Actions

Each invoice row has two action buttons:

**View** - Opens the invoice receipt page (`/settings/invoces/{stripeId}`). Clicking fetches the invoice details from the backend and navigates to the receipt view. If the invoice was already fetched, the cached data is used.

**Download** - Downloads the invoice as a PDF:

Invoice PDFs are fetched as binary blobs using `responseType: 'blob'`. A temporary object URL is created and a programmatic click triggers the download with the correct filename. This approach avoids opening the PDF in a new tab.

The download uses `responseType: 'blob'` to handle binary PDF data, creates a temporary download link, and programmatically clicks it.

---

## Invoice Receipt Page

The individual receipt page (`/settings/invoces/{stripeId}`) shows a formatted invoice:

- **Header**: Platform logo and "RECEIPT" title
- **Bill To**: Customer name, email, address
- **Invoice Info**: Invoice number and date
- **Line Items**: Description, payment method, and price
- **Totals**: Price, discount, subtotal, sales tax, and total amount

The receipt includes a download button for PDF export. The layout is designed for printing with a clean, structured format.

### Receipt Loading

The receipt page first checks if the invoice data is already cached in Redux (`state.invoice.invoiceDetails`). If not, it fetches from the backend:

The receipt page first checks the Redux cache for the invoice data. If available, it renders immediately without an API call. Otherwise, it fetches from the backend and caches the result for subsequent visits.

If the invoice is not found, the user is redirected back to the invoices list with a warning alert.

---

## Interview Talking Points

**On the Stripe integration for adding cards:** "Adding a payment method uses Stripe Elements, which renders the card input in a secure iframe. The platform never touches the raw card number. Stripe returns a tokenized payment method that we store as a reference. This keeps us out of PCI compliance scope."

**On the download-as-blob pattern:** "Invoice PDFs are downloaded as binary blobs. The API returns raw PDF data, and we create a temporary URL using `URL.createObjectURL`. This avoids opening the PDF in a new tab and then having the user manually save it. The download happens directly on click with the correct filename."

**On invoice caching:** "Invoice details are cached in a Redux slice keyed by Stripe ID. When the user navigates from the invoice list to the receipt, we check the cache first. If the data is already loaded, we skip the API call. This makes navigating back and forth between invoices instant."

## Related Documents

- [Settings Overview](overview.md) - Module structure
- [Subscription Settings](subscription-settings.md) - Plan and addon management
- [External Integrations](../integrations/external-integrations.md) - Stripe integration
- [Custom Table System](../frontend/custom-table-system.md) - Table component used for invoice list
