# Sourcing Call Dashboard

## Purpose
Static, client-side analytics dashboard for reviewing sourcing team call quality at Cityflo. Visualizes per-member performance (Abhishekh, Ashley, Bhumika, Sneha), grade distributions, interest levels, checklist hit-rates, and individual call insights scored against parameters P1-P*.

## Tech Stack
- **Vanilla HTML/CSS/JS** — single-file app (`index.html`, ~49 KB, ~1500 lines).
- **Chart.js 4.4.7** via jsDelivr CDN for visualizations.
- **Google Fonts**: Plus Jakarta Sans + JetBrains Mono.
- No build step, no package.json, no framework, no bundler.
- Deployed as **GitHub Pages** via `.github/workflows/deploy.yml` on push to `main`.

## Data Format
Single file: `data/dashboard_data.json` (~1.6 MB, pre-generated externally — this repo does NOT contain the generator).

Top-level structure:
```
meta: { generated_at, total_calls, members[], grade_boundaries[], parameters{P1,P2,...} }
calls: [ { call_sid, call_date, call_num, member, duration_sec, grade, grade_label, interest_level, ... } ]
individual_insights / common_gaps / checklist_items / interest_counts / hit_rate / grade_dist ...
```

Fetched at runtime: `fetch('data/dashboard_data.json')` at line ~546 in `index.html`. On failure, loader shows "Failed to load data".

## How to Update
1. Regenerate `data/dashboard_data.json` externally (generator lives elsewhere — ask team / check pipeline).
2. Drop the new JSON into `data/` replacing the old file.
3. `git add data/dashboard_data.json && git commit && git push origin main`.
4. GitHub Actions (`deploy.yml`) auto-deploys the entire repo root to Pages.

## How to Deploy
- Push to `main` → `actions/deploy-pages@v4` publishes the repo root.
- Local preview: `python3 -m http.server 8000` then open `http://localhost:8000` (must be served — opening `index.html` via `file://` breaks `fetch`).

## Gotchas
- **CORS / file://** — cannot be opened directly in browser; must use an HTTP server because of the `fetch` call.
- **JSON schema is implicit** — keys consumed by `index.html` must match generator output exactly. If adding a member or parameter, both the generator and any hardcoded references in `index.html` may need updates.
- **Single-file frontend** — all CSS/JS inlined in `index.html`. Search by feature keyword; no module boundaries. Edit carefully; there are no tests.
- **Data file is large** (~1.6 MB) and committed to git. Consider Git LFS if history grows; currently plain git.
- **Members list is hardcoded in data** (`meta.members`), not in HTML — frontend reads it dynamically.
- **No linting, no CI checks** beyond the Pages deploy. Breakage will only surface in the browser console.
- **Branch name is `main`** (not `master`) — deploy workflow keys off `main` only.
