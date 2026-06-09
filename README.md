# 🌾 PivotPump

A single-file fertigation calculator for center-pivot irrigation. It matches **fertilizer-pump output** to **pivot speed** so a tank of fertilizer empties exactly as the pivot completes its pass — and tells you, mid-run, whether you're on track to finish.

No install, no build, no internet needed. The whole app is `index.html` — open it in any browser (phone or desktop) and it runs offline. Your pumps and pivots are saved in the browser.

## What it does

Three modes:

1. **Find pivot speed** — given the gallons in the tank and the pump setting, find the pivot speed (timer % or target depth) that empties the tank as the pass finishes.
2. **Find pump setting** — given the gallons and the pivot speed, find the pump setting (% of capacity).
3. **On-track check** — given starting gallons, current gallons, total degrees, and degrees left, measure your **actual** pace (gallons used vs. degrees covered) and report whether you'll finish empty, run dry early, or have leftover — plus the exact new pivot/pump setting to correct it. Because it measures real consumption, it stays accurate even when the pump or pivot speed are off-calibration.

There's also an optional accuracy check (enter hours run + the dial settings) that tells you how far off the pump and pivot each are.

## Setup areas

- **⚙️ Pumps** — each pump is defined once by its **max capacity (GPH at 100%)**; injection rate = capacity × setting% ÷ 100. Assign a pump to a pivot in that pivot's Edit panel. Pumps can be swapped between pivots.
- **Pivots** — each carries a calibration. Two conventions:
  - **Timer** pivots: `cal` = hours for a full 360° pass at 100% timer. Pass time = `cal × 100 / timer% × deg/360`.
  - **Depth** pivots: `cal` = hours to apply 1 inch over 360°. Pass time = `cal × inches × deg/360`.
- **💾 Backup / Restore** — Copy your setup to text (save it in a note or email), and paste it back to restore or move to a new device. The in-browser data is the only copy, so back it up.

## The model

- Injection rate (GPH) = pump max capacity × (setting% ÷ 100)
- "Finished properly" = tank empties exactly as the pass completes
- Water depth readout uses actual pass hours; an acre-foot = 325,851 gallons

Math mirrors the original "HJV pivot" spreadsheet it was built from, verified to reproduce its numbers.

## Preloaded pivots

Switzer, Spanjer, HBF (depth-controlled) and East Young, West Young (timer-controlled), with seed calibrations from the source spreadsheet. Update pump max-capacities to your real equipment in the Pumps area.

## Notes

- Predictive modes assume injection rate scales linearly with the pump setting — true for a fixed-rate injector.
- All state lives in browser `localStorage`; clearing site data wipes it (use Backup/Restore).
