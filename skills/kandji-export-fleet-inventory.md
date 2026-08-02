---
name: Export fleet inventory from Prism
description: Answer fleet-wide inventory and compliance questions in an Iru (formerly Kandji) tenant using the Prism category endpoints, and pull a full export with the request-then-poll export job.
api: openapi/kandji-endpoint-management-openapi.json
provider: Iru (formerly Kandji)
operations:
  - count
  - deviceInformation
  - applications
  - filevault
  - requestCategoryExport
  - getCategoryExport
generated: '2026-08-01'
method: generated
---

# Export fleet inventory from Prism

Read-only. Prism is the correct surface for any question that spans more than a handful of devices —
it answers per category across the whole fleet in one call, instead of N calls to
`/api/v1/devices/{device_id}/...`.

## Steps

1. **Size the answer first.** `GET /api/v1/prism/count` (`count`) before paging anything.
2. **Query the category** that matches the question. Each is a `GET /api/v1/prism/<category>` and
   supports `limit`/`offset` (and `cursor` on the paged surfaces), plus `blueprint_ids`,
   `device_families` and `device_id` facets:
   - `device_information` (`deviceInformation`) — hardware, OS, enrollment
   - `apps` (`applications`) — installed software across the fleet
   - `filevault` (`filevault`) — disk encryption state
   - `activation_lock` (`activationLock`)
   - `application_firewall` (`applicationFirewall`)
   - `certificates` (`certificates`)
   - `desktop_and_screensaver` (`desktopAndScreensaver`)
   - `gatekeeper_and_xprotect` (`gatekeeperAndXProtect`)
   - `installed_profiles` (`installedProfiles`)
   - `kernel_extensions` (`kernelExtensions`)
   - `launch_agents_and_daemons` (`launchAgentsAndDaemons`)
   - `local_users` (`localUsers`)
   - `startup_settings` (`startupSettings`)
   - `system_extensions` (`systemExtensions`)
   - `transparency_database` (`transparencyDatabase`)
   - `cellular` (`cellular`)
3. **For a full dump, use the export job, not pagination.**
   `POST /api/v1/prism/export` (`requestCategoryExport`) starts an asynchronous export, then poll
   `GET /api/v1/prism/export/{export_id}` (`getCategoryExport`) until it is ready.
4. **Pace the poll.** There is no documented poll interval and no `Retry-After`; the tenant budget is
   10,000 requests/hour shared with every other token and the MCP server. Poll on a backoff, not a
   tight loop.

## Choosing between Prism and the device endpoints

| Question | Use |
|---|---|
| "Is *this* Mac encrypted?" | `GET /api/v1/devices/{device_id}/details` |
| "How many Macs are unencrypted?" | `GET /api/v1/prism/filevault` |
| "What is installed on *this* device?" | `GET /api/v1/devices/{device_id}/apps` |
| "Who has version X installed?" | `GET /api/v1/prism/apps` |

## Rules

- Export ids are tenant-scoped; a `404` on `export_id` means the job is not in this tenant.
- Prism categories are read-only — there is no Prism write operation.
- Errors are `{"error": "<message>"}` with only 400/401/404 declared. See `../errors/`.
