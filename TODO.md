# Heliolytics — TODO

> Phase tracker. Updated as work progresses.

---

## Phase 1: Data Acquisition — 95% done ✅

### Completed ✅
- [x] BLE scan + connect
- [x] ECDH + AES authentication (sect163k1 curve)
- [x] Chunked protocol (send/receive/ACK)
- [x] Fetch all 22 data-returning type codes
- [x] Save raw .bin files per type
- [x] Session metadata: `session.json`, `types.json`
- [x] Validate dump against Zepp export (96.3% sleep match, 99%+ on all other types)

### Remaining ⬜
- [ ] Add `0x39`, `0x55`, `0x57` to `dumpTypeCodes` in `constants.dart`
  - These 3 types return data but are currently in `probeTypeCodes` only
  - Fix: move to `dumpTypeCodes` (3-line change)

### Won't do / Not needed
- `0x02` (manual HR), `0x12` (manual stress): device returns nothing, no readings exist
- `0x2C` (device stats): device rejects this request
- `0x07` (debug logs): intentionally excluded from `dumpTypeCodes` — 20 MB of firmware noise

---

## Phase 2: Parsing — not started

Binary layouts are verified in parser modules. Implementation order by confidence:

### High confidence (implement first)
- [ ] `0x48` sleep sessions — **594B/session, 96.3% exact match against Zepp export**
  - File: `lib/core/ble/parsers/sleep_session.dart` (currently a stub)
  - Layout: see `lib/core/ble/parsers/sleep_session.dart`
- [ ] `0x49` HRV RMSSD — 6B/rec, absolute timestamps, simple
- [ ] `0x3A` resting HR — 6B/rec, same format as 0x49
- [ ] `0x3D` max HR — 6B/rec, same format
- [ ] `0x38` sleep resp rate — 8B/rec
- [ ] `0x2E` skin temperature — 8B/rec, signed Int16 / 100 = °C
- [ ] `0x13` stress — 1 byte per minute, skip 0xFF
- [ ] `0x25` SpO₂ spot — 1 header byte + 65B records
- [ ] `0x05` workout summary — protobuf (layout confirmed: duration, sport_type, calories, HR)

### Medium confidence
- [ ] `0x01` activity — 8B/rec, round-relative timestamps (need roundStart from handshake)
- [ ] `0x26` SpO₂ sleep — layout TBD (small file, low priority)
- [ ] `0x0D` PAI scores — layout TBD

### Unknown layout (probe first)
- [ ] `0x39` readiness — 3.4 KB, layout unknown
- [ ] `0x4A` HRV trend — 17 KB, layout unknown
- [ ] `0x4E` sleep segments — 648 B, layout unknown
- [ ] `0x55` RR interval stream — 9 MB, layout unknown
- [ ] `0x57` RR blocks — 1.8 MB, layout unknown
- [ ] `0x3B` activity sessions — 1.4 KB, layout unknown

---

## Phase 3: Storage — not started

- [ ] Set up TimescaleDB
- [ ] Create hypertables (schema in Heliolytics backend repo)
- [ ] ETL pipeline: .bin → parsed → hypertables
- [ ] Dedup rules:
  - Sleep: discard `score=0 AND total_mins=0`, deduplicate on `session_start`
  - Zepp CSV backfill for May 7–8 (before BLE fetch window)

---

## Phase 4: Analytics + UI — not started

- [ ] Design dashboard
- [ ] Sleep trends (stage breakdown, score trend, sleep debt)
- [ ] HRV baseline + daily deviation
- [ ] Resting HR trend
- [ ] Activity zones from 0x01 kind bytes
- [ ] Readiness score (once 0x39 is decoded)

---

## Known Facts (don't re-research)

| Question | Answer |
|----------|--------|
| What resolution is 0x01? | 1 record per minute (round-relative) |
| Is 0x48 pre-computed or raw? | Pre-computed on device — 594B per night, fully decoded |
| What's the timezone byte? | `0x16` = 22 × 15 min = 330 min = UTC+5:30 (IST) |
| How many nights in the dump? | 29 sessions (BLE), 33 sessions (Zepp export incl. May 7–10) |
| Do 0x02/0x12/0x2C return data? | No — device returns empty/rejected for all three |
| What does 0x01 byte[3] contain? | Heart rate bpm (confirmed: sleep avg 59.5, active avg 73.9) |
| What is 0x13 record size? | 1 byte per minute (NOT 4 bytes — earlier analysis was wrong) |
| What is 0x25 record stride? | 65 bytes (1 header byte + 65B records) |
| Temperature decode formula | `getInt16(bytes, 2) / 100.0` (signed, not unsigned) |
