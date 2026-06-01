# Samsung Frame TV Art Changer (Home Assistant Add-on)

This add-on manages Samsung Frame TV Art via a Home Assistant Ingress web app.

## Core functionality

- Gallery with art from Home Assistant (`/media/frame`) and from the TV.
- Sync-status per item (`HA`, `TV`, `SYNC`, `ACTIVE`).
- 2-way visibility in the gallery:
  - HA items become visible and can be activated to TV.
  - TV items become visible, including background thumbnail fetch via queue.
- Upload with interactive crop to fixed output `3840x2160` (JPEG quality 90).
- Crop tools:
  - Zoom
  - Rotation slider (`-45..45`)
  - Extra `90 graden` rotation
  - Flip horizontally
- Actions per item:
  - Activate on selected TVs
  - Remove from `tv`, `ha` of `both`
- Non-blocking refresh met metadata (`stale`, `refresh_in_progress`, `last_refresh`).
- Caching:
  - TV snapshot cache in memory (TTL)
  - Thumbnail cache op disk (`/data/cache/thumbs`)
- Settings page in the UI:
  - Manage TV IPs
  - Refresh interval
  - Snapshot TTL
  - Network scan for supported Samsung TVs

## Add-on Configuration

In `config.yaml` from the add-on:

- `tv`: comma-separated IP list (initial TV list).
- `automation_token`: token voor REST fallback endpoint.
- `ingress: true`: UI via Home Assistant Ingress.
- `stdin: true`: native automation trigger via `hassio.addon_stdin`.

Example add-on options:

```yaml
tv: "192.168.10.170"
automation_token: "kies-een-lang-geheim-token"
```

## UI Usage

Open in Home Assistant:
`Instellingen -> Add-ons -> Samsung Frame TV Art Changer -> Open Web UI`.

Important:

- The sidebar option (`In zijbalk tonen`) is a Home Assistant UI option on the add-on page.
- The add-on does not need to be configured separately.

## Automation (Native, Recommended)

Use Home Assistant service `hassio.addon_stdin`.

Example: choose a random item, upload it if necessary, and activate it.

```yaml
action:
  - service: hassio.addon_stdin
    data:
      addon: 88e5264e_hass-frametv-artchanger
      input: >-
        {"action":"random_activate","tv_ips":["192.168.10.170"],"ensure_upload":true,"activate":true}
```

Example: trigger manual refresh.

```yaml
action:
  - service: hassio.addon_stdin
    data:
      addon: 88e5264e_hass-frametv-artchanger
      input: '{"action":"refresh"}'
```

Important:

- Use at `addon:` always your real add-on ID from Home Assistant.
- This is often the case for custom repositories `<repo_hash>_hass-frametv-artchanger` (like `88e5264e_hass-frametv-artchanger`).
- You can find this ID on the add-on page or via Developer Tools in the service call.

Supported stdin actions:

- `{"action":"random_activate","tv_ips":[...],"ensure_upload":true,"activate":true}`
- `{"action":"refresh"}`

## Automation (REST Fallback)

Only use as a fallback when `hassio.addon_stdin` doesn't fit into your flow.

Endpoint:

- `POST /api/automation/random`
- Auth: `Authorization: Bearer <automation_token>`

Example `rest_command` in Home Assistant:

```yaml
rest_command:
  frame_random_art:
    url: "http://<jouw-addon-hostname>:8099/api/automation/random"
    method: POST
    headers:
      Authorization: "Bearer YOUR_AUTOMATION_TOKEN"
      Content-Type: "application/json"
    payload: >-
      {
        "tv_ips": ["192.168.10.170"],
        "ensure_upload": true,
        "activate": true
      }
```

## API Overview

- `GET /api/health`
- `GET /api/tvs`
- `GET /api/settings`
- `PUT /api/settings`
- `POST /api/settings/discover`
- `GET /api/gallery`
- `POST /api/refresh`
- `GET /api/thumb/{asset_id}`
- `POST /api/upload`
- `POST /api/items/{asset_id}/activate`
- `DELETE /api/items/{asset_id}`
- `POST /api/automation/random`

## API Meta & Errors

Responses include (where applicable):

- `meta.stale`
- `meta.refresh_in_progress`
- `meta.last_refresh`
- `meta.request_id`

Errors are backward-compatible:

- `detail` continues to exist
- extra `error.code`, `error.message`, `error.retryable`, `error.request_id`

Examples of error codes:

- `INVALID_INPUT`
- `UNAUTHORIZED`
- `NOT_FOUND`
- `TV_OFFLINE`
- `TV_UNSUPPORTED`
- `UPLOAD_FAILED`
- `DELETE_FAILED`
- `NO_RANDOM_ASSETS`
- `INTERNAL_ERROR`

## Migration

- Existing `/media/frame` images are indexed.
- Legacy `uploaded_files.json` is automatically migrated.

## Standalone Local Launch (Development/Test)

For local testing from your own system on the same network:

```powershell
cd c:\HomeAssistant\homeassistant-addons\homeassistant-samsung-frametv-artchanger
.\run-local.ps1 -TvIps "192.168.10.170"
```

`run-local.ps1` makes `standalone-media` in `standalone-data` automatically if they do not exist yet.

Then open:

- `http://localhost:8099`
- or from another device in your LAN: `http://<jouw-pc-ip>:8099`

## Troubleshooting

- No items visible:
  - Check TV IP and whether the TV is online.
  - Trigger manually `Refresh`.
  - Check add-on logs.
- Thumbnails stay up `Bezig...`:
  - TV thumbnail fetching runs via queue (one by one); give it some time.
