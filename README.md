# Relief Pilot

**OFP briefing notes for relief and augmented crew — Atlas Air / Polar Air Cargo**

> ⚠️ **Disclaimer:** Relief Pilot is NOT certified aviation software. It is an informational tool only. All data must be independently verified against the official OFP and dispatch release. Do not use as a sole reference for any operational decision.

---

## Overview

Relief Pilot is a progressive web app (PWA) that parses Atlas Air and Polar Air Cargo Operational Flight Plans (OFPs) and presents a structured, interactive briefing note. It is designed for iPad, optimised for iOS Slide Over, and works fully offline in airplane mode after the initial load.

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
- Flight cannot be deleted while toggle is ON — prevents accidental loss before report submission

### Additional
- **Per-user data isolation** — each browser profile has its own private storage; shared iPads with separate Safari profiles maintain separate flight records
- **Dark mode** — full iOS semantic color token system (light / dark / auto)
- **Slide Over optimised** — icon-only tab bar and compact layout at ≤400pt width
- **Apple Pencil Scribble** — all text fields accept stylus input
- **Offline-first** — service worker pre-caches app shell and pdf.js at install; works fully in airplane mode after first load

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

#### Offline-first service worker (new)
- Added `<link rel="manifest">` and service worker registration — previously neither was wired into the app, so offline mode never worked
- pdf.js and its worker are now pre-cached at SW install time, making PDF import available offline
- Proper cache-first fetch strategy with background revalidation; safe 503 fallback on total network failure; no bare `fetch()` passthrough

#### NOTAM raw data display (new)
- Tappable alert items on Airport cards now show the triggering NOTAM entry or TAF group in a scrollable `<pre>` block with full word-wrapping — previously used an `<input>` (single-line only) which truncated all data

#### Descent subcard — arrival airport status (new)
- Live color dot + issue tags for the selected arrival airport at the top of the Arrival subcard
- Tap the ICAO code to switch to any non-origin airport on the release (diversion support)

#### TLR Landing items on Descent (new)
- DDG items with TLR Landing or Takeoff+Landing cross-references appear automatically below the Parking field on the Descent phase

#### Per-user data isolation (new)
- Each browser profile generates a stable UUID on first visit; all flight records and preferences are namespaced to that UUID
- Crew members sharing one iPad but using separate Safari profiles have completely isolated data

#### Slide Over layout (new)
- `@media (max-width: 400px)` breakpoint targets iPadOS Slide Over (~320pt)
- Tab bar icons only (labels hidden); enroute split stacks vertically; tighter padding throughout

#### Airport card weather alert changes
- PROB40 standalone trigger removed — captured by ceiling/visibility triggers if relevant
- "Below mins" label removed — alert shows `🔴 — VIS x  CIG xft` directly

#### Instructions screen (revised)
- Full rewrite covering all current features: phase tabs, enroute events, descent airport picker, TLR Landing items, tappable alerts, weather categories, data isolation
- Help icon `?` added to flight screen nav bar (left of FCIR toggle), links directly to Instructions
- Instructions accessible from both the flight screen and Settings

---

### v1.9.0

#### Enroute subcard
- OFF time entry cascades IN time using OFP ETE (`tripMinutes`); alternate ETAs recalculate from offZ
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

#### Tappable airport alert triggers (new)
- Each alert row with a `›` chevron opens the raw NOTAM/TAF source in a read-only sheet

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
