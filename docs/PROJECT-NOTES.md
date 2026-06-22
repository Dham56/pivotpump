# PivotPump — Project Notes

Technical notes and design decisions for the app. (Sanitized for the public repo — personal details and credentials are kept out and live only in the maintainer's private notes.)

## What it is
A single self-contained `index.html` fertigation calculator that matches fertilizer-pump output to center-pivot speed, so a tank empties exactly as the pivot completes its pass. No build step; runs offline in any browser. Hosted via GitHub Pages. Built from the original "HJV pivot" spreadsheet and verified to reproduce its numbers.

## Core model (do not change without care)
- **Pump:** injection rate (GPH) = `pump max capacity × setting% ÷ 100`. Settings are a percent of total capacity. Pumps are a shared library (`pumps[]`) assigned to pivots by index; they can be swapped between pivots.
- **Pivot speed → time — two conventions per pivot:**
  - **Timer** pivots: `cal` = hours for a full 360° pass at 100% timer. Pass time = `cal × 100 / timer% × deg/360`.
  - **Depth** pivots: `cal` = hours to apply 1 inch over 360°. Pass time = `cal × inches × deg/360`.
  - East Young / West Young are timer; Switzer / Spanjer / HBF are depth.
- **"Finished properly"** = tank empties exactly as the pass completes.
- **Precip readout** uses the actual pass hours (not normalized to 360°); an acre-foot = 325,851 gal.

## On-track check (observation-based)
Deliberately ignores the (often-inaccurate) dialed pump/speed. Inputs: starting gal, current gal, total degrees, beginning + current position, direction (forward = clockwise). Degrees covered = angular travel in that direction (wrap-around handled). It compares actual `gal/°` pace to the pace needed and recommends a new pivot setting and/or pump % by **scaling the current dial by the factor** — so calibration error cancels. Guards near-empty ("essentially empty") and over-limit (>100%) cases. Also derives actual **GPH** and **°/hr** between two logged checks, flagging pivot **stopped / idle / refill**. Auto-saves a snapshot to the activity log.

## Features
Find pivot speed · Find pump setting · On-track check · Log & history (log position+gallons → application profile, dry-wedge detection, Christiansen uniformity, season totals, CSV export) · Notes & photos (photos in IndexedDB, device-only) · Activity log (automatic audit trail) · Backup/Restore (JSON payload `v3`) · Cloud sync.

## Calibrations (East Young confirmed against AgSense telemetry: ~9.7–9.9 hr@100% and ~425 GPM)
| Pivot | Mode | cal | deg | pump GPH | tank | GPM | acres |
|---|---|---|---|---|---|---|---|
| East Young | timer | 9.72 hr@100% | 360 | ~55 | 3000 | 425 | 122 |
| West Young | timer | 14.88 | 360 | 30 | 3000 | 500 | 121.4 |
| Switzer | depth | 101 hr/in | 338 | 36 | 1450 | 550 | 117 |
| Spanjer | depth | 107.93 | 360 | 18 | 1450 | 550 | 117.6 |
| HBF | depth | 76.52 | 360 | 55 | 1450 | 750 | 126.9 |

## Make-up wedge
If the tank runs dry before the pass finishes, the missed wedge (degrees that got water only) should get roughly **double the normal rate** on the next pass to even out the application (its own share plus the share it missed).

## Storage & sync
- Working store is browser `localStorage` (offline-first). Photos are in IndexedDB (device-only).
- **Cloud sync:** Firebase Firestore, email/password auth, local-first. The full state (the `v3` backup payload) is mirrored to one Firestore doc per user; syncs across devices and works offline. `cloudMaybePush()` (debounced) pushes; `onSnapshot` pulls; last-write-wins; a pre-sync safety copy is stashed before applying remote. Security rule: only the signed-in owner can read/write their own doc.

## Maintenance
- **`PP_BRIEFING`** (a constant string in `index.html`, attached by the "Get help" button) is a **manual snapshot** of the project for handoff to a fresh Claude chat. Update it whenever the app changes materially (model, features, calibrations).
- Deploy: edit `index.html`, commit, `git push` → GitHub Pages rebuilds in ~30–50s.
