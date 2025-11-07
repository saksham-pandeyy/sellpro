# Inventory Module

The inventory module is the operational backbone of the platform. It manages product stock across multiple warehouses, tracks shipments between locations, handles production orders from suppliers, and forecasts future demand.

---

## Why This Module Matters for Portfolio

Inventory management involves real engineering challenges:
- **Data synchronization** with external APIs that have rate limits and pagination
- **Multi-location stock tracking** across FBA warehouses, 3PL facilities, and incoming shipments
- **Demand forecasting** combining historical sales, seasonality, and lead times
- **Status management** for dozens of shipment, order, and product states
- **Real-time sync indicators** so users know if their data is current

---

## Module Structure

The inventory module is organized by sub-domain:

```
inventory/
  3pl/                          # Third-party logistics
  futurists/                    # Demand forecasting
  production-orders/            # Manufacturing management
  products/                     # Product management
  sales/                        # Sales analytics
  shipments/                    # FBA shipment management
```

---

## Key Features Summary

| Feature | Description | Key Pages |
|---------|-------------|----------------|
| Inventory Health | Real-time stock levels, days of supply, inbound quantities | Inventory List |
| Product Management | ASIN catalog, images, sales history, cost settings | Product List, Single Product |
| Sales Analytics | Unit sales, revenue, comparisons, gauges | Sales Page |
| FBA Shipments | Multi-step creation wizard with label generation | Shipments, Add Shipment |
| 3PL/AWD | Third-party warehouse and Amazon AWD integration | 3PL Page |
| Production Orders | Manufacturing lifecycle from purchase order to receipt | Production Orders |
| Demand Forecasting | Calendar-based demand forecasting | Futuristic Data |
| Label Creation | Barcode generation for products and shipments | Create Labels |

---

## Data Flow Overview

Data flows from external APIs through background sync workers into the database, and then into the frontend state:

```
External APIs → Background Sync → Database → Frontend State → UI Components
```

Sync runs in steps, each pulling a different data type. When a sync step is running, the sidebar shows an animated icon next to the relevant menu item.

**Sync steps and their routes:**

| Sync Step | Menu Item | Data |
|-----------|-----------|------|
| inventory_list | Inventory | Stock levels, quantities, conditions |
| product_list | Products | ASINs, categories, attributes |
| sales | Sales | Sales history, units, revenue |
| shipment_list | Shipments | Inbound/outbound shipments |
| three_pl_awd | 3PL | Third-party warehouse stock |
| reports | P&L Report | Financial data |
| orders | Reviews | Customer orders |

The sync status is checked via an API endpoint and displayed in the sidebar. Users can see at a glance whether their data is up to date.

---

## State Management

The inventory module uses multiple state slices, each managing a different domain:
- Inventory list and stock levels
- Product catalog and details
- Sales statistics and comparisons
- Shipment list and tracking
- Order review data
- Demand forecasting data
- 3PL warehouse data

Slices are separated by domain to prevent data collisions and allow independent fetching.

---

## Data Prefetching

When the dashboard loads, common datasets (inventory, products, shipments, sales) are prefetched in the background. It only fetches data that has not been loaded yet. By the time the user navigates to the Inventory page, the data is often already available.

---

## Related Documents

- [Inventory Health](inventory-health.md) - Stock monitoring and alerts
- [Product Management](product-management.md) - Product catalog and details
- [FBA Shipments](fba-shipments.md) - Shipment creation wizard
- [3PL and AWD](3pl-and-awd.md) - Third-party logistics integration
- [Production Orders](production-orders.md) - Manufacturing lifecycle
- [Demand Forecasting](futuristic-analytics.md) - Demand forecasting
- [Label Generation](label-generation.md) - Barcode and label creator
