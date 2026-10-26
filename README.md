# Iru (formerly Kandji)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
