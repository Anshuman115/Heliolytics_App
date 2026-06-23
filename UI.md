# Heliolytics App — UI

Dark, metric-first interface built on a small design system. Presentation lives in
`design_system/` (tokens + reusable components) and `screens/` + `widgets/`; all data
comes from providers (no logic in widgets).

## Navigation

Bottom tabs (`HelioBottomNav`) inside `HelioShell`:

| Tab | Screen |
|-----|--------|
| Home | `HomeScreen` — recovery / sleep / strain rings, my-day, activities |
| Sleep | `SleepHubScreen` — nightly list + hero |
| Activity | `ActivityHubScreen` — workouts / auto sessions |
| More | `SettingsHubScreen` — device, cloud API, diagnostics |

Routes (`GoRouter`): `/` shell · `/auth` · `/scan` · `/settings/api` ·
`/health/:dayKey` (health-monitor grid) · `/metric/:dayKey/:metricId` (drill-down) ·
`/activity/detail`. An auth redirect routes to `/auth` only when no strap key is stored.

## Screens

- **Home** — top date nav + battery, primary score rings, health-monitor shortcut, my-day & activities.
- **Health monitor** — live/continuous HR hero + a tile grid (Resting HR, HRV, SpO₂, Resp Rate, Skin Temp, Stress); each tile drills into a metric.
- **Metric detail** — hero ring/value + stats + chart. Per-minute series (HR, temperature, stress, …) load **lazily** on open and are cached, keeping the home path light.
- **Sleep / Activity** — night list with stage breakdown; workouts and auto-detected sessions with per-session HR/calories.
- **Settings** — device hero (status, battery, sync/upload/scan), Cloud API, diagnostics (raw dump), about.

## Design system (`design_system/`)

- **Tokens** — `helio_colors`, `helio_metric_colors` (per-metric accents), `helio_spacing`
  (single 4/8/12/16/24/32 scale), `helio_typography` (ALL-CAPS metric labels, large score
  numerals), `helio_radii`.
- **Theme** — `helio_theme` (dark canvas `#000000`, elevated surfaces, white-@10% borders).
- **Components** — score rings, surface cards, vital tiles, monitor panel, section headers,
  top/bottom nav, date nav, sync strip, empty/loading states, buttons, text fields.

## Conventions

- Every screen is a `ConsumerWidget` / `ConsumerStatefulWidget`; `ref.watch` in `build`, `ref.read` in callbacks.
- Spacing only via `HelioSpacing`; metric colors only via `HelioMetricColors`; no magic numbers.
- One responsibility per file, ≤150 lines; widgets no deeper than ~3 levels.

See [ARCHITECTURE.md](ARCHITECTURE.md) for data flow and layering.
