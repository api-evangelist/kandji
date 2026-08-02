---
name: Run a device action safely
description: Dispatch an MDM device action (lock, erase, restart, shutdown, clear passcode, lost mode) in an Iru (formerly Kandji) tenant with the confirmation and verification steps the API's lack of an idempotency contract demands.
api: openapi/kandji-endpoint-management-openapi.json
provider: Iru (formerly Kandji)
operations:
  - listDevices
  - lockDevice
  - eraseDevice
  - restartDevice
  - shutdown
  - clearPasscode
  - enableLostMode
  - disableLostMode
  - getDeviceCommands
  - getDeviceActivity
generated: '2026-08-01'
method: generated
---

# Run a device action safely

**This skill takes destructive, irreversible actions on physical hardware in someone's hands.**
Iru's own MCP guidance states that for destructive operations (erase, delete, lock) the assistant
"should summarize the impact and require your explicit approval before executing, unless you've
added that action to an allowlist." Follow that literally.

## The critical constraint

The Iru Endpoint Management API documents **no idempotency key and no retry contract**
(`../conventions/kandji-conventions.yml`). A `POST /api/v1/devices/{device_id}/action/erase` that
times out has **no defined de-duplication semantics**. Treat every action as **at-most-once**:
dispatch, then verify by polling — never retry blind.

## Steps

1. **Resolve the target precisely.** `GET /api/v1/devices` (`listDevices`) filtered on
   `serial_number` (preferred) or `asset_tag`. Confirm exactly one match. If more than one, stop and
   ask.
2. **Restate the impact to the human** before any write: device name, serial, assigned user,
   Blueprint, and what the action does. For `erase` say plainly that all data on the device is
   destroyed and the device is unenrolled from management.
3. **Get explicit approval.** Do not proceed on an ambiguous "yes do it" that predates the restated
   impact.
4. **Dispatch exactly one action.** All are `POST` with the `device_id` in the path:
   - `POST /api/v1/devices/{device_id}/action/lock` (`lockDevice`)
   - `POST /api/v1/devices/{device_id}/action/erase` (`eraseDevice`) — **irreversible**
   - `POST /api/v1/devices/{device_id}/action/restart` (`restartDevice`)
   - `POST /api/v1/devices/{device_id}/action/shutdown` (`shutdown`)
   - `POST /api/v1/devices/{device_id}/action/clearpasscode` (`clearPasscode`)
   - `POST /api/v1/devices/{device_id}/action/enablelostmode` (`enableLostMode`)
   - `POST /api/v1/devices/{device_id}/action/disablelostmode` (`disableLostMode`)
   Lower-consequence actions in the same family: `blankpush`, `dailycheckin`, `updateinventory`,
   `updatelocation`, `renewmdmprofile`, `reinstallagent`, `setname`, `remotedesktop`,
   `unlockaccount`, `deleteuser`, `playlostmodesound`.
5. **Verify — do not retry.** Poll `GET /api/v1/devices/{device_id}/commands` (`getDeviceCommands`)
   for the dispatched MDM command and its state, and
   `GET /api/v1/devices/{device_id}/activity` (`getDeviceActivity`) for the audit trail. A device
   that is offline will show the command queued; that is expected, not a failure.
6. **On a timeout or 5xx**, poll step 5 first. Only re-dispatch if the command is genuinely absent
   from the command list, and re-confirm with the human before doing so.

## Lost mode

`enableLostMode` / `disableLostMode` are dispatch actions. The state is read at
`GET /api/v1/devices/{device_id}/details/lostmode` (`getDeviceLostModeDetails`) and cleared with
`DELETE /api/v1/devices/{device_id}/details/lostmode` (`cancelLostMode`).
`POST .../action/playlostmodesound` (`playLostModeSound`) sounds the device.

## Rules

- Never batch destructive actions across a device list without per-device human confirmation.
- Never call an action to "test" connectivity — use `dailycheckin` or `blankpush`, which are benign.
- `401` / `404` / `400` bodies are `{"error": "<message>"}`; branch on status. See `../errors/`.
- The tenant hourly budget (10,000 req/hr) is shared with the MCP server — polling loops in step 5
  should be paced, not tight.
