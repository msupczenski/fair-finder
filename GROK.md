# Fair Finder — Grok bot brief

Give this file to any Grok that will touch the app. Repo first, then this doc. Do not invent a new stack.

## What it is

Phone map for walking the **Bloomsburg Fair** (PA). Search any of ~623 official vendors, drop a pin, walk there. Live GPS + accuracy circle. Full-screen PWA feel.

- Live: https://fairfinder.supczenski.com
- Repo: https://github.com/msupczenski/fair-finder
- Source list: https://bloomsburgfair.com/vendors/
- Host: Cloudflare Pages off `main`
- Owner phones from iOS Safari / home-screen. Cache is aggressive.

## Current live version

**v33** (2026-09-29). Dark mode was added in v34–v35 and **reverted**. Do not put Carto/Stadia/Mapbox tiles back — Carto showed “API key required”.

Status bar bottom-right must show `v33` after a ↻ refresh.

## Files that matter

| Path | Role |
|---|---|
| `index.html` | Entire public app. Single file + JSON. Bump `VER` on every real change. |
| `pin.html` | Hidden calibrate page (password `bloom26`). Multi-pin add / drag / delete / “pin here” from GPS. |
| `vendor-fixes.json` | ~42 owner-walked GPS pins. **Source of truth for exact stands.** Merge, never wipe. |
| `vendors-0.json` … `vendors-7.json` | Official vendor dump. Keep name/loc/desc in parity with the fair site. |
| `landmarks.json` | Halls / barns. |
| `_headers` | `Cache-Control: no-store` for HTML/JSON. Do not weaken. |

There is no build step. No npm. No React.

## How a pin is placed (`posOf`)

1. Exact match in `vendor-fixes.json` or `localStorage.fairVendorFixes` → use that lat/lng.
2. Else `alongStreet()`: booth number along a street that already has calibrated samples.
3. Else `rawPos()`: intersection grid from `DEFAULT_REFS` (7th & B, 4th & B, 11th Herb, E Ave, Cattle Barn, etc.).

Addresses look like `A Ave W: 103`, `9th Street: 46`, `E Ave W:40/50`, `10th Street: 117 & 4th Street: 28`. Parse **first clause only**. Multi-location vendors stay at the first clause.

Numbered streets run roughly N–S. Avenues A–F run roughly E–W. OSM labels on the grounds are the visual check, not the street model.

## Calibration (owner workflow)

1. Open `/pin.html`, password `bloom26`.
2. Search vendor → stand on the spot → **Pin here** (uses GPS) or tap the map.
3. Tap a pin → Edit (drag) or Delete.
4. Export JSON from that page and paste it here. Merge into `vendor-fixes.json` in the repo so every phone gets it.
5. Phone-only localStorage is not enough — owner needs pins to survive a cache clear and to show up for the coding Grok later.

Intersection refs the owner already walked (keep these):

- 7th & B `40.995483, -76.467482`
- Veterans Way & 5th `40.994636, -76.466171`
- 4th & B `40.995845, -76.466701`
- 11th Herb Row `40.995366, -76.469628`
- E and F by grandstand `40.995307, -76.463637`
- B Ave & 2nd `40.997586, -76.462771`

Vince’s Cheesesteak is `E Ave W:40/50` and has a walked pin. Do not “fix” it by re-parsing the address.

## What the owner cares about

- Finding food / leather / bikes / crafts while walking the grounds.
- Pins that sit on the **labeled OSM street**, not a guessed parallel.
- Full screen (browser chrome is annoying; PWA meta is already in `index.html`).
- Keyboard must not cover the result list (suggest panel is **above** the map; `body.typing` hides the bottom sheet).
- Bottom sheet stays short and can collapse. Do not grow a pin dump list on the main map.
- iPhone home-screen cache: new routes (`/pin.html`) beat fighting `/calibrate`. Always bump `VER` and `?v=` + `cache: 'no-store'`.

## Hard rules for coding Groks

1. **Always send the full `index.html` body** when using GitHub create-or-update. A stub overwrite has happened more than once. Check the committed `size` is ~21k, not 8 bytes.
2. Read current `index.html` SHA before updating.
3. Never delete `vendor-fixes.json` keys unless the owner says that pin is wrong.
4. Do not add tile APIs that need keys.
5. Do not add “I am here” extra buttons; locate FAB `◎` and follow `➤` are enough.
6. Food filter showing every food stand at once is noisy — owner has complained. Prefer search-first; chips are secondary.
7. Self-test on the live URL after deploy: search `Vince's`, `Asian Grill`, `Barb's Funnel Cakes`, `9th Street`. Pin must sit on the named road in OSM, not a block off.
8. If pins look “all off,” stop extrapolating from one sample. Walked GPS beats math.

## Known still-wrong geometry

OSM 9th Street and the interpolated 9th line have not agreed. Highlight polyline can cut across 7th/8th. Owner asked to highlight the road; v33 does that from fitted booth step, which is why 9th looks drunk. Prefer highlighting the **OSM-aligned grid** (lock numbered streets to `ST_LNG[n]`, avenues to `AVE_LAT[letter]`) if you touch highlight again.

A Ave W booths (Barb’s 103 vs Carl’s 79) flew north toward West Pine when a single pin + default step was applied. Do not revive that.

## Deploy

Push `main` → Cloudflare Pages. No Wrangler needed for normal HTML edits.

After push: owner taps ↻ (rewrites `?v=timestamp`). If iOS still shows old `VER`, tell them to close the tab, not just pull-to-refresh.

## How to continue a session

```
Repo: https://github.com/msupczenski/fair-finder
Live: https://fairfinder.supczenski.com
Read GROK.md and index.html first.
Current VER must stay visible in the sheet.
I will paste vendor-fixes JSON when I walk more stands.
```

If they paste JSON, merge by vendor **name** into `vendor-fixes.json` and bump nothing else unless they asked for a UI change.
