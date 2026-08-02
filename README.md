# OneRail

OneRail is a last-mile delivery orchestration company. Its OmniPoint platform rate-shops and dispatches
orders across a network of 1,000+ connected carriers and 12M+ drivers, optimizes multi-stop routes for
shipper-owned fleets, and streams delivery events back for end-to-end visibility and proof of delivery.

- Website: https://www.onerail.com/
- Developer Hub: https://developer.onerail.io/hc/en-us
- Status: https://onerail.cronitorstatus.com
- Trust Center: https://trust.onerail.com/

## APIs

| API | Spec | Operations | Base host |
|---|---|---|---|
| OneRail Delivery API | OpenAPI 3.0.1 (v0.0.3) | 31 | `onerail-delivery-api-prod.azurewebsites.net` |
| OneRail Operations API | OpenAPI 3.0.1 (v1.0.0) | 438 | `onerail-operations-prod.azurewebsites.net` |

Both specs were harvested live from the Swagger UI bootstrap (`/api-docs/swagger-ui-init.js`) on the
production hosts — OneRail publishes no standalone `/openapi.json` or `/swagger.json`.

Authentication is an organization App ID / API Key header pair (`X-ONERAIL-APP-ID` + `X-ONERAIL-API-KEY`)
minted per shipper by OneRail; there is no self-serve signup.

## Artifacts

`openapi/` `overlays/` `authentication/` `scopes/` `conventions/` `errors/` `lifecycle/` `data-model/`
`conformance/` `sandbox/` `asyncapi/` (webhook event catalog) `mcp/` (candidate tools + crosswalk)
`skills/` `agentic-access/` `security/` `well-known/` `llms/`

Not published by OneRail: llms.txt, any `/.well-known/` document, an A2A agent card, an MCP server,
GraphQL, first-party SDKs, a CLI, a Postman workspace, a GitHub organization, a public changelog,
rate limits, or a vulnerability-disclosure program.
