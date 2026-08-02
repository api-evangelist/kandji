---
name: Renew an Automated Device Enrollment token before it expires
description: Audit and renew Apple Automated Device Enrollment (ADE) tokens in an Iru (formerly Kandji) tenant so zero-touch enrollment does not break when an Apple Business/School Manager token lapses.
api: openapi/kandji-endpoint-management-openapi.json
provider: Iru (formerly Kandji)
operations:
  - listADEIntegrations
  - getADEIntegration
  - downloadADEPublicKey
  - renewADEIntegration
  - listDevicesAssociatedToADEToken
generated: '2026-08-01'
method: generated
---

# Renew an Automated Device Enrollment token before it expires

Writes tenant configuration. An expired ADE token silently breaks zero-touch enrollment for every
device attached to it, so this is a high-value scheduled audit — but the renewal itself involves a
round trip through Apple Business Manager / Apple School Manager that a human must perform.

## Steps

1. **List the integrations.** `GET /api/v1/integrations/apple/ade` (`listADEIntegrations`).
2. **Inspect each one.** `GET /api/v1/integrations/apple/ade/{ade_token_id}` (`getADEIntegration`)
   for its expiry and Apple account. Flag anything expiring inside your window.
3. **Size the blast radius.**
   `GET /api/v1/integrations/apple/ade/{ade_token_id}/devices` (`listDevicesAssociatedToADEToken`)
   — how many devices depend on this token. Report this number to the human before renewing.
4. **Download the public key.**
   `GET /api/v1/integrations/apple/ade/public_key/` (`downloadADEPublicKey`). This is the key the
   human uploads to Apple Business/School Manager to generate a fresh server token.
5. **Human step, outside this API.** The administrator uploads the public key in Apple
   Business/School Manager and downloads the new `.p7m` server token.
6. **Renew.** `POST /api/v1/integrations/apple/ade/{ade_token_id}/renew` (`renewADEIntegration`)
   with the new token.
7. **Verify.** Re-run step 2 and confirm the new expiry. Re-run step 3 and confirm the device count
   is unchanged.

## Related operations

- `POST /api/v1/integrations/apple/ade/` (`createADEIntegration`) — add a new ADE integration.
- `PATCH /api/v1/integrations/apple/ade/{ade_token_id}` (`updateADEIntegration`).
- `DELETE /api/v1/integrations/apple/ade/{ade_token_id}` (`deleteADEIntegration`) — **destructive**:
  detaches every device in step 3 from zero-touch enrollment. Requires explicit human approval.
- `GET /api/v1/integrations/apple/ade/devices` (`listADEDevices`),
  `GET /api/v1/integrations/apple/ade/devices/{device_id}` (`getADEDevice`),
  `PATCH /api/v1/integrations/apple/ade/devices/{device_id}` (`updateADEDevice`) — assign an ADE
  device to a Blueprint before it enrolls.

## Rules

- Never `DELETE` an ADE integration as a way to "reset" it. Renew instead.
- The renew call has no idempotency key. If it times out, re-read step 2 to see whether the new
  expiry landed before re-posting.
- The trailing slash on `POST /api/v1/integrations/apple/ade/` and
  `GET /api/v1/integrations/apple/ade/public_key/` is significant — the spec declares those paths
  with the slash and the list path without it.
- Errors are `{"error": "<message>"}` with only 400/401/404 declared. See `../errors/`.
