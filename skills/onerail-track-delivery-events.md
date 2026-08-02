---
name: Receive and interpret OneRail delivery events
description: Register a webhook endpoint on the shipper organization and correctly handle the OneRail delivery event stream from carrier acceptance through proof of delivery.
api: openapi/onerail-operations-api-openapi.yml
generated: '2026-08-02'
method: generated
source: https://developer.onerail.io/hc/en-us/articles/50726344844315-Delivery-Events-and-Webhooks
operations:
  - updateWebhookUrl
  - getAllDeliveryNotificationStatuses
  - updateDeliveryNotificationStatus
  - retryPendingDeliveryEvents
  - getDetails
---

# Track delivery events

**Nothing is sent until an endpoint is registered.** OneRail states explicitly that a webhook URL must be
provided and configured *before* events will be delivered.

## Steps

1. **Register the endpoint.** `updateWebhookUrl`
   (`PATCH /v1/organization/{organizationId}/webhook-url` on the Operations API). Send `null` to remove it.

2. **Handle the envelope.** Every event carries `deliveryId` (OneRail's id), `orderId` (yours),
   `eventType`, `eventOn` (ISO 8601), and usually `vin`, `from.storeNumber`,
   `shipper.contractedShipperCostCent` and `shipper.computedDistanceMile`. Key your state machine on
   `orderId`; use `deliveryId` for OneRail-side lookups.

3. **Handle the happy path in order** — but do not assume ordering is guaranteed; use `eventOn`:
   `ACCEPTED_BY_LP` → `DRIVER_ASSIGNED` (adds `driverName`, `driverPhone`) → `EN_ROUTE_TO_PICKUP` →
   `ARRIVED_FOR_PICKUP` → `PICKED_UP` → `EN_ROUTE_TO_DELIVERY` → `ARRIVED_FOR_DELIVERY` → `DELIVERED`
   (adds `podUrl` and `signature` image links).

4. **Handle the terminal exceptions.** `CANCELED_BY_ONERAIL`, `CANCELED_BY_SHIPPER`, and
   `NO_MATCHING_SLA_SHIPPER_CONTRACTS` — the last means the order was created without a valid service
   level and must be corrected before it can dispatch.

5. **Reconcile.** `getAllDeliveryNotificationStatuses` (`GET /v1/delivery/notification-statuses`) shows
   the webhook attempt state OneRail recorded for each delivery; `retryPendingDeliveryEvents` retries
   pending ones. Fall back to `getDetails` on the Delivery API for authoritative delivery state.

## Rules

- **OneRail publishes no webhook signing scheme, shared secret, or replay protection.** Treat the payload
  as unauthenticated: use a hard-to-guess endpoint path, verify the referenced `deliveryId`/`orderId`
  against your own records, and never take a state transition on the payload alone for high-consequence
  actions — confirm with `getDetails`.
- Events are at-least-once in practice (OneRail retries failed attempts), so make your handler idempotent
  on `(deliveryId, eventType, eventOn)`.
- Custom event types are available per organization on request, so treat unknown `eventType` values as
  non-fatal.
