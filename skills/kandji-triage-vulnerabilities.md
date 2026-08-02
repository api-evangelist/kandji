---
name: Triage a CVE across the fleet
description: Use the Iru (formerly Kandji) Vulnerability Management API to find which devices and which software builds are exposed to a CVE, and cross-reference EDR threat detections.
api: openapi/kandji-endpoint-management-openapi.json
provider: Iru (formerly Kandji)
operations:
  - listVulnerabilities
  - getVulnerabilityDescription
  - listAffectedDevices
  - listAffectedSoftware
  - listDetections
generated: '2026-08-01'
method: generated
---

# Triage a CVE across the fleet

Read-only. Safe to run without human confirmation.

## Steps

1. **Find the CVE.** `GET /api/v1/vulnerability-management/vulnerabilities` (`listVulnerabilities`).
   Page with `limit`/`offset`; narrow with the declared query facets before paging deeply.
2. **Read the description.**
   `GET /api/v1/vulnerability-management/vulnerabilities/{cve_id}` (`getVulnerabilityDescription`).
3. **Get exposed devices.**
   `GET /api/v1/vulnerability-management/vulnerabilities/{cve_id}/devices` (`listAffectedDevices`).
4. **Get exposed software builds.**
   `GET /api/v1/vulnerability-management/vulnerabilities/{cve_id}/software` (`listAffectedSoftware`)
   — this is what tells you which version to patch to.
5. **Check open detections.**
   `GET /api/v1/vulnerability-management/detections` (`listDetections`) for the current detection
   state across the fleet.
6. **Cross-reference EDR.** `GET /api/v1/threat-details` (`getThreatDetails`) and
   `GET /api/v1/behavioral-detections` (`getBehavioralDetections`) for active malware/behavioural
   findings on the same devices. These are separate surfaces from vulnerability management — a
   device can be vulnerable without a detection, and vice versa.
7. **Enrich each device.** Feed the `device_id` values from step 3 into
   `GET /api/v1/devices/{device_id}` (`getDevice`) and
   `GET /api/v1/devices/{device_id}/apps` (`getDeviceApps`) to confirm the installed build, or use
   the fleet-wide `GET /api/v1/prism/apps` (`applications`) in a single call instead of looping.

## Remediating

This API has no "patch this CVE" operation. Remediation is delivered through the Library:
publish or update the fixed app version as a Custom App / Auto App and assign it to the affected
devices' Blueprint — see `kandji-publish-a-custom-app.md`. Tag the affected devices first
(`POST /api/v1/tags`, `createTag`) if you need a durable cohort.

## Rules

- Do not loop step 7 device-by-device across a large fleet — the tenant budget is 10,000
  requests/hour shared with every token and the MCP server. Use Prism for fleet-wide reads.
- `404` on a `cve_id` means it is not tracked in this tenant, not that the CVE does not exist.
- Errors are `{"error": "<message>"}`; only 400/401/404 are declared. See `../errors/`.
