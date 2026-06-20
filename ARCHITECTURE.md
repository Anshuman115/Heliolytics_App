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

Parsing for display happens on the **Go server** after ingest. The app keeps minimal on-device parsers for BLE paging only.

## Layer-first layout

```
lib/
  main.dart, app.dart
  router/                 GoRouter + auth redirect
  screens/                Full pages (ConsumerWidget)
  widgets/                Composed UI (charts, home sections, …)
  providers/              Riverpod Notifiers + provider defs
  services/
    ble/                  Protocol, sync pipeline, auth, parsers
    network/              Dio + HMAC token helper
    config/               API URL/key secure storage
    metrics_api_client.dart
    cloud_sync_repository*.dart
    session_store.dart
  models/                 Domain / JSON entities
  utils/                  Logger, formatters, time, chart helpers
  constants/              Sync windows, build markers, literals
  design_system/          Tokens, theme, shared components
```

### Layer rules

| Layer | Responsibility | Must not |
|-------|----------------|----------|
| **screens** | Layout, navigation (`context.push`) | Call BLE or Dio directly |
| **widgets** | Dumb / lightly composed UI | Own business rules |
| **providers** | State, orchestration, `ref.watch` services | Import Flutter widgets (except types) |
| **services** | BLE, HTTP, storage, repositories | Depend on UI |
| **models** | Plain data + JSON | Side effects |

## Key modules

| Path | Role |
|------|------|
| `providers/sync_orchestrator.dart` | BLE session state machine + auto-connect |
| `providers/live_health_provider.dart` | Cloud metrics snapshot (GET APIs) |
| `services/ble/sync_orchestrator_*.dart` | Fetch, upload, incremental window logic |
| `services/metrics_api_client.dart` | Days, series, sleep, workouts, coverage |
| `services/cloud_sync_repository_impl.dart` | Multipart ingest upload |
| `router/app_router.dart` | Routes, auth guard |

## Navigation (GoRouter)

| Route | Screen |
|-------|--------|
| `/` | `HelioShell` (bottom tabs) |
| `/auth` | Strap auth key |
| `/scan` | BLE device scan |
| `/settings/api` | Cloud API config |
| `/health/:dayKey` | Health monitor grid |
| `/metric/:dayKey/:metricId` | Metric drill-down |
| `/activity/detail` | Workout / session detail |

## Deploy

- **Mobile:** Play Store APK/AAB from this repo
- **API + DB + web:** `Heliolytics/deploy/install.sh` (sibling repo)
