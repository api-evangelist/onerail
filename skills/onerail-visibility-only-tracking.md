---
name: Track a shipment fulfilled outside the OneRail network
description: Create a visibility-only delivery so OneRail tracks a truckload or shipment that was booked and fulfilled by an external carrier, and feed its events back for shipper-facing transparency.
api: openapi/onerail-delivery-api-openapi.yml
generated: '2026-08-02'
method: generated
source: https://developer.onerail.io/hc/en-us/articles/50726327516571-API-Use-Case-Visibility
operations:
  - createVisibility
  - addVisibility
  - getDetails
---

# Visibility-only tracking

Use this when the shipment is **not** being rate-shopped, routed, or dispatched by OneRail. Visibility
deliveries do not engage the OneRail carrier network; they exist so a shipper keeps one tracking surface
across carriers it books itself.

## Steps

1. **Create the visibility record.** `createVisibility` (`POST /visibility/create`).
   Minimum: `orderId` (unique identifier for the customer order) and the carrier's **SCAC** (Standard
   Carrier Alpha Code) — SCAC is required so OneRail can attribute the events the logistics partner
   reports. Include `shipmentId` as well when you need truckload-level visibility above the order level.

2. **Attach the individual deliveries.** `addVisibility`
   (`POST /visibility/{orderId}/add-deliveries`) for each delivery on the truckload.

3. **Feed events back.** The assigned logistics partner posts to `POST /visibility/{orderId}/event`:
   `PICKED_UP` (loaded at origin), `EN_ROUTE_TO_DELIVERY` (in transit), `DELIVERED`, and `GEO` for
   real-time GPS position where the partner supports it.
   Body shape: `{"eventType": "PICKED_UP", "orderId": "MasterBOL123", "eventOn": "2025-02-01T08:00:00Z"}`.

4. **Read back.** `getDetails` (`GET /delivery/{id}/details`).

## Rules

- **`shipmentId` is the truckload designator; `orderId` is one customer order on that truckload.** Getting
  this backwards silos the tracking.
- The logistics partner that will report events is chosen and registered **at creation time** and cannot
  be inferred later.
- Visibility orders are never rate-shopped or matched to carriers — do not call `getDeliveryRates` for them.
- OneRail provides visibility tracking but **not routing** for these shipments.
