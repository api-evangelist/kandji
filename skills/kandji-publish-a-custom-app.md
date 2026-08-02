---
name: Publish a Custom App and assign it to a Blueprint
description: Upload a package to an Iru (formerly Kandji) tenant as a Custom App Library Item and assign it to a Blueprint, using the two-step pre-signed S3 upload contract.
api: openapi/kandji-endpoint-management-openapi.json
provider: Iru (formerly Kandji)
operations:
  - uploadCustomApp
  - createCustomApp
  - listBlueprints
  - assignLibraryItem
  - getLibraryItemStatuses
generated: '2026-08-01'
method: generated
---

# Publish a Custom App and assign it to a Blueprint

Writes tenant configuration and, once assigned, causes software to install on managed devices.
Confirm the target Blueprint with a human before assigning.

## The upload is two contracts, not one

Binary upload does **not** go to `api.kandji.io`. `POST /api/v1/library/custom-apps/upload`
(`uploadCustomApp`) returns a pre-signed S3 URL, and the upload itself is described by a **separate
OpenAPI document** the provider publishes — `openapi/kandji-upload-to-s3-openapi.json`, whose
server is literally "the S3 upload URL provided by the custom app creation endpoint. This is a
variable URL that changes for each upload request."

## Steps

1. **Request the upload slot.** `POST /api/v1/library/custom-apps/upload` (`uploadCustomApp`).
2. **PUT the package** to the pre-signed URL returned in step 1, per
   `openapi/kandji-upload-to-s3-openapi.json`. Do not send the Iru bearer token to S3.
3. **Create the Library Item.** `POST /api/v1/library/custom-apps` (`createCustomApp`), referencing
   the uploaded file.
4. **Choose the Blueprint.** `GET /api/v1/blueprints` (`listBlueprints`); confirm the target with a
   human. `GET /api/v1/blueprints/templates/` (`getBlueprintTemplates`) if you need a new one, and
   `POST /api/v1/blueprints` (`createBlueprint`) to create it.
5. **Assign.** `POST /api/v1/blueprints/{blueprint_id}/assign-library-item` (`assignLibraryItem`).
6. **Verify the assignment.** `GET /api/v1/blueprints/{blueprint_id}/list-library-items`
   (`listLibraryItems`).
7. **Watch rollout.** `GET /api/v1/library/library-items/{library_item_id}/status`
   (`getLibraryItemStatuses`) and
   `GET /api/v1/library/library-items/{library_item_id}/activity` (`getLibraryItemActivity`).

## The iOS/iPadOS variant

In-house IPA apps use a parallel surface with an explicit upload-status poll:
`POST /api/v1/library/ipa-apps/upload` (`uploadInHouseApp`) →
`GET /api/v1/library/ipa-apps/upload/{pending_upload_id}/status` (`uploadInHouseAppStatus`) →
`POST /api/v1/library/ipa-apps` (`createInHouseApp`). Poll the status endpoint until the upload
completes before creating the Library Item.

## Other Library Item types

Custom Profiles (`/api/v1/library/custom-profiles`) and Custom Scripts
(`/api/v1/library/custom-scripts`) follow the same list/create/get/update/delete shape but have no
binary upload step.

## Rules

- Updates are `PATCH`, never `PUT` — the API uses no `PUT` anywhere.
- No idempotency key exists. If step 3 or step 5 times out, list first
  (`GET /api/v1/library/custom-apps`, `listCustomApps`) to check whether the object was created
  before retrying, or you will create a duplicate Library Item.
- Assigning a Custom App to a Blueprint pushes software to every device in that Blueprint. Confirm
  the Blueprint's device count via `GET /api/v1/devices?blueprint_id=...` before assigning.
- Errors are `{"error": "<message>"}` with 400/401/404 only. See `../errors/kandji-problem-types.yml`.
