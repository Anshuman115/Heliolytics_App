# Heliolytics — Architecture

## Current Phase: Phase 1 — Data Acquisition

Goal: fetch and persist every raw binary blob the strap sends. No parsing yet.

---

## System Overview

```
┌────────────────────────────────────────────────────────┐
│              Amazfit Helio Strap 2025 (ZeppOS 4.x)        │
│         user device · local timezone · BLE fetch         │
└──────────────────────────┬─────────────────────────────┘
                           │ BLE (Huami chunked protocol)
                           ▼
┌────────────────────────────────────────────────────────┐
│                   Heliolytics (Flutter)                 │
│                                                        │
│  lib/core/ble/                                         │
│    scanner.dart         → find the strap               │
│    connector.dart       → GATT connect                 │
│    device_handshake.dart       → ECDH + AES auth              │
│    type_sync_engine.dart→ request each type code       │
│    sync_orchestrator.dart → orchestrate full session  │
│    gatt_framing.dart     → chunked protocol framing     │
│    parsers/             → Phase 2 stubs (one per type) │
│                                                        │
│  lib/core/constants.dart                               │
│    knownTypeCodes       → Gadgetbridge-confirmed codes │
│    dumpTypeCodes        → what we actually fetch       │
│    probeTypeCodes       → brute-force scan range       │
└──────────────────────────┬─────────────────────────────┘
                           │ writes raw .bin files
                           ▼
┌────────────────────────────────────────────────────────┐
│              helio_dump_v3/  (app-private storage)      │
│                                                        │
│  session.json           → sessionId, startedAt, MAC    │
│  types.json             → per-type status/bytes/samples│
│  0x01_raw.bin           → activity stream (153 KB)     │
│  0x48_raw.bin           → sleep blobs (17 KB)          │
│  0x49_raw.bin           → HRV RMSSD (52 KB)            │
│  ... (22 types total)                                  │
└────────────────────────────────────────────────────────┘
```

---

## Fetch Flow (Phase 1)

```
sync_orchestrator.dart
  └─ for each code in dumpTypeCodes:
       1. type_sync_engine.dart: send 0x01 start-sync command
       2. strap replies: expected sample count
       3. type_sync_engine.dart: send 0x02 transfer command
       4. gatt_framing.dart: receive chunked packets, reassemble
       5. sync_orchestrator.dart: write raw bytes to .bin file
       6. type_sync_engine.dart: send 0x03/0x09 ACK
```

Each type is fetched sequentially (one at a time). The device rejects
concurrent requests. Full 30-day fetch takes ~3 minutes.

---

## Data Types — Fetch Coverage

| Code | Label | Size | Status | Bytes in dump |
|------|-------|------|--------|--------------|
| `0x01` | Activity (HR/steps/kind) | 8B/rec | fetched ✅ | 153 KB |
| `0x05` | Workout summaries | protobuf | fetched ✅ | 109 B |
| `0x06` | Workout details | binary | fetched ✅ | 78 KB |
| `0x07` | Debug logs | raw text | fetched ✅ | 20 MB |
| `0x0D` | PAI scores | 4B/rec | fetched ✅ | 2 KB |
| `0x13` | Stress auto | 1B/min | fetched ✅ | 20 KB |
| `0x25` | SpO₂ spot | 65B/rec | fetched ✅ | 102 KB |
| `0x26` | SpO₂ sleep | TBD | fetched ✅ | 1.3 KB |
| `0x27` | Accelerometer | 4B/rec | fetched ✅ | 3 KB |
| `0x2E` | Skin temperature | 8B/rec | fetched ✅ | 163 KB |
| `0x38` | Sleep resp rate | 8B/rec | fetched ✅ | 11 KB |
| `0x39` | Daily readiness | TBD | fetched ✅ | 3.3 KB |
| `0x3A` | Resting HR | 6B/rec | fetched ✅ | 168 B |
| `0x3B` | Activity sessions | TBD | fetched ✅ | 1.4 KB |
| `0x3D` | Max HR | 6B/rec | fetched ✅ | 96 B |
| `0x46` | Continuous HR | 4B/rec | fetched ✅ | 188 B |
| `0x48` | Sleep sessions | 594B/rec | fetched ✅ | 17 KB |
| `0x49` | HRV RMSSD | 6B/rec | fetched ✅ | 52 KB |
| `0x4A` | HRV trend | TBD | fetched ✅ | 17 KB |
| `0x4E` | Sleep segments | 4B/rec | fetched ✅ | 648 B |
| `0x55` | RR interval stream | TBD | fetched ✅ | 9 MB |
| `0x57` | RR blocks | TBD | fetched ✅ | 1.8 MB |
| `0x02` | Manual HR | 6B/rec | empty (no readings) | — |
| `0x12` | Stress manual | 1B/min | empty (no readings) | — |
| `0x2C` | Device statistics | TBD | device rejected | — |

22 types return data. 3 are empty/rejected (not errors — device just has no data for them).

---

## Tech Stack

- **Flutter** (Dart) — mobile app
- **flutter_blue_plus** — BLE
- **Riverpod** — state management
- **Hive** — local key-value storage
- **TimescaleDB** — Phase 2 server-side time-series storage

---

## Phase Roadmap

### Phase 1 (current): Data Acquisition ✅
- [x] BLE connect + auth
- [x] Fetch all 22 data types
- [x] Save raw .bin files
- [x] Session metadata JSON
- [ ] Add `0x39`, `0x55`, `0x57` to `dumpTypeCodes` in constants.dart

### Phase 2: Parsing
- [ ] Implement parsers using verified layouts in `lib/core/ble/parsers/`
- [ ] 0x48 sleep parser (594 B/session layout confirmed)
- [ ] 0x01 activity parser (round-relative timestamp alignment needed)
- [ ] 0x49 HRV parser (absolute timestamps, straightforward)
- [ ] 0x2E temperature parser (signed Int16 / 100 = °C)
- [ ] 0x13 stress parser (1 byte per minute, skip 0xFF)

### Phase 3: Storage
- [ ] TimescaleDB schema (design in backend repo)
- [ ] ETL pipeline: .bin → parsed → hypertables
- [ ] Deduplication rules (sleep: dedup on session_start; filter score=0 records)

### Phase 4: Analytics + UI
- [ ] Dashboard
- [ ] Trend analysis
- [ ] Derived metrics (RHR trend, sleep debt, HRV baseline)
