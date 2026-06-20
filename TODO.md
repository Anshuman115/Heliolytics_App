# Heliolytics App — Status

Flutter-only repo. Server and web status live in sibling repos.

## Done

- [x] BLE sync → cloud upload (no local .bin persistence)
- [x] Incremental watermarks + workout backfill
- [x] Metrics UI from Go API (days, series, workouts, activities)
- [x] On-device parsers limited to BLE paging anchors only
- [x] Layer-first `lib/` layout (screens / providers / services / models)
- [x] v3 dark UI redesign (`design_system/` + restyled screens)
- [x] Dead code cleanup (orphan routes, duplicate spacing/colors, unused widgets)
- [x] Singleton `AppLogger` + Talker Dio logging (debug)

## Left

- [ ] Production backfill constants (currently dev-tuned in `lib/constants/constants.dart`)
- [ ] Play Store release pipeline

See **Heliolytics** repo for API/parser work and **Heliolytics_Web** for dashboard.
