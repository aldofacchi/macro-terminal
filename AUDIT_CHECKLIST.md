# Professional Audit Checklist — CPG Terminal

> Complete quality assurance checklist for CPG websites, dashboards, terminals, and apps.
> Run through before every major release or after significant changes.

---

## A. Code Quality

- [ ] Syntax validation — No unclosed tags, mismatched brackets, unterminated strings
- [ ] Strict mode violations — Undefined variables, redeclared constants, implicit globals
- [ ] Dead code — Unreachable branches, unused functions, orphaned event listeners
- [ ] Memory leaks — Intervals/timeouts that never clear, growing arrays without bounds, localStorage bloat
- [ ] Race conditions — Concurrent fetches overwriting each other, render called before data ready
- [ ] Error handling — Every `fetch()` has catch/timeout, every `JSON.parse()` is wrapped, every DOM query checks null
- [ ] XSS surface — Every external string goes through `esc()` before innerHTML, no raw URL injection
- [ ] API key exposure — Keys never in URLs that get logged, never in error messages shown to user
- [ ] No `var` declarations (use `const`/`let`)
- [ ] No `document.write()` usage
- [ ] Function names descriptive and consistent
- [ ] Git history clean (no secrets in past commits)

---

## B. Data Integrity

- [ ] Unit correctness — FRED units match display (thousands vs millions vs billions vs raw)
- [ ] Value plausibility — Gold ~$4,400, FFR ~3.5–5.5%, CPI ~2–4%, housing starts ~1,200–1,600K
- [ ] Stale data detection — Series dates checked against expectations (monthly series shouldn't be 3 months old)
- [ ] Cross-tab consistency — Same series shows same value on every tab that references it
- [ ] RT vs FRED alignment — When both sources exist, RT should be close to FRED (within normal daily movement)
- [ ] Null propagation — A null upstream value shouldn't produce NaN, `undefined`, or `null%` in display
- [ ] Calculation verification — Derived metrics (spreads, ratios, YOY changes) manually spot-checked against raw inputs
- [ ] Sort order correctness — Markets ranked by score, news sorted by date, FOMC meetings chronological
- [ ] Price splits handled (GLD ETF vs GC=F futures — different scales)
- [ ] Date formats consistent (MMM DD, YYYY or equivalent)

---

## C. Visual / UI

- [ ] Text readability — All text legible against background at every theme
- [ ] Color contrast — WCAG AA minimum (4.5:1 for body text, 3:1 for large text)
- [ ] Truncation — No text clipped mid-word, no values cut off by card boundaries
- [ ] Alignment — Grid columns line up, table cells consistent width, no jagged edges
- [ ] Overflow — No horizontal scroll on the page body, cards don't bleed outside containers
- [ ] Empty states — "Awaiting data" or "--" shown, never blank cards with no explanation
- [ ] Loading states — Progress feedback during fetch, skeleton or spinner where needed
- [ ] Sparkline quality — Lines visible, scaled to card width, no flat lines when data has variance
- [ ] Badge consistency — BULL/BEAR/NEUTRAL/CAUTION badges use consistent colors everywhere
- [ ] Tooltip behavior — All hover tooltips appear fully on screen, don't clip at edges
- [ ] Color coding consistent (green=positive, red=negative across all tabs)
- [ ] No layout shift on data load (skeleton or fixed-size containers)

---

## D. Interactivity

- [ ] Every button works — REFRESH, DATA, THEME, KEYS, EXPORT all trigger correct action
- [ ] Tab switching — Every tab renders on click, active state updates, no flicker
- [ ] Dropdown functionality — Compare tab selectors work, re-render on change
- [ ] Modal behavior — Opens/closes cleanly, X button always reachable, scrollable if content exceeds viewport
- [ ] Keyboard navigation — Tab/Enter works for buttons, Escape closes modals
- [ ] Link targets — External links open in new tab (`target="_blank"`), no broken URLs

---

## E. Responsive / Cross-Browser

- [ ] Desktop (1920px) — Full layout, all columns visible
- [ ] Laptop (1440px) — Tab bar scrollable, no cut-off content
- [ ] Tablet (768px) — Grids collapse to 2 columns, cards stack properly
- [ ] Mobile (375px) — Single column, touch targets 44px+, no horizontal scroll
- [ ] Print — Export produces clean output, no dark backgrounds, no hidden content that should show
- [ ] Tested in Chrome (primary)
- [ ] Tested in Safari
- [ ] Tested in Firefox
- [ ] No ES2022+ features without fallback (optional chaining, nullish coalescing OK)

---

## F. Performance

- [ ] Cold load time — First load with empty cache completes in reasonable time
- [ ] Warm load time — Cached data loads instantly, only stale series re-fetched
- [ ] DOM size — Not rendering all 17 tabs' HTML simultaneously if not needed
- [ ] localStorage size — Total usage under 5MB limit, old entries pruned
- [ ] Network waterfall — Batch sizes don't trigger rate limits, proxy cascade doesn't cascade unnecessarily
- [ ] Render jank — Tab switches feel instant, no visible layout shift
- [ ] No DOM thrashing in render loops (batch reads, then batch writes)
- [ ] `requestAnimationFrame()` used for high-frequency WebSocket updates

---

## G. API / Network

- [ ] Proxy cascade — Each proxy tested individually, fallback order optimal
- [ ] Key validation — Invalid/expired keys handled gracefully
- [ ] Rate limits — BLS 500/day, NewsAPI 100/day, FRED 120/min — verify we stay under
- [ ] CORS behavior — Direct calls work where possible, proxies only when needed
- [ ] Timeout handling — 8–10s timeouts on all fetches, no hanging requests
- [ ] Cache invalidation — Stale cache served when API fails, but refreshed when API recovers
- [ ] `Promise.allSettled()` for batch operations (not `Promise.all()`)
- [ ] Per-series status tracking (ok/cached/stale/error)
- [ ] Data Health panel or equivalent diagnostic UI
- [ ] All API endpoints use HTTPS
- [ ] API key setup modal present and functional

---

## H. Business Logic

- [ ] Regime computation — G/I/L scores map correctly to regimes, thresholds match documentation
- [ ] FedWatch math — Implied rates from ZQ futures match CME FedWatch within tolerance
- [ ] Scenario probabilities — Base + Bull + Bear = 100%
- [ ] Underwriting rates — Derived rates (Bridge = SOFR+300, Agency Perm = 10Y+175) match formulas
- [ ] Signal composite — 21 indicators correctly categorized, BULL/BEAR thresholds sensible
- [ ] Market scoring — Score 0–100 reflects actual market fundamentals, ranking order makes sense

---

## I. Security

- [ ] No secrets in source — API keys only in localStorage, never hardcoded
- [ ] No `eval()` — No dynamic code execution from external data
- [ ] CSP compliance — Inline scripts/styles acceptable for single-file app but noted
- [ ] HTTPS only — All API calls use HTTPS, no mixed content
- [ ] All external data passed through `esc()` before HTML insertion
- [ ] All URLs passed through `escUrl()` before use in `href`/`src`
- [ ] localStorage `setItem` wrapped in try/catch for quota errors
- [ ] Cache keys prefixed to avoid collisions between projects

---

## J. Documentation / Maintainability

- [ ] CLAUDE.md accuracy — Line numbers, function names, tab indices all match current code
- [ ] Comment accuracy — Comments describe what code actually does, not what it used to do
- [ ] Magic numbers — Thresholds and constants explained or named
- [ ] Line count documented in CLAUDE.md with section map
- [ ] `.claude/launch.json` configured for dev server

---

## K. Theming

- [ ] All themes render every tab without visual breakage
- [ ] CSS custom properties (variables) used — no hardcoded colors in JS
- [ ] Theme persisted in localStorage and restored on reload
- [ ] Theme picker accessible and functional
- [ ] Print stylesheet functional (`@media print`)

---

## L. Real-Time & WebSocket

- [ ] WebSocket connection has reconnect logic
- [ ] High-frequency updates throttled via `requestAnimationFrame()`
- [ ] Market hours detection (ET timezone, DST-aware, weekend check)
- [ ] Adaptive refresh intervals (faster during market hours, slower after close)
- [ ] WebSocket cleanup on page unload (`beforeunload` listener)
- [ ] Fallback to REST polling when WebSocket unavailable
