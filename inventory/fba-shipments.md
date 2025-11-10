# FBA Shipments

The FBA (Fulfillment by Amazon) shipment module manages the process of sending inventory to Amazon's warehouses. It is a multi-step wizard that guides users through product selection, packing, labeling, carrier selection, and confirmation.

---

## What You Will Learn

- How the multi-step shipment creation wizard works with state management
- How products are selected and packed into boxes with validation
- How shipping methods are determined based on package dimensions
- How tracking is managed and validated
- How errors at each step are caught and communicated

---

## Shipment Lifecycle

```
Draft → Submitted → In Transit → Delivered → Closed
                                  |             |
                                  v             v
                             Cancelled     Cancelled
```

Each status transition triggers specific API calls and UI updates:

| Status | User Action Required |
| ----------- | ------------------------------ |
| Draft | Add products, pack boxes |
| Submitted | Confirm shipment plan |
| In Transit | Enter carrier tracking info |
| Delivered | Wait for Amazon to process |
| Closed | Review completed shipment |
| Cancelled | Confirm cancellation |

---

## Shipment List Page

The shipment list shows all inbound shipments with their current status in a searchable, sortable table:

- Shipment ID (with link to detail view)
- Status (color-coded badge)
- Destination warehouse
- Number of units
- Estimated delivery window
- Created date
- Carrier name (if assigned)

**Filters:** Status filter (multi-select), Date range filter, Search by shipment ID or carrier

**Bulk actions:** Select multiple shipments with checkboxes, Cancel selected (draft only)

---

## Shipment Creation Wizard

Creating a new FBA shipment is a multi-step process with client-side state persistence. The wizard state is managed locally (not in the global state store) because it is only relevant during creation.

### Step 1: Choose Address and Products

The user selects:
1. **Shipping address** — Origin address (can be saved for future shipments)
2. **Inventory source** — Which warehouse the stock is coming from
3. **Products to ship** — Select products and quantities

Products are searchable by ASIN, SKU, or name. Quantity cannot exceed available stock. At least one product must be selected to proceed.

### Step 2: Packing Details

The user defines how products are packed into cases. This is the most complex step with nested validation:

**Validation checks:**
- Total units across all cases must match total units selected in Step 1
- Box dimensions must be within limits
- Weight must be within carrier limits
- Each case must have at least one product

**Auto-pack feature:** An "Auto Pack" option distributes products into cases based on typical case sizes. Users can override any case manually.

### Step 3: Shipping Confirmation

The final step shows a summary and lets the user confirm. Shipping method options:

| Method | Description | Typical Transit |
| ------------------------- | ------------------------------------------------ | --------------- |
| Small Parcel | Individual boxes via standard carriers | 3-5 days |
| LTL (Less Than Truckload) | Palletized freight for larger shipments | 5-7 days |
| FTL (Full Truckload) | Full truck for very large shipments | 3-5 days |
| Partnered Carrier | Discounted rates via partner carriers | 2-5 days |

---

## External API Integration

### Shipment Submission

When the user confirms, the frontend calls the backend, which interacts with the external API:

1. Backend creates shipment plan with external API
2. Extract shipment ID from response
3. Create shipment record in database with status 'submitted'
4. Return shipment details to frontend

### Tracking Update

Users can enter tracking information after the shipment is in transit. The tracking number format is validated based on the carrier before submission.

---

## Error Handling

| Error Scenario | How It Is Handled |
| -------------------------------- | --------------------------------------------------------------- |
| API rate limit exceeded | Backend retries with exponential backoff. User sees progress indicator |
| Invalid box dimensions | Client-side validation prevents submission. Error shown inline |
| Product out of stock | Quantity selector prevents selecting more than available |
| API validation error | Backend returns structured error. Frontend shows notification |
| Network failure during submit | Retry button appears. Wizard state is preserved |
| Partial creation failure | Backend rolls back resources. Returns error |

---

## Interview Talking Points

**On the wizard design:** "FBA shipments require sequential decisions. You cannot pack boxes before you know what you are shipping. You cannot choose a shipping method before you know box dimensions. The wizard enforces this order and validates each step before the user can proceed."

**On the auto-pack feature:** "The auto-pack uses a heuristic that packs same products together and limits each box to a typical weight. It is not perfect, but it saves users from manually distributing hundreds of units across dozens of boxes. Users can always adjust individual cases afterward."

**On external API integration complexity:** "The inbound shipment API is one of the most complex to work with. It requires creating a plan, confirming it, printing labels, then providing tracking. Each step has its own validation rules. The wizard abstracts this complexity behind a guided interface."

---

## Related Documents

- [Inventory Module Overview](overview.md) - Module architecture
- [3PL and AWD](3pl-and-awd.md) - Third-party logistics integration
- [Production Orders](production-orders.md) - Manufacturing lifecycle
- [Label Generation](label-generation.md) - Barcode and label creation
