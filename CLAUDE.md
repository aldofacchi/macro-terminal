# SH Macro Terminal — CPG Student Housing Intelligence

## Project Overview

Single-page Bloomberg-style macro terminal for student housing investment analysis. Built for Centurion Property Group (CPG). Tracks 147 FRED series, real-time market prices (Yahoo Finance), BLS employment data, Census demographics, IPEDS/Scorecard university data, Zillow rents, and Finnhub economic calendar — all rendered client-side with zero backend.

## Architecture

- **Single file**: `index.html` (~4300 lines) — all HTML, CSS, and JS in one file
- **No build step**: Open in browser or serve with any static HTTP server
- **No dependencies**: Zero npm packages, no frameworks — vanilla JS only
- **Data flow**: Browser → CORS proxies → FRED/BLS/Census/Yahoo/etc → localStorage cache → render

## Dev Server

```bash
# Start local server (configured in .claude/launch.json)
python3 -m http.server 8080 --bind 127.0.0.1

# Or use the Claude Preview tool:
# preview_start with name "cpg-terminal"
```

The server runs at `http://localhost:8080`. The app requires API keys stored in `localStorage` — enter them through the setup modal on first load.

## API Keys Required (all free)

| Key | Source | Used For |
|-----|--------|----------|
| FRED | fred.stlouisfed.org | 147 macro/housing/rates series |
| NewsAPI | newsapi.org | Live RE & macro headlines |
| College Scorecard | collegescorecard.ed.gov | Graduate earnings, debt, STEM % |
| Finnhub | finnhub.io | Economic calendar, WebSocket streaming |
| BLS | data.bls.gov | Same-day employment/CPI releases |
| Census | api.census.gov | MSA demographics, renter %, age cohorts |

Keys are stored in `localStorage` only — never sent to any server other than the respective API.

## Code Structure (inside index.html)

```
Lines 1-320      CSS — all styles, responsive breakpoints, themes
Lines 322-476    HTML — setup modal, regime banner, topbar, tabs, panels, footer
Lines 478-738    Theme system — 10 color palettes, picker UI
Lines 739-757    Security — HTML sanitizer (esc/escUrl for XSS prevention)
Lines 758-927    FRED series config — 127 series with IDs, frequencies, limits
Lines 928-957    Static data — university profiles, market scores
Lines 958-1007   External API config — keys, endpoints, IPEDS/BLS/Census mappings
Lines 1008-1600  API fetchers — BLS, IPEDS, Scorecard, Zillow, Yahoo Finance
Lines 1600-1770  FedWatch computation — CME-style rate probability engine
Lines 1770-2420  Tab renderers — Realtime, News, Universities, MSA Markets
Lines 2420-2800  Core engine — state, cache, FRED fetch, auto-refresh, tabs
Lines 2800-2945  KPI tooltips — definitions for every tracked series
Lines 2945-3230  Scheduling — RT refresh, news, Finnhub WS, cleanup
Lines 3230-3395  Data helpers — val(), chg(), yoy(), spark(), formatting
Lines 3395-4215  Tab renderers — Regime, Overview, Rates, Liquidity, Growth,
                 Inflation, Housing, Student, Markets, Scenarios, Underwrite, Signals
```

## Key Functions

- `computeRegime()` — Growth/Inflation/Liquidity scoring → 4-state regime (Goldilocks/Reflation/Stagflation/Disinflation)
- `val(k)` — Get latest value for a series (prefers Yahoo RT over FRED when available)
- `fredFetch(k)` — Fetch from FRED with 4-proxy cascade + localStorage cache
- `fetchAll()` — Batch fetch all 127 FRED series in groups of 8
- `renderTab(name)` — Dispatch to appropriate tab renderer
- `esc(str)` / `escUrl(url)` — XSS sanitization for all external data

## CORS Proxy Cascade

FRED and several APIs don't support browser CORS. The app uses a fallback chain:
1. `api.allorigins.win` (wraps response in `{contents}`)
2. `corsproxy.io` (raw passthrough)
3. `api.codetabs.com` (raw passthrough)
4. `thingproxy.freeboard.io` (raw passthrough)

If all fail, stale localStorage cache is used.

## Refresh Intervals

- FRED data: 60 minutes
- Real-time prices (Yahoo): 2 min during market hours, 15 min after close
- News: 10 minutes
- Finnhub calendar: 12 hours
- Finnhub WebSocket: streaming (sub-second when connected)

## Tab Index

| Tab | ID | Content |
|-----|----|---------|
| 00 REGIME | regime | G/I/L framework, yield curve regime, FedWatch |
| 01 OVERVIEW | overview | Market pulse, calculated fields |
| 02 RATES & CREDIT | rates | Yield curve, credit spreads, CMBS |
| 03 LIQUIDITY | liquidity | Fed balance sheet, RRP, M2, CRE stress |
| 04 GROWTH & PMI | growth | PMI, NFP decomposition, intermarket |
| 05 INFLATION | inflation | CPI/PCE complex, shelter, pipeline |
| 06 HOUSING | housing | HPI, supply pipeline, Zillow ZORI |
| 07 STUDENT HOUSING | student | University profiles, WICHE cliff |
| 08 MARKETS | markets | Market rankings, cap rate spreads |
| 09 SCENARIOS | scenarios | Base/Bull/Bear with live stress test |
| 10 UNDERWRITING | underwrite | Live rates, debt yield, pre-lease |
| 11 SIGNALS | signals | 21-indicator composite signal |
| RT LIVE PRICES | realtime | Yahoo Finance RT, MOVE, calendar |
| 12 NEWS FEED | news | NewsAPI live headlines |
| 13 UNIVERSITIES | universities | IPEDS + Scorecard live data |
| 14 MSA MARKETS | markets2 | Census ACS demographics by MSA |

## Testing

No automated test suite. To verify manually:
1. Start dev server (`python3 -m http.server 8080`)
2. Open `http://localhost:8080`
3. Enter API keys in setup modal
4. Verify each tab renders without console errors
5. Check Data Health panel (⊞ DATA button) for series status

## Common Issues

- **CORS failures**: Proxies go down periodically. The cascade handles this, but if all 4 fail, data shows as stale/cached.
- **BLS rate limits**: Without API key, limited to 25 requests/day. With key, 500/day.
- **Census CORS**: Census API may block browser requests — falls back to allorigins proxy.
- **Yahoo Finance**: v7 batch endpoint deprecated. Using v8/chart per-symbol with proxy.
- **Zillow CSV**: Large CSV download (~20MB). Cached 24h in localStorage.
