# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Two self-contained, zero-dependency HTML golf tracker apps. Each file is a complete application — HTML + CSS + JS in a single file. No build step, no package manager, no server. Open directly in a browser.

| File | Purpose |
|---|---|
| `handicap-tracker.html` | Handicap improvement tracker (16 → 10 goal, WHS calculator, stats, insights) |
| `cognizant-classic-tracker.html` | PGA Tour tournament leaderboard tracker |

## Development

**Run:** Open the file directly in a browser (`open handicap-tracker.html` or drag into browser).

**No build, lint, or test commands exist.** All development is live-reload via browser refresh.

For rapid iteration, use browser DevTools (F12) — the Console catches JS errors and localStorage can be inspected under Application → Storage.

**Clear persisted data during testing:**
```js
localStorage.clear()  // in browser console
```

## Architecture: handicap-tracker.html

The JS is organized into named sections marked with `// ── SECTION ──` comments (lines ~502–1145). Reading order matters:

1. **STORAGE** (line ~502) — Two localStorage keys: `hcp_rounds` (array of Round objects) and `hcp_profile` (goal config). All reads/writes go through `getRounds()` / `saveRounds()` / `getProfile()` / `saveProfileData()`.

2. **WHS HANDICAP CALCULATOR** (line ~524) — `calcDifferential(gross, rating, slope)` computes a single differential. `calcHandicapIndex(rounds)` uses `WHS_TABLE` (reduced-round lookup for <20 rounds) to return the index. `buildTimeSeries(rounds)` produces the per-round historical index array used by the chart.

3. **VIEWS** (line ~615) — `showView(name)` switches between `dashboard`, `chart`, `stats`, `insights`, `history`. The chart is only drawn when its view is active (lazy render).

4. **BENCHMARKS** (line ~893) — `BENCHMARKS` array is the single source of truth for all stat targets (16-hcp baseline vs 10-hcp goal). Both the Stats panel and Insights derive from this array.

5. **INSIGHTS** (line ~969) — `calcStats5(rounds)` computes trailing 5-round averages, then gaps are normalized against the benchmark range. The stat with the highest `normalized` gap score surfaces as the primary insight.

6. **REFRESH** (line ~1130) — `refresh()` re-renders all views. Call this after any data mutation (save/delete round, profile change).

**Round object shape:**
```json
{
  "id": "uuid", "date": "2026-03-01", "course": "...", "tees": "White",
  "courseRating": 72.0, "slopeRating": 130, "grossScore": 87,
  "fairwaysHit": 8, "fairwaysTotal": 14, "gir": 6, "putts": 33,
  "penalties": 2, "differential": 12.4
}
```

## Architecture: cognizant-classic-tracker.html

Simpler, static-data app. Player scores are hardcoded in a JS array near the top of the `<script>` block. Key constants: `PAR = 71`, `TOTAL_PAR = 284`. No localStorage. `renderTable()` / `filterTable()` / `sortTable(col)` are the core functions.

## UI Conventions

- **handicap-tracker.html** palette: deep navy `#0a0e1a`, gold accent `#c6a84b`, teal success `#06b6d4`, red danger `#ef4444`
- **cognizant-classic-tracker.html** palette: dark green `#0b1a10`, gold `#c9a84c`
- Both use CSS custom properties (`--var-name`) at `:root` — update palette there
- No CSS framework; layout uses CSS Grid (`grid-template-columns`) and Flexbox
- Canvas API (no Chart.js) draws the handicap progress chart in `drawChart()` — re-call on window resize

## Git Workflow

Main branch is `main`, remote is `origin` (GitHub: `oakleymize-tech/golf-trackers`). Each file is independently deployable — changes to one don't affect the other.

**Commit and push after every meaningful unit of work** — a feature added, a bug fixed, a refactor completed. Never leave work uncommitted at the end of a session. The GitHub repo is the source of truth and must always reflect current state.

Commit message format:
- First line: short imperative summary (`Fix chart resize on mobile`, `Add stroke index field to round form`)
- Body (if needed): one or two lines explaining *why*, not *what*
- Always append the co-author trailer:
  ```
  Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
  ```

Push immediately after committing (`git push`). Do not batch multiple sessions of work into a single push.
