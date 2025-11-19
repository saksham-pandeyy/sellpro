# Demand Forecasting

The demand forecasting module predicts future inventory events based on current stock levels, sales velocity, and lead times. It uses a calendar view to show projected stockouts, recommended production orders, and suggested shipment dates.

---

## What You Will Learn

- How demand forecasting works
- How the calendar view organizes events by type
- How manual demand adjustments (spikes) work
- How the report generation process works
- How event types are color-coded for quick scanning

---

## The Core Idea

Demand forecasting answers: "What is going to happen with my inventory, and what should I do about it?"

The system takes current stock levels, applies historical sales velocity, factors in lead times, and projects forward to identify upcoming events.

---

## Event Types

The system generates several types of events:

| Event Type | Description | Color |
| ----------------------- | ---------------------------------------------------------- | ----------- |
| Projected Stockout Risk | Stock is expected to run out on this date | Red |
| Create Production Order | Recommended date to place a production order | Orange |
| Create Shipment | Recommended date to create an FBA shipment | Blue |
| Shipment Other | General shipment-related events | Light Blue |
| Production Other | General production-related events | Light Orange |
| Send From 3PL | Recommended date to transfer from 3PL to FBA | Green |
| Transfer/3PL | General 3PL transfer events | Light Green |
| Other | Miscellaneous events | Gray |

---

## Calendar View

The calendar displays events in monthly, weekly, or daily views with:
- Color-coded events by type
- Click on an event for details
- Navigation between months
- "All events" view showing the full list as a table

### All Events Report

A special "All Events" report generates events for all products at once. This report is generated asynchronously because it can take time to process. When the report is ready, the user receives a notification with a link to view it.

---

## Spikes (Manual Adjustments)

Spikes are manual adjustments to the forecast. Users can add spikes for:
- **Seasonal increases** — Expected demand spike during holidays
- **Promotional events** — Known promotions that will increase sales
- **Supply disruptions** — Known supply chain issues

Each spike has: product or product group, date range, impact percentage, and description. The system incorporates spikes into the forecast calculations.

---

## Configurable Parameters

The module has configurable parameters:
- **Lead time** — Days from order to delivery (per product or default)
- **Safety stock** — Minimum stock to maintain (per product or default)
- **Sales lookback period** — How many days of sales history to use
- **Default days of supply target** — Target stock level in days

Changing a parameter recalculates all events.

---

## Data Sources

Demand forecasting uses data from:
1. Current stock levels
2. Sales velocity (30-day average by default)
3. Lead times from product settings
4. Safety stock from product settings
5. Manual spike adjustments
6. Expected inbound stock from production orders
7. In-transit stock from FBA shipments

---

## Event Color Coding

Each event type has a specific color scheme. Red highlights critical events (stockouts), orange shows recommended actions (production orders), and blue indicates shipment-related events. The color scheme makes it easy to scan the calendar and identify priorities.

---

## Report Generation Flow

The all-events report is generated asynchronously:
1. User requests the report
2. Backend starts generating
3. Frontend polls for completion every 30 seconds, up to 5 attempts
4. When complete, a notification appears with a view button
5. Click navigates to the calendar view with data

---

## Interview Talking Points

**On the forecasting approach:** "The forecasting is rule-based rather than ML-based. It takes current stock, applies historical sales velocity, factors in lead times, and projects forward. This is simpler and more transparent than a machine learning model. Users can see exactly why an event was generated and adjust the inputs."

**On the async report generation:** "Generating events for all products can take 30 seconds or more. We process this asynchronously so the user is not blocked. The polling service checks for completion and shows a notification when ready."

---

## Related Documents

- [Inventory Module Overview](overview.md) - Module architecture
- [Inventory Health](inventory-health.md) - Current stock levels
- [Production Orders](production-orders.md) - Manufacturing lifecycle
- [FBA Shipments](fba-shipments.md) - Shipment creation
- [Report Polling](../notifications/report-polling.md) - Async report generation
