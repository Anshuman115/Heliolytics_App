# Heliolytics App — Status

Flutter-only repo. Server and web status live in sibling repos.

## Done

- [x] BLE sync → cloud upload (no on-device health-data persistence)
- [x] Server-driven per-type fetch windows (coverage API; reinstall-safe)
- [x] Metrics UI from Go API (days, series, sleep, workouts, activities)
- [x] On-device logic limited to BLE paging anchors; parsing is server-side
- [x] Layer-first `lib/` layout; one responsibility / ≤150 lines per file
- [x] Dark UI: rings, health-monitor grid, design-system tokens
- [x] Lazy-loaded per-minute datasets (series/HR/temp) to cut API load
- [x] Recovery/readiness score shown (device `0x39`, else server-computed)
- [x] Settings + health-monitor UI refactor; exact step count; cold-start nav fix
- [x] Singleton `AppLogger` + Talker Dio logging (debug)

## Left

- [ ] Production backfill constants (currently dev-tuned in `lib/constants/constants.dart`)
- [ ] Play Store release pipeline for the `v5` line

See **Heliolytics** repo for API/parser/recovery-score work and **Heliolytics_Web** for the dashboard.
