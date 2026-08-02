---
name: Rate shop a delivery and dispatch it through the OneRail network
description: Price a prospective delivery across every qualified carrier in the OneRail network, create the order from the chosen rate, then finalize it for dispatch.
api: openapi/onerail-delivery-api-openapi.yml
generated: '2026-08-02'
method: generated
source: https://developer.onerail.io/hc/en-us/articles/50726438250267-API-Use-Case-Partial-Update-and-Get-Rates
operations:
  - checkDeliveryAvailability
  - getDeliveryRates
  - create
  - patchPartialUpdate
  - getDetails
---

# Rate shop and dispatch

Base URL: `https://onerail-delivery-api-prod.azurewebsites.net` (staging and UAT hosts exist — see
`sandbox/onerail-sandbox.yml`). Send both credential headers on **every** request:

```
X-ONERAIL-APP-ID: <app id issued to this shipper>
X-ONERAIL-API-KEY: <api key issued to this shipper>
```

Credentials are minted per shipper organization by OneRail; there is no self-serve issuance.

## Preconditions

The pickup location must already exist on the organization. If it does not, create it first with
`POST /v1/location` on the Operations API (`openapi/onerail-operations-api-openapi.yml`).

## Steps

1. **Optional — confirm the lane is servable.** `checkDeliveryAvailability`
   (`POST /delivery/check-availability`) with `pickUpData` and `dropOffData`. A minimal request returns
   `availability` plus `dataCompleteness: MINIMAL`. Add `items`/`containers` and `serviceLevels` to get
   per-service-level `estimatedPrice`. Do not treat a MINIMAL response as a quote.

2. **Rate shop.** `getDeliveryRates` (`POST /delivery/get-rates`) with the full delivery payload —
   `pickUpData`, `dropOffData`, `orderData.items[]`, `orderData.containers[]`, `deliveryType`
   (`BUSINESS` or `RESIDENTIAL`). Only carriers that respond in time are returned; a missing carrier is a
   timeout, not a refusal. Set `saveDraftDelivery: false` to price without creating anything, or `true`
   to have OneRail persist a draft.

3. **Create the order.** `create` (`POST /order/create`) or `createV2` (`POST /v2/order`) when you also
   need return legs. Required: `deliveryType`, `orderData.orderId`, and `address1`/`city`/`state`/`zip`
   on both `pickUpData` and `dropOffData`. Set `state: DRAFT` while details are still moving.
   `orderId` is yours — it is echoed on every response and every webhook event, so make it your
   correlation key.

4. **Fill in the decision.** `patchPartialUpdate` (`PATCH /order/{orderId}/partial-update`) appends items
   and containers; `putPartialUpdate` (`PUT /order/{orderId}/partial-update`) replaces them. Use this to
   attach the selected carrier, final costs, and `requestLabel: true` when the label is needed now.

5. **Finalize.** Transition `state` to `READY_TO_DISPATCH`. If the order carries no valid service level,
   OneRail will not dispatch and emits the `NO_MATCHING_SLA_SHIPPER_CONTRACTS` webhook event instead of
   failing the HTTP call — watch for it.

6. **Read back.** `getDetails` (`GET /delivery/{id}/details`).

## Rules

- **Money is integer cents** (`netAmount`, `priceCent`, `contractedShipperCostCent`). Timestamps are ISO 8601.
- **There is no `Idempotency-Key` header on this API.** Retrying `create` is not replay-safe. Check state
  with `getDetails` before retrying a create that timed out. See `conventions/onerail-conventions.yml`.
- **Errors** are `application/json` — `{code, message, errors, errorDetails[{detail, pointer}]}`, not RFC 9457.
  `errorDetails[].pointer` is an RFC 6901 JSON Pointer at the offending request field; read it before retrying.
  Validation failures may instead return `{message: "Validation errors", errors: [{message, location}]}`.
- Hazmat items need `hazardousMaterialDetails`; freight items need `nmfc` and `freightClass`.
