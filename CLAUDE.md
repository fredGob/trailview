# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TrailView is a **single-file PWA** (Progressive Web App) that analyzes GPX trail/running files entirely client-side. There is no backend, no build step, no package manager. Everything lives in `index.html` (~570 lines of embedded HTML + CSS + JS).

## Running / Developing

Open `index.html` directly in a browser, or serve it with any static server:
```bash
npx serve .
# or
python -m http.server 8000
```
The Service Worker (`sw.js`) requires HTTPS or localhost to activate.

There are **no tests, no linter, no build commands**. Validation is manual: load a GPX file and verify map, charts, and splits render correctly in both dark and light themes.

## Architecture

### Single-file structure
- **`index.html`** — the entire app: CSS variables, HTML layout, and all JS logic
- **`sw.js`** — Service Worker with network-first (map tiles) / cache-first (assets) strategy, cache name `trailview-v1`
- **`manifest.json`** — PWA metadata (standalone display, icons)

### External dependencies (CDN, no npm)
- **Leaflet 1.9.4** — map rendering
- **Chart.js 4.4.0** — elevation, HR, and pace charts
- **Google Fonts** — Space Mono (monospace/stats) + DM Sans (body)

### Key JS globals in `index.html`
| Variable | Purpose |
|----------|---------|
| `mapInst` | Leaflet map instance |
| `cElev`, `cHR`, `cPace` | Chart.js instances (elevation, heart rate, pace) |
| `ZONES` | Heart rate zone definitions (Z1–Z5 with colors and %FCmax ranges) |

### Data pipeline (all in `processGPX()`)
1. **Parse** — DOM-based XML parsing of GPX `<trkpt>` elements (lat, lon, ele, time, hr)
2. **Distances** — Haversine formula, cumulative
3. **Elevation** — Kalman filter smoothing (`kalman()`, noise=10), then D+/D− with 3m dead-band (`calcDPlus()`)
4. **HR zones** — 5-zone model from age-based FCmax (220 − age)
5. **Splits** — per-km pace and average HR
6. **Render** — stats strip, Leaflet map with polyline + start/end markers, 3 Chart.js charts, zone bars, split cards

### Theme system
- CSS variables under `:root, [data-theme="dark"]` and `[data-theme="light"]`
- JS reads variables via `getThemeColors()` for chart grids/ticks, map markers, gradients
- `applyChartTheme()` live-updates Chart.js instances on toggle
- Persisted in `localStorage` key `tv-theme`; falls back to `prefers-color-scheme`

## Conventions

- **Language**: French UI (labels, toasts, section titles)
- **Fonts**: Space Mono for all numeric/stat values; DM Sans for body text
- **Colors**: use CSS variables (`--accent`, `--bg`, `--surface`, etc.) — avoid hardcoded hex in JS; read from computed styles via `getThemeColors()` when needed
- **HR zone colors** (Z1–Z5) are semantic and stay the same across themes
- All chart colors (grid, ticks, gradients) must adapt to the active theme
