# Known Blockers

Things that are **not buildable right now**, and exactly why — so this
never has to be re-explained or re-discovered. If you ask for one of
these again, the answer is here, not a re-investigation. Each entry
stays until the blocker is actually resolved (a new data source found,
a permission granted, etc.) — then it moves to
[HISTORY.md](HISTORY.md) as a resolved note, not deleted.

## Data blockers (no free source exists)

### Individual bonds (any real bond data at all)
**Blocked since:** 2026-09-19. **Confirmed across:** Finnhub, EODHD, Tradier, Polygon/Massive, Financial Modeling Prep.
No vendor offers free, self-serve, individual-bond data (price, yield,
maturity, CUSIP lookup) — the closest thing that exists is Alpaca's
Broker API, which requires a business partnership application, not
reachable by a solo project. **Current answer:** browse bond ETFs
instead (already built, see the "Bond ETFs" home page category) — this
is the permanent answer unless the project ever pays for EODHD's bond
bundle (~$100/month).

### Options Greeks / Implied Volatility
**Blocked since:** 2026-09-24, confirmed via a live request to msv-api's
`/api/alpaca` options-snapshots endpoint (a real AAPL options chain). The
response contains `dailyBar`, `latestQuote`, `latestTrade`, `minuteBar`,
`prevDailyBar` — no `greeks` field and no `impliedVolatility` field
anywhere. This requires Alpaca's paid OPRA/real-time options feed, not
available on the free "indicative" feed this app uses. **Current
answer:** none — a permanent free-tier blocker, same category as the
bonds/bid-ask/volume entries above. Everything else free in that same
response (last trade price, today's volume) was previously unused and
has since been surfaced in the Options card UI (msv-web PR, 2026-09-24) —
see [HISTORY.md](HISTORY.md).

### Bid / Ask price, on anything
**Blocked since:** project start (documented in the root `CLAUDE.md`
from early on). Finnhub's free `/quote` endpoint has never included
bid/ask, for any instrument type — stock, ETF, crypto, all the same.
Confirmed directly, not assumed. **No workaround found yet** — this
would need a different quote source entirely. Not researched further
as of 2026-09-21.

### Today's trading volume (not average volume — the live number)
**Blocked since:** 2026-09-21, surfaced while scoping the richer ETF
page. Finnhub's free `/quote` and `/stock/metric` don't return a
live/today's-volume field for any instrument type — confirmed via a
live request. 10-day and 3-month **average** volume ARE available and
already shown; today's actual volume is not.

### After-hours / pre-market ("overnight") price
**Blocked since:** project start (documented in root `CLAUDE.md`).
Neither Finnhub nor Twelve Data returns a premarket/extended-hours
price on their free tiers — confirmed directly (Twelve Data's
`extended_hours=true` param returns a byte-identical response with or
without it). **2026-09-21 decision:** show a clearly-labeled
placeholder for this instead of omitting it entirely, matching the
homepage world map's existing "Sample data" pattern — not real,
disclosed as such.

### ETF/fund "vital stats" — NAV, net assets/AUM, expense ratio, holdings, sector weighting
**🅿️ PARKED, 2026-09-21 — Jozsua's explicit decision, not being pursued.**
Blocked since 2026-09-19, confirmed a dead end on free tiers on
2026-09-21 (real FMP key provided, tested live) — see below for the
research trail. Rather than pay for a data plan to unlock this, Jozsua
chose to park it. **Don't pick this back up as a default next step** —
only revisit if a paid plan is deliberately chosen later.

- Finnhub's dedicated `/etf/*` endpoints (profile/holdings/sector/
  country) are premium-gated — confirmed live, all four return
  `"You don't have access to this resource."`. `/stock/metric` returns
  zero yield/NAV/AUM fields for ETFs.
- **Financial Modeling Prep, tested with a real key:** the key itself
  works fine (`/stable/quote`, `/stable/profile` both return real,
  current data). But `/stable/etf/holdings`, `/stable/etf/info`, and
  `/stable/etf/sector-weightings` all return
  `HTTP 402 "Restricted Endpoint... upgrade your plan"` — confirmed
  live, not guessed. Per FMP's own pricing page, ETF/mutual fund
  holdings specifically require their **Ultimate** tier — their top
  plan, not Starter or even Premium. Realistic cost is well past casual-
  upgrade territory (their Starter/Premium range alone runs
  $29–199/month; Ultimate is priced above that).
- **Twelve Data, tested through the existing live proxy** (same key
  already used for charts, so this cost nothing extra to check): the
  `/etfs` endpoint returns a real fund name but ISIN/CUSIP come back as
  `"request_access_via_add_ons"`, and the `/statistics` endpoint
  (fund-level stats) returns `HTTP 403`: *"available exclusively with
  pro or ultra or venture or enterprise plans."* Same story, different
  vendor.

**Conclusion: there is no free path to NAV/AUM/expense ratio/holdings/
sector weighting for ETFs, full stop — three vendors checked, three
paywalls, all confirmed by live requests, not assumed.** The only way
forward is a paid plan (FMP Ultimate or Twelve Data Pro+, cost not
precisely confirmed but clearly a real recurring expense, not a few
dollars) — a genuine build-vs-spend decision, not an engineering one.
**Until/unless that's decided, this specific data stays unavailable.**

**Re-confirmed 2026-09-30 with a brand new FMP key** (Jozsua provided a
fresh one): identical result — `/stable/etf/holdings` and `/stable/etf/
info` still return "Restricted Endpoint", `/stable/etf/sector-weighting`
returns an empty array even for SPY/QQQ. **Also newly tested and also
blocked: `/stable/company-screener`** (the Stock Screener endpoint) —
same "Restricted Endpoint" response, so the hoped-for broader-market
screener upgrade path (see TODO.md) is confirmed not available on free
either. Unlike the 2026-09-21 test, **this key WAS kept and IS in active
use** — `/stable/profile` (also confirmed working both times) turned out
genuinely useful for real ETF fund names/descriptions (see the resolved
item above), so this key is now stored as a Cloudflare secret
(`FMP_API_KEY`) on `msv-api`, not discarded. If a paid FMP plan is ever
chosen to unlock the items below, no new key exchange is needed.

### ETF/fund top holdings, sector weightings, portfolio composition
**Same blocker as above** — same FMP endpoints, same key, re-confirmed
still restricted 2026-09-30.

### Full official fund name + issuer, for any arbitrary searched ETF
**RESOLVED 2026-09-30 — a fresh FMP key was tested live and its
`/stable/profile` endpoint genuinely works** (unlike `/stock-screener`,
`/etf/holdings`, and `/etf/info`, which returned the same "Restricted
Endpoint" error as 2026-09-21's test — confirmed again on the same fresh
key, not assumed carried over). `/profile` returns a real fund name,
issuer-agnostic fund-specific description, website, ISIN/CUSIP, and beta
for ANY ticker — not just the curated list. Wired in as
`fetchFmpEtfProfile()` (msv-web's `script.js`), proxied via `msv-api`'s
new `/api/fmp` route, 24h-cached given FMP's tight 250/day free budget.
The old 2026-09-21 workaround (`ETF_FUND_INFO`, a ~60-ticker curated
lookup) stays in place as the fallback when FMP has no key configured or
fails for a given ticker — not removed, just no longer the only source.
**Still does NOT cover:** NAV, AUM, expense ratio, holdings, sector
weighting — see below, unchanged.

### Added 2026-09-27

- **ETF expense ratios, holdings and assets under management** — not
  available on any free source checked (Finnhub free `/etf/*` endpoints are
  paywalled). The ETFs page shows price, change and range only. This also
  means sector "representative companies" are a curated list, not the
  tracking ETF's real holdings.
- **CoinGecko community & developer stats** — the free plan returns
  `community_data: null` and an empty `developer_data` (verified live
  2026-09-27), so no GitHub-activity / social "project health" panel.
- **World Bank has no Taiwan data** — Taiwan appears on the map with
  market data only, no macro/governance figures.

## Standing watch-items (not blockers today, but a real constraint later)

### TradingView widget — free tier is non-commercial only
**Added:** 2026-09-30. The free "Advanced Real-Time Chart" widget
(`msv-web/tradingview.js`) is embedded as an optional chart source.
Confirmed directly from TradingView's own terms of service: free-widget
use is restricted to non-commercial sites — "we do not permit commercial
usage of any of our services or APIs [without] separate agreement." Fine
today (no subscriptions/ads on $MSV). **The moment $MSV starts charging
money or running ads, this needs either a paid TradingView agreement or
removal in favor of the in-house chart.js chart** (already built, zero
licensing risk). Revisit this specific item before any monetization
launch — don't let it slip through as a "we'll deal with it later."

## Permission / access blockers

### Cloudflare notification settings
**Blocked since:** 2026-09-21. The Cloudflare API token available in
this environment (a `wrangler` OAuth token) does not include the
Notifications permission scope — confirmed via a live API call, which
returned a 403 Authentication error on the Notifications endpoint.
**Not fixable from here** — changing notification policies (e.g.
limiting Workers Builds emails to failures only) needs Jozsua to do it
himself in the Cloudflare dashboard (Settings → Notifications). Exact
steps are in `TODO.md`.

## What "blocked" does NOT mean

A blocker here means **no free/reachable path exists today**, not
"nobody looked." Each entry above was investigated with live requests
before being recorded — see the linked history/research docs for the
actual evidence. If a new free API, a new permission, or a paid budget
ever changes the picture, update the relevant entry here (move it to
"resolved" in HISTORY.md) rather than leaving a stale blocker on record.
