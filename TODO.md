# Heliolytics App — Status

Flutter-only repo. Server and web status live in sibling repos.

## Done

- [x] BLE sync → cloud upload (no local .bin persistence)
- [x] Incremental watermarks + 90-day workout backfill
- [x] Metrics UI from Go API (days, series, workouts, activities)
- [x] On-device parsers limited to BLE paging anchors only

## Left

- [ ] Production backfill constants (currently dev-tuned in `lib/core/constants.dart`)
- [ ] Play Store release pipeline

See **Heliolytics** repo for API/parser work and **Heliolytics_Web** for dashboard.
