# PilotBrief

**OFP briefing notes for relief and augmented crew**  
Version 1.9.5 · SW cache `pb-v1.9.6` · Atlas Air / Polar Air Cargo OFP format

---

## What It Is

PilotBrief is a progressive web app (PWA) that parses an Atlas Air / Polar Air Cargo Operational Flight Plan and produces a structured, interactive crew briefing note. It runs entirely in the browser — no server, no account, no data leaves the device.

**Core capabilities:**
- PDF, iOS Share, or text OFP import (Atlas/Polar format)
- Three-phase briefing note: Preflight / Enroute / Descent
- Parsed flight header, fuel (Block, MIN, REMF, ballast), ETOPS, POR, CFS/ETP, OEI/TLR, DDG/MEL, CAT II/III authorisation, slot time, VZFW/CK weight limit alert
- Airport cards with weather and NOTAM evaluation for every release airport
- NOTAM tree grouped by runway in numerical order — Closed → Approach/Navaid → Lighting → Procedure
- Weather evaluation per FOM §7.1.31 — Open / Operational / Red thresholds
- Descent phase: selectable airport summary (DEST + alternates), DDG landing items, CAT II/III per runway
- Curfew detection from Company NOTAMs
- FCIR/ASAP narrative notepad with copy-to-clipboard
- Offline-first — works in airplane mode after first load
- iPad Slide Over optimised (two-row nav bar at ≤ 400 px)

---

## Repository File Structure

```
/
├── index.html          # Entire application (HTML + CSS + JS, single file)
├── sw.js               # Service worker — offline-first cache + Web Share Target relay
├── manifest.json       # PWA manifest — app name, icons, display mode, share_target
├── _headers            # HTTP cache / security headers (Netlify / Cloudflare Pages)
├── .gitignore          # Excludes OFP PDFs and dev artefacts
├── icons/
│   ├── icon-192.png    # PWA icon — Add to Home Screen
│   └── icon-512.png    # PWA icon — splash / store
└── .github/
    └── workflows/
        └── deploy.yml  # GitHub Actions → GitHub Pages deployment
```

---

## Deployment

### GitHub Pages (recommended)

1. Push this repository to GitHub.
2. Go to **Settings → Pages → Source** and select **GitHub Actions**.
3. The `deploy.yml` workflow triggers automatically on every push to `main`.
4. Live URL: `https://<your-username>.github.io/<repo-name>/`

### Add to iPad Home Screen

1. Open the live URL in Safari on iPad.
2. Tap **Share ↑ → Add to Home Screen**.
3. The app installs as a standalone PWA with offline capability.
4. iOS Share from Files / Mail routes OFP PDFs directly into the app — see [iOS Share](#ios-share-target) below.

---

## Importing an OFP

Three methods are supported:

| Method | How |
|---|---|
| **① PDF File** | Tap **＋** → tap the drop zone or drag an OFP PDF onto it |
| **② iOS Share** | Install to Home Screen first. In Files or Mail, open the PDF → Share ↑ → select **PilotBrief** |
| **③ Paste Text** | Tap **＋** → paste OFP text into the text area → tap Import Text |

---

## iOS Share Target

PilotBrief registers as a system share target for PDF files via the [Web Share Target API](https://web.dev/web-share-target/). When a PDF is shared to PilotBrief from the iOS share sheet, the service worker intercepts the multipart POST, stashes the file in an ephemeral relay cache (`pb-share-relay`), and redirects to the app shell, which reads and parses the OFP automatically.

**Requirement:** The app must be installed to the iPad Home Screen. The share target is not active from a browser tab.

---

## Offline Behaviour

The service worker pre-caches `index.html`, `manifest.json`, and the pdf.js CDN assets at install time. After the first successful online load, the app is fully functional in airplane mode. Flight data is persisted in `localStorage` — the service worker never touches it.

Cache key: `pb-v1.9.6` — bump this string in `sw.js` with every release to force clients to pick up updated assets.

---

## Versioning

| Identifier | Current | Meaning |
|---|---|---|
| `APP_VERSION` | `1.9.5` | User-visible feature version, shown in the app header |
| `CACHE_NAME` | `pb-v1.9.6` | SW cache key — incremented on every deployment to force cache refresh |

`APP_VERSION` tracks user-facing feature changes. `CACHE_NAME` is incremented on every deployment (including infrastructure-only changes such as the v1.9.6 Web Share Target addition) to ensure all clients receive updated assets.

---

## Aircraft Support

| Type | OFP Format |
|---|---|
| B747-400F (CF6-80C2) | Atlas Air / Polar Air Cargo |
| B747-8F | Atlas Air |

NOTAM and weather rulesets are modular and version-controlled per ADR-001. Customer-specific rulesets are designed for future hot-swap loading at runtime.

---

## Disclaimer

PilotBrief is **not** certified aviation software. It is an informational tool only. All data must be independently verified against the official OFP and dispatch release. Do not use as a sole reference for any operational decision.
