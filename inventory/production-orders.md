# Production Orders

The production orders module manages the manufacturing lifecycle. It tracks purchase orders from suppliers, incoming inventory, and costs associated with production.

**Why this matters for portfolios:** Production order management demonstrates handling real-world supply chain workflows. Interviewers in e-commerce or logistics roles will want to hear about tracking supplier relationships, inbound inventory, and cost accrual across the procurement lifecycle.

---

## What You Will Learn

- How production orders are created and tracked
- How products are added to production orders
- How costs and payments are managed
- How shipment attachments work

---

## Production Order Lifecycle

```
Create PO → Add Products → Add Costs → Place Order → Receive Shipment → Payment → Close Order
```

### 1. Create Purchase Order

The user creates a new production order:
- **Supplier** — Select from contacts or enter manually
- **Order date** — When the order was placed
- **Expected delivery** — Estimated arrival date
- **Order name** — Auto-generated based on supplier and date

### 2. Add Products

Products are added to the order through a modal:
- Search and select products from the catalog
- Enter quantity ordered
- Enter unit cost
- Review total line cost

### 3. Add Costs

Additional costs can be added (separate from product costs):
- Shipping/freight costs
- Customs/duties
- Inspection fees
- Other miscellaneous costs

### 4. Receive Shipment

When the order arrives, the user records receipt:
- Confirm quantities received
- Note any damages or shortages
- Update expected versus actual delivery date

### 5. Payment

Record payment details:
- Payment method
- Amount paid
- Payment date
- Invoice or receipt attachment

---

## Production Order Name Auto-Generation

The system auto-generates production order names based on supplier name and date. The format is standardized (e.g., PO-SUPPLIERNAME-DATE) to make orders easy to find in lists and searches. Special characters are sanitized to prevent issues in URLs or file names.

---

## Production Order Table

The production orders list shows:
- Order name
- Supplier
- Status (draft, placed, in transit, received, closed)
- Total cost
- Expected delivery date
- Number of products

---

## Attaching Shipments

Production orders can be linked to FBA shipments. When products arrive from a supplier, they can be forwarded directly to Amazon FBA through the attached shipment.

The attach shipment flow lets users:
- Select an existing shipment
- Create a new shipment from the production order products
- Assign quantities to the shipment

---

## Production Order Details

A detail view shows the full order information:
- **Header** — Supplier, dates, status
- **Products table** — SKU, product name, quantity ordered/received, unit cost, line total
- **Costs table** — Cost type, amount, notes
- **Payments table** — Payment method, amount, date
- **Attached shipments** — Links to related FBA shipments
- **Notes** — Free-form notes about the order

---

## Interview Talking Points

**On the auto-naming convention:** "Production order names are auto-generated for consistency. The standardized format makes orders easy to find. Sanitizing the supplier name prevents special characters from breaking URLs or file names."

**On the attach shipment workflow:** "Linking production orders to shipments was a requested feature. When inventory arrives from a supplier, sellers often want to send it straight to Amazon FBA. The attach shipment flow connects these two workflows so users do not have to re-enter product information."

---

## Related Documents

- [Inventory Module Overview](overview.md) - Module architecture
- [FBA Shipments](fba-shipments.md) - Shipment creation
- [3PL and AWD](3pl-and-awd.md) - Logistics integration
- [Demand Forecasting](futuristic-analytics.md) - Demand forecasting
