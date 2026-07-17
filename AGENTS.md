# AGENTS.md — Heliolytics_App (Flutter)

## Project
Flutter app for Heliolytics. Dart + Riverpod + flutter_secure_storage + BLE.
Public repo, deployed to Play Store.

Part of a 3-repo system, cloned as **siblings** under one parent:

| Repo | Role |
|------|------|
| `Heliolytics_App` (this) | Flutter — BLE sync, uploads raw session bytes |
| `Heliolytics` | Go API — parses, stores, serves metrics. The hub |
| `Heliolytics_Web` | Next.js dashboard — reads metrics |

**The core split: the phone never parses health data and never persists it.** It
fetches raw bytes off the strap and uploads them. The server parses. The app reads
the parsed result back over HTTP.

## Docs — read before changing a feature

| Doc | When |
|-----|------|
| [docs/features/ble-sync.md](docs/features/ble-sync.md) | Sync, coverage, type fetch, upload |
| [docs/features/band-alerts.md](docs/features/band-alerts.md) | Call/app forwarding, vibration patterns |
| [docs/features/home-and-rings.md](docs/features/home-and-rings.md) | Shell, tabs, rings, the two-tier metrics split |
| [docs/features/sleep.md](docs/features/sleep.md) | Hypnogram, clock-axis consistency chart |
| [docs/features/activity.md](docs/features/activity.md) | Workouts, auto sessions, HR zones |
| [docs/features/settings-and-device.md](docs/features/settings-and-device.md) | Auth key, pairing, cloud API, signing |

`docs/features/` is published. The rest of `docs/` — `protocol/` (byte layouts, type
codes, `roundStart`, paging), `validation/` (offline replay, accuracy), and
`decisions/` (why it's built this way) — is **local-only and gitignored**. Consult
those locally; never cite them from a published file.

## Structure Rules (layer-first)
- lib/screens/ → full pages only
- lib/widgets/ → screen-specific UI chunks (charts, sections)
- lib/providers/ → Riverpod Notifiers + provider definitions
- lib/services/ → BLE, API, repos, config, network (no UI)
- lib/models/ → entities, JSON models
- lib/router/ → GoRouter
- lib/utils/ → helpers, logger, formatters
- lib/constants/ → app literals
- lib/design_system/ → shared tokens + reusable components
- Flow: screens → providers → services → models
- One file, one responsibility. If a file handles more than one concern, split it.
- No file longer than 150 lines.
- No class with more than one reason to change.

> Known debt: 28 files still exceed the cap, worst first `band_link.dart` (432) and
> `health_monitor_screen.dart` (338). Don't add new ones; split when you touch one.

## Naming
- Files: snake_case (home_screen.dart, live_health_provider.dart)
- Classes: PascalCase (HomeScreen, LiveHealthNotifier)
- Providers: camelCase + Provider suffix (liveHealthProvider)
- Variables: camelCase, descriptive (hrvRmssdMs not value)

## Riverpod Rules
- Providers live in lib/providers/
- No logic inside build() — move to provider
- AsyncNotifier for all async state, never FutureBuilder
- ref.watch in build; ref.read in callbacks

## Code Style
- No widget deeper than 3 levels of nesting, extract widgets
- No build() longer than 40 lines
- Every screen is a ConsumerWidget or ConsumerStatefulWidget
- No direct API calls from UI layer, always through provider → service
- Sync to backend only — no on-device health-data persistence
- No print() — use AppLogger.instance.log in lib/utils/app_logger.dart
- No magic numbers — all literals in lib/constants/constants.dart
- Spacing: HelioSpacing only (design_system/tokens/helio_spacing.dart)
- Metric colors: HelioMetricColors (design_system/tokens/helio_metric_colors.dart)

## BLE Rules
- All BLE logic lives in lib/services/ble/
- Key modules: band_link, type_sync_engine, device_handshake, gatt_framing,
  sync_orchestrator (provider), sync_orchestrator_run, pairing_curve_b163,
  encrypted_endpoint, sync_page_anchor
- Never call BLE methods directly from UI
- One parser file per data type code in lib/services/ble/parsers/
  (fetch framing only — health parsing is the server's job)
- `BandLinkPort` is the test seam. Change its signature and the mocks in `test/`
  must follow, or `flutter analyze` fails with `invalid_override`
- One BLE link, shared. `BandSessionProvider` is the mutex — band alerts mode
  blocks sync and live HR by design

## Cross-repo invariants — break these and another repo breaks
- **`X-Heliolytics-Token` format** (`ts.nonce.sig`, HMAC-SHA256 over `"ts:nonce"`)
  must stay byte-identical across `lib/services/network/heliolytics_token.dart`,
  Go `internal/auth/signing.go`, and web `lib/api/signing.ts`
- **The server owns sync state.** No phone-side bookmarks — ask `/metrics/coverage`
- Per-page `roundSegments` anchors must be uploaded; server parsers depend on them

## Verification
- `flutter analyze` is the authoritative compile check — must be **0 errors**
- An APK build needs several GB free

## Git
- Commits: type(scope): message — e.g., feat(ble): add workout parser
- Types: feat, fix, chore, refactor, test, docs
- One logical change per commit. No WIP commits on main.

## Published vs. local-only

This repo is public. `AGENTS.md` and `docs/features/` **are published** — write them
for an outside reader, not just for yourself.

Local-only (in `.gitignore`, on disk for personal reference only):

| Path | Purpose |
|------|---------|
| `docs/protocol/` | Byte layouts, type codes, handshake internals |
| `docs/validation/` | Offline replay + accuracy write-ups |
| `docs/decisions/` | Design rationale, discussion index |
| `docs/README.md` | Local index across all of the above |
| `scripts/` | Local Python validation helpers |
| `helio_dump*/` | Raw BLE session dumps |
| `zepp-export/` | CSV ground truth |
| `reports/` | Generated HTML validation reports |
| `reference_parsed_v*/` | Parsed reference output |

**Never cite a local-only path from a published file.** A published doc that links to
`docs/protocol/` is a broken link for everyone who clones the repo — describe the
thing in prose instead.

**`test/` is tracked and public** — 4 files under `test/core/` and
`test/features/`. Treat test edits as real commits.

## Attribution and legal hygiene
- No third-party project names in code comments, commit messages, or tracked markdown
- No "ported from", "direct port", or file-path attribution to other repos
- Describe behavior in terms of Huami BLE / Gadgetbridge-compatible protocol only
- Hardware/protocol names (Huami, ZeppOS, Amazfit) are fine where they name the
  actual device or wire format. **Competitor product names are not.**
