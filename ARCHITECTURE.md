# Heliolytics App — Architecture

Flutter-only repo. Server and web are separate projects.

## Platform (3 repos)

| Repo | Responsibility |
|------|----------------|
| **Heliolytics_App** (this repo) | BLE fetch, upload raw sessions, metrics UI |
| **Heliolytics** | Go ingest, parse, PostgreSQL, Docker deploy |
| **Heliolytics_Web** | Next.js dashboard |

## Data flow

```
Strap ──BLE──► Flutter (raw .bin + catalog JSON)
                    │ POST /api/v1/ingest
                    ▼
               Go API (Heliolytics repo)
                    │
                    ▼
               PostgreSQL
               ╱         ╲
         Next.js web    Flutter GET /metrics
```

## This repo

| Path | Role |
|------|------|
| `lib/core/ble/` | Protocol, fetch, paging anchors |
| `lib/features/cloud_sync/` | Upload to Go API |
| `lib/features/health_data/` | Metrics UI (API-backed) |
| `lib/features/ble_discovery/` | Scan, auth key, sessions |

Parsing for display happens on the **Go server** after ingest. The app keeps minimal on-device parsers for BLE paging only.

## Deploy

- **Mobile:** Play Store APK/AAB from this repo
- **API + DB + web:** `Heliolytics/deploy/install.sh` (sibling repo)
