# Alert Categories

Alerts are grouped into categories based on the source system. Each category has its own set of alert types, severity levels, and display components.

---

## What You Will Learn

- How shipment alerts are categorized
- How inventory alerts are categorized
- How production order alerts are categorized
- How severity levels work
- How alert display components are organized

---

## Shipment Alerts

Monitors shipments for delays, missing information, and delivery deadlines.

| Alert Type                | Severity | Description                                               | Action                           |
| ------------------------- | -------- | --------------------------------------------------------- | -------------------------------- |
| Shipment Past ETA         | Critical | Shipment has passed its expected delivery date            | Update tracking or contact carrier |
| Missing Tracking ID       | Warning  | Shipment is in transit but no tracking number provided    | Enter tracking information       |
| Delivery Window Deadline  | Warning  | Shipment delivery window is approaching                   | Prepare to receive inventory     |
| Missing Shipment Fields   | Warning  | Required shipment fields are incomplete                   | Complete shipment details        |

---

## Inventory Alerts

Monitors product stock levels, pricing, and health.

| Alert Type                   | Severity | Description                                        | Action                            |
| ---------------------------- | -------- | -------------------------------------------------- | --------------------------------- |
| Low Stock                    | Warning  | Stock below 28 days of supply                      | Create replenishment order        |
| Out of Stock                 | Critical | Stock is at zero                                   | Create immediate replenishment    |
| Excess Inventory             | Info     | Stock exceeds 90 days of supply                    | Consider promotion or removal     |
| Price Change                 | Info     | Product price has changed significantly            | Review pricing strategy           |
| Competitor Price             | Info     | Competitor price change detected                   | Adjust pricing                    |
| Sale Ending in 3 Days        | Warning  | Promotional sale price is ending soon              | Plan next pricing strategy        |
| Fulfillment Price Adjustment | Info     | FBA fulfillment fee has changed                    | Review profitability              |
| High Return Rate (Month)     | Warning  | Return rate exceeded threshold this month          | Investigate product quality       |
| High Return Rate (Year)      | Warning  | Return rate exceeded threshold this year           | Review long-term trends           |
| Negative Feedback Detected   | Critical | Customer left negative feedback                    | Respond to customer               |

---

## Production Order Alerts

Monitors production order timelines.

| Alert Type                   | Severity | Description                                          | Action                           |
| ---------------------------- | -------- | ---------------------------------------------------- | -------------------------------- |
| Production Order ETA Arrived | Info     | Production order expected delivery date has arrived  | Confirm receipt or update ETA    |

---

## Alert Category Configuration

The alert categories are defined as a configuration object that maps alert types to their category:

```javascript
const alertCategories = [
    {
        name: 'Shipments',
        types: [
            'three_pl_shipment_past_eta',
            'three_pl_shipment_missing_fields',
            'shipment_missing_tracking_id',
            'delivery_window_deadline',
        ]
    },
    {
        name: 'Inventory',
        types: [
            'product_stock_close_to_28days',
            'product_sale_price_end_in_3_days',
            'negative_feedback_detected',
            'fulfillment_price_adjustment',
            'excess_inventory_warning',
            'high_return_rate_detected_in_month',
            'high_return_rate_detected_in_year',
        ]
    },
    {
        name: 'Production Orders',
        types: [
            'production_order_eta_date_arrived',
        ]
    },
];
```

---

## Alert Display Components

Each category has dedicated display components:

- **ShipmentAlerts** - Renders shipment-related alerts
- **InventoryAlerts** - Renders inventory-related alerts
- **ProductionOrderAlerts** - Renders production order alerts
- **AlertCategories** - Organizes alerts by category in the UI

---

## Interview Talking Points

**On separating alerts by category:** "Each alert category has its own monitor and display component. Shipment alerts check different conditions than inventory alerts. This separation means a change to inventory alert logic does not risk breaking shipment alerts. Each component is independently testable."

## Related Documents

- [Alert Architecture](alert-architecture.md) - System overview
- [Alert Workflow](alert-workflow.md) - How alerts are processed
- [Tips System](tips-system.md) - Optimization tips
- [Inventory Health](../inventory/inventory-health.md) - Inventory monitoring
