---
name: Build and optimize a multi-stop route for an internal fleet
description: Sequence orders onto OneRail routes for a shipper-owned fleet, choosing between OneRail's optimizer and a caller-supplied stop order.
api: openapi/onerail-delivery-api-openapi.yml
generated: '2026-08-02'
method: generated
source: https://developer.onerail.io/hc/en-us/articles/50726388637083-API-Use-Case-Routing
operations:
  - createRoutes
  - create
  - getDetails
---

# Route an internal fleet

## Preconditions (Operations API — `openapi/onerail-operations-api-openapi.yml`)

Internal-fleet routing only behaves correctly once the account is configured:

- **Markets** — the geographic zones the fleet operates in, with service hours and cut-off times.
- **Fleet assets** — every vehicle with capacity, weight limits and type, each linked to its market.
- **SLAs** — pickup and drop-off commitments with buffer times and end-of-day deadlines.

## Steps

1. **Create the orders** with `create` (`POST /order/create`), including pickup/drop-off locations,
   delivery windows and service levels.

2. **Create the route.** `createRoutes` (`POST /v1/routes`) with a `routes[]` array; each route has an
   `id`, an `orders[]` array, and `skipOptimizer`:
   - `skipOptimizer: true` — OneRail honours your sequence exactly. Use for time-sensitive or
     priority-ordered runs.
   - `skipOptimizer: false` — OneRail reorders stops to minimize travel time and distance while
     respecting constraints. Use for high-volume or complex routes.

3. **Read back** with `getDetails` per delivery to confirm assignment.

## Constraints the optimizer enforces — plan for rejection

- **Market hours are the hardest constraint.** Total drive time plus loading/unloading must finish inside
  the market window; the engine maximizes route length up to that boundary.
- **Vehicle capacity** — item/container length, width, height, weight and quantity must fit the vehicle's
  volume and weight maximums.
- **Appointment windows** — `opensOn`/`closesOn` per stop are strict. **Deliveries whose windows cannot be
  met may simply fail to be assigned to a route.** Always verify what actually landed on the route rather
  than assuming every submitted order was accepted.
- Multi-leg runs that return to the pickup to refill are supported by defining the pickup as both start
  and midpoint; capacity resets on reload.
