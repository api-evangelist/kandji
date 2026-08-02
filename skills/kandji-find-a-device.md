---
name: Find a device and read its posture
description: Locate an enrolled device in an Iru (formerly Kandji) tenant by serial number, name or user, then read its inventory, status, installed apps and Blueprint Parameter results.
api: openapi/kandji-endpoint-management-openapi.json
provider: Iru (formerly Kandji)
operations:
  - listDevices
  - getDevice
  - getDeviceDetails
  - getDeviceStatus
  - getDeviceApps
  - getDeviceParameters
generated: '2026-08-01'
method: generated
---

# Find a device and read its posture

Read-only. Nothing in this skill changes state on a device.

## Before you start

- Base URL is tenant-specific: `https://{subdomain}.api.kandji.io` (US) or
  `https://{subdomain}.api.eu.kandji.io` (EU). The value is shown in the Iru Web App under
  **Access → API tokens** as "Your organization's API URL". Never guess it.
- Send `Authorization: Bearer <token>` on every request. See `../authentication/kandji-authentication.yml`.
- The token must have the matching device permissions enabled in its permission grid, or the call fails.
- Budget: 10,000 requests/hour for the whole tenant, shared with every other token and with the MCP
  server. See `../rate-limits/kandji-rate-limits.yml`. List broadly, then narrow — do not iterate
  device-by-device across a fleet.

## Steps

1. **Search the fleet.** `GET /api/v1/devices` (`listDevices`). Narrow with the query facets the
   spec declares: `serial_number`, `device_name`, `asset_tag`, `mac_address`, `model`, `os`,
   `user_id`, `blueprint_id`, `filevault_enabled`. Page with `limit` and `offset`.
2. **Take the `device_id`** from the match. Ids are tenant-scoped — an id from another tenant
   returns 404.
3. **Read the record.** `GET /api/v1/devices/{device_id}` (`getDevice`) for the summary.
4. **Read full inventory.** `GET /api/v1/devices/{device_id}/details` (`getDeviceDetails`) for the
   expanded hardware/OS/security record.
5. **Read management status.** `GET /api/v1/devices/{device_id}/status` (`getDeviceStatus`).
6. **Read installed software.** `GET /api/v1/devices/{device_id}/apps` (`getDeviceApps`).
7. **Read policy results.** `GET /api/v1/devices/{device_id}/parameters` (`getDeviceParameters`) for
   Blueprint Parameter pass/fail state.

## Fleet-wide instead of per-device

If the question is about many devices rather than one, do **not** loop step 3–7. Use the Prism
reporting endpoints, which answer per-category across the fleet in one call — `GET /api/v1/prism/device_information`,
`/api/v1/prism/apps`, `/api/v1/prism/filevault`, `/api/v1/prism/certificates`,
`/api/v1/prism/installed_profiles`. See `kandji-export-fleet-inventory.md`.

## Rules

- Error bodies are `{"error": "<message>"}` — not RFC 9457, and there is no machine-readable error
  code. Branch on the HTTP status only. See `../errors/kandji-problem-types.yml`.
- `401` means the token is missing, malformed, revoked, from another tenant, **or possibly** lacks
  the endpoint permission — the spec declares no `403`, so do not infer which.
- `404` on a device id means "not in this tenant" — re-resolve it from `listDevices`, do not retry.
- No `429` is declared but the hourly limit is real. Back off on any unexpected status.
- Do **not** call the device secrets endpoints (`/secrets/filevaultkey`, `/secrets/bypasscode`,
  `/secrets/recoverypassword`, `/secrets/unlockpin`) as part of a posture read. They return
  credential material and should only be fetched for a specific, human-authorized recovery task.
