# 3PL and AWD Integration

The 3PL (Third-Party Logistics) and AWD (Amazon Warehousing and Distribution) module manages inventory stored outside of Amazon's fulfillment centers. It tracks stock at multiple warehouse locations and handles transfers between them.

---

## What You Will Learn

- How 3PL warehouses are managed
- How stock is tracked across multiple locations
- How shipments from 3PL to FBA work
- How Amazon AWD is integrated

---

## 3PL Warehouse Management

Each 3PL warehouse is a separate entity with its own stock levels, addresses, and contact information.

**Warehouse details:**
- Name and address
- Contact information
- Stock levels for each product
- Inbound and outbound shipment tracking

### 3PL Stock Table

Shows stock levels for each product at each warehouse:
- Product ASIN and name
- Current stock level
- Reserved units
- Available units
- Inbound units
- Last updated timestamp

---

## 3PL Shipments

Shipments between 3PL warehouses and FBA are managed through a separate flow:

1. **Create shipment** — Select source warehouse, destination, products, quantities
2. **Add tracking** — Enter carrier and tracking information
3. **Confirm receipt** — Mark shipment as received at destination

### Shipment Types

| Type | From | To |
| ----------- | --------------- | --------------- |
| 3PL to FBA | 3PL Warehouse | Amazon FBA |
| FBA to 3PL | Amazon FBA | 3PL Warehouse |
| 3PL to 3PL | 3PL Warehouse | Another 3PL |

---

## Amazon AWD

Amazon Warehousing and Distribution is Amazon's upstream storage service. The integration supports:
- **Stock monitoring** — View inventory stored in AWD
- **Auto-replenishment tracking** — See when AWD is sending stock to FBA
- **Transfer history** — Track movements from AWD to FBA

---

## Routing

The 3PL module uses nested routes so each 3PL warehouse has its own URL path while sharing the same layout and navigation context.

---

## Stock Modals

The 3PL module includes modal interfaces for managing stock:
- Create a new 3PL warehouse record
- Add stock to a 3PL warehouse
- Edit warehouse details
- View product details in 3PL context

Each modal handles validation, API calls, and state updates independently.

---

## Package Information

When creating shipments from a 3PL, the package information form collects:
- Package dimensions (length, width, height)
- Package weight
- Number of units per package
- Total number of packages
- Carrier selection

---

## Related Documents

- [Inventory Module Overview](overview.md) - Module architecture
- [FBA Shipments](fba-shipments.md) - FBA shipment creation
- [Inventory Health](inventory-health.md) - Stock monitoring
- [Production Orders](production-orders.md) - Manufacturing lifecycle
