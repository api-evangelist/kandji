# Iru (formerly Kandji)

Iru, Inc. — San Francisco — is an AI-powered IT and security platform unifying workforce identity
and access, cross-platform endpoint management (macOS, iOS, iPadOS, tvOS, visionOS, Windows,
Android), endpoint detection and response, vulnerability management, compliance automation and a
customer-facing trust center behind a single agent.

- Website: https://www.iru.com/
- API reference: https://api-docs.kandji.io/
- Docs: https://docs.iru.com/
- Status: https://status.kandji.io
- GitHub: https://github.com/kandji-inc
- Secondary-market listing: https://forgeglobal.com/kandji_stock/

## Name note — Kandji vs Iru

The harvest backlog listed this company as **Iru** under the slug **kandji**. Both are correct at
different points in time:

- **Iru** is the authoritative current company and brand name. Kandji rebranded to Iru on
  **22 October 2025**, and the live site, the Platform Services Agreement and the legal entity all
  read "Iru, Inc.".
- **Kandji** survives everywhere the plumbing lives: the API hostnames `{subdomain}.api.kandji.io` /
  `{subdomain}.api.eu.kandji.io`, the API reference at `api-docs.kandji.io`, the status page at
  `status.kandji.io`, the support site at `support.kandji.io`, the `kandji-inc` GitHub organization,
  the `io.kandji.*` macOS unified-logging subsystems, and even the MCP connector path
  (`/mcp-server/connector/kandji/tools`). The docs say so themselves: "Kandji is now Iru, but many
  URLs and notes within this documentation will continue to reference Kandji for some time."
- `kandji.com` does **not** redirect to `iru.com` (HTTP 405 to a HEAD probe on 2026-08-01).
- The Forge Global listing is still filed under the Kandji name and returned HTTP 403 to automated
  fetches.

The slug stays `kandji` for catalog continuity; `name:` in `apis.yml` is `Iru`, with
`x-former-name: Kandji`.

## APIs

| API | Surface |
|---|---|
| Iru Endpoint Management API | REST, OpenAPI 3.0.0, 94 paths / 121 operations, tenant-scoped bearer token |
| Iru Library Item Upload API | Single pre-signed S3 upload contract, its own OpenAPI document |
| Iru MCP Server | First-party hosted MCP, tenant-scoped URL, `X-API-Key` + `X-MCP-Profile` |

## Artifacts in this repo

`openapi/` `collections/` `overlays/` `mcp/` `skills/` `agentic-access/` `authentication/`
`conventions/` `errors/` `data-model/` `lifecycle/` `changelog/` `rate-limits/` `conformance/`
`packages/` `cli/` `llms/` `well-known/` `security/`

Notable finds:

- The OpenAPI is real and complete but was **not** at any conventional location — it is linked only
  from the `## OpenAPI Specs` section at the bottom of `https://docs.iru.com/llms.txt`
  (`https://docs.iru.com/openapi/iru-endpoint-openapi.json`). `api-docs.kandji.io` is a Postman
  documenter, and every `/openapi.json`-style probe on the API and docs hosts 404s.
- The spec declares **no `operationId` and no `tags`** on any of its 121 operations. Both are added
  as a non-destructive OpenAPI Overlay in `overlays/`, tagged using the provider's own Postman
  folder grouping.
- There is **no idempotency contract** on an API whose write surface includes `erase`, `lock` and
  `shutdown`. See `conventions/` and the safety rules in `skills/`.
- Errors are a one-field `{"error": "..."}` envelope — not RFC 9457 — and no `403`, `429` or `5xx`
  is declared anywhere, despite a documented 10,000 req/hour tenant limit.
- No `/.well-known/` document of any kind, and no A2A agent card.
