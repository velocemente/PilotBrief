# Relief Pilot

**OFP briefing notes for relief and augmented crew — Atlas Air / Polar Air Cargo**

> ⚠️ **Disclaimer:** Relief Pilot is NOT certified aviation software. It is an informational tool only. All data must be independently verified against the official OFP and dispatch release. Do not use as a sole reference for any operational decision.

---

## Overview

Relief Pilot is a progressive web app (PWA) that parses Atlas Air and Polar Air Cargo Operational Flight Plans (OFPs) and presents a structured, interactive briefing note. Designed for iPad, optimised for iPadOS Slide Over, and works fully offline in airplane mode after the initial load.

---

## Features

### OFP Import
- PDF upload via file picker or drag-and-drop
- iOS Share sheet (open OFP PDF in Files or Mail → Share → Relief Pilot)
- Paste raw OFP text

### Briefing Tab — Three Phase Cards

**Preflight**
- ATIS letter + notes and Parking (departure)
- OEI runway picker — TLR EFP procedures from the OFP TLR section; SPECIAL proc display; SEE CO CHART notepad
- STAB trim entry
- OFP fuel (Block / MIN / FRES / GA) with Block color-coded by TOGW margin
- Notes section: airport status dots, DDG/MEL list with TLR badges, overflight permits, curfew alerts, perishables banner, free-text notepad

**Enroute**
- OFF time entry → live IN time recalculation using OFP ETE
- E.ENT — ETOPS airspace entry time (ETOPS flights)
- RDA — Redispatch Authorisation time (2 hrs before POR)
- POR — waypoint, time, and MINF (minimum fuel at Point of Redispatch)
- CO RPT schedule at 4-hour intervals with RDA/POR suppression window
- Scratchpad (keyboard + Apple Pencil Scribble)

**Descent**
- Arrival airport status row with live color dot and issue tags — tap ICAO to change arrival airport (diversion support)
- ATIS letter + notes and Parking (arrival — independent from departure fields)
- TLR Landing items — DDG items with TLR Landing or Takeoff+Landing cross-references displayed automatically below Parking
- Arrival fuel (REMF / ALT 1 / FRES / GA)

### Airports Tab
- Airport cards for every airport on the release: ORIG, DEST, DEST ALT, EN ROUTE ALT, TKOF ALT, CFS ALT
- Combined weather and NOTAM evaluation against ETA ±1 hr window (CFS uses OFP-published WX window)
- Each alert item is tappable — shows the exact NOTAM entry or TAF group that triggered it, fully wrapped
- Weather categories: Open / Operational / Instruments / 🔴 with actual ceiling and visibility values
- Curfew proximity row from STATION COMPANY NOTAMS
- Per-airport NOTAM and WX fields — auto-populated from OFP, crew-supplementable

### OFP Tab
- Original OFP rendered page-by-page from the imported PDF
- Falls back to raw text when PDF binary is unavailable

### Alternates Tab
- Parsed alternate routes (Destination Primary, Initial Airport, Initial Alternate)
- Routes are crew-editable; per-route clipboard copy
- ETP waypoint coordinates with clipboard copy (ETOPS flights)

### FCIR / ASAP Toggle
- In the flight screen nav bar, left of the FCIR/ASAP toggle
- Activates a narrative notepad on every phase for event documentation
- Flight ID turns yellow; flight list row stays highlighted across sessions
- Flight cannot be deleted while toggle is ON

### Additional
- **Per-user data isolation** — each browser profile has its own private storage
- **Dark mode** — full iOS semantic color token system (light / dark / auto)
- **Slide Over optimised** — icon-only tab bar and compact layout at ≤400pt width
- **Apple Pencil Scribble** — all text fields accept stylus input
- **Offline-first** — service worker pre-caches app shell and pdf.js at install; works fully in airplane mode after first load

---

## Files

```
index.html          ← single-file app (all HTML/CSS/JS)
sw.js               ← service worker (offline-first, cache v1.9.0-dev-r2)
manifest.json       ← PWA manifest
notam-rules.js      ← NOTAM interpretation ruleset (reference / test module)
_headers            ← Netlify/Cloudflare Pages headers (MIME, cache, security)
icons/
  icon-192.png      ← PWA icon (192×192)
  icon-512.png      ← PWA icon (512×512)
.github/
  workflows/
    deploy.yml      ← GitHub Actions → GitHub Pages on push to main
```

---

## Supported OFP Formats

- **Atlas Air** (B747-8F, B747-400F) — primary format, fully tested
- **Polar Air Cargo** (B747-400F) — same dispatch system, same format

---

## Technology

- Vanilla HTML / CSS / JavaScript — zero runtime dependencies
- pdf.js 3.11.174 (CDN, pre-cached by service worker at install)
- iOS semantic color tokens aligned to `DesignTokens.swift`
- SF Pro / `-apple-system` font stack; 8pt spacing grid; 44pt touch targets
- Service worker: offline-first cache with background revalidation

---

## Changelog

### v1.9.0-dev-r2

#### SPECI observation parsing (fix)
- Origin weather sections opening with `SPECI` (special observation) rather than routine `METAR` were not parsed — the wx/NOTAM block parser anchored on `^METAR ICAO` only, silently dropping all weather and NOTAMs for the origin airport when the OFP started the section with a SPECI
- Fixed: section anchor now matches `^(?:METAR|SPECI)\s+ICAO` — both observation types parsed identically

#### Approach minimums NOTAM rules (new)
Two new evaluation rules added to both `evalAirport()` in `index.html` and `_classifyNotamSlot()` in `notam-rules.js`:

**CAT II/III NOT AVBL → 🔴 Red**
- Fires when a NOTAM contains `CAT II.*NOT AVBL`, `CAT III.*NOT AVBL`, or similar — covers the Atlas OFP parenthetical format `RWY 07(CAT II) NOT AVBL DUE TO TEMPO OBST`
- Loss of Cat II/III precision approach capability is treated as a red alert at ETA; yellow if outside window

**IAP amended / minimums raised → ⚠ Yellow**
- Fires on NOTAMs containing: `IAP`, `INCREASED FR \d` (Atlas-style raised minimums), `DA/HAT` or `MDA/HAT` values, `PROC NA`, `LPV DA`, `LNAV MDA`, or `RNAV.*AMDT`
- Catches the full range of Atlas OFP approach procedure amendment patterns including: `ILS Z RWY 07(CAT I) INCREASED FR 287(200)FT TO 387(300)FT` and `RNAV (RNP) Z RWY 07(LPV) INCREASED FR 430(343)FT TO 510(423)FT`

#### notam-rules.js updates (v1.9.0-dev-r2)
- `NM.CAT_II_NA` — new pattern: Cat II/III NOT AVBL, handles Atlas parenthetical format
- `NM.IAP_CHG` — new pattern: IAP/ILS/LPV/LNAV/RNAV/RNP amended minimums
- `NM.APCH_CHG` — expanded: now includes `INCREASED\s+FR\s+\d` to catch Atlas minimums-raised phrasing
- `_classifyNotamSlot()` DEST/ALT path: `CAT_II_NA` → slot 2 red; `IAP_CHG` added to slot 3 condition
- `_classifyNotamSlot()` ORIG path: `CAT_II_NA` → slot 4 red; `IAP_CHG` added to slot 4 condition

#### Offline-first service worker (fix, from prior build)
- Added `<link rel="manifest">` and service worker registration — previously missing
- pdf.js pre-cached at install; safe 503 fallback on total network failure

#### NOTAM raw data panel (fix, from prior build)
- Tappable alert items now use a `<pre>` block with `word-break:break-word` — previously used an `<input>` (single-line only) which truncated long NOTAMs

#### Descent subcard additions (from prior build)
- Arrival airport status row with tappable ICAO and airport picker
- TLR Landing items auto-displayed below Parking

#### Per-user data isolation (from prior build)
- UUID-namespaced localStorage keys per browser profile

---

### v1.9.0

#### Enroute subcard
- OFF time entry cascades IN time using OFP ETE; alternate ETAs recalculate from offZ
- RDA displayed as single time (2 hrs before POR), not a window
- POR shows waypoint + time + MINF from OFP column 2

#### DDG / MEL parsing rewrite
- Structured ATA code + description parse; TLR cross-reference from TLR section
- Badges: `*TLR Takeoff` (yellow) · `*TLR Landing` (tint) · `*TLR Takeoff & Landing` (orange)
- 25-series items filtered; duplicates deduplicated

#### ATIS + Parking — per-phase isolation
- Departure and Arrival ATIS/Parking are now fully independent state fields

#### TOGW color coding fixed
- Limit codes `L` (LDW+trip) and `I` (dispatcher) now parsed correctly; was `[SP]`, now `[SPLI]`

---

### v1.8.x
- Per-NOTAM UTC timestamp window matching
- CFS / ETOPS alternate airport cards with OFP WX windows
- Curfew detection from STATION COMPANY NOTAMS
- FCIR / ASAP toggle with narrative and deletion lock
- TAF ETA window extraction (FM / TEMPO / BECMG / PROB)

---

## License

Copyright © 2025. All rights reserved.

This software is proprietary and confidential. It is provided for **private use only** by authorized personnel. Unauthorized copying, distribution, modification, public deployment, or use by any person or organization outside of those explicitly authorized is strictly prohibited.

No open-source license is granted. This repository and its contents may not be forked, redistributed, sublicensed, or used as the basis for any derivative work without prior written permission.
