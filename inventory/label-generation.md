# Label Generation

The label generation module creates barcodes and shipping labels for products, boxes, and shipments. It is used during the FBA shipment process and for general product labeling.

---

## What You Will Learn

- How barcodes are generated in the browser
- How the label creation page works
- How labels integrate with the shipment workflow
- How labels are printed

---

## How Barcodes Are Generated

Barcodes are generated directly in the browser using a client-side barcode generation library. No server-side image generation is needed.

**Supported barcode formats:**
- Code 128 (default)
- EAN-13
- UPC-A

The barcode value is typically the product ASIN, SKU, or a shipment identifier.

---

## Label Creation Page

The Create Labels page provides a standalone label generator:

1. **Select label type** — Product label, box label, or shipment label
2. **Enter identifier** — ASIN, SKU, or shipment ID
3. **Configure label** — Size, format, quantity
4. **Generate** — Barcodes are rendered in the browser
5. **Print** — Labels are sent to the printer

---

## Integration with Shipments

During the FBA shipment creation wizard, labels are generated as part of the packing step:
- Each box in the shipment gets a unique box label
- Each product in the box can have individual product labels
- Labels include the FBA shipment ID and destination warehouse

---

## Interview Talking Points

**On client-side barcode generation:** "Generating barcodes in the browser avoids server load, works offline, and gives instant feedback. The library generates SVG barcodes that scale cleanly for printing."

**On printing from web apps:** "Printing labels from a web app is harder than it sounds. Labels need to be formatted to standard sizes so they work with thermal printers."

---

## Related Documents

- [Inventory Module Overview](overview.md) - Module architecture
- [FBA Shipments](fba-shipments.md) - Shipment creation wizard
