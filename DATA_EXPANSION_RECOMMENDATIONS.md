# Data Expansion Recommendations

Jozsua asked (2026-09-24): "while we are on free plans for all these API
services, please give recommendations of how we can flood this app with
knowledge, information, and data points... I want this app to be
extremely detailed." This doc answers that directly — **what's
realistically free and unused today**, not a wishlist of paid upgrades.
Same standard as [API_RESEARCH.md](API_RESEARCH.md): claims here are
either confirmed by a live request, or explicitly flagged as an
unconfirmed candidate worth checking before building.

## 1. CoinGecko: crypto's missing historical price chart (highest-value single gap)

**Confirmed live, 2026-09-24** — `/coins/{id}/market_chart?vs_currency=usd&days=30`
needs no API key and returns real historical price data (tested against
Bitcoin: 721 hourly price points over 30 days, real prices/market
caps/volumes, all three arrays). Today, crypto tickers have **no
historical price chart at all** — `chart.js`'s own header comment
confirms Twelve Data can't do crypto candles either, so this is a
genuine, currently-unfilled gap, not a duplicate of anything that
already works. Building it would give crypto tickers the one major
capability stock/ETF tickers already have (the Price Chart card) that
they're currently missing entirely.

Also worth a look on the same free endpoint family: `community_data`
(Reddit/Twitter followers) and `developer_data` (GitHub commits/stars/
forks) on the existing `/coins/{id}` full-detail call already used for
the crypto deep-dive page — both already arrive in that response and are
currently discarded by `flattenCoinGeckoDetail()` in script.js. A
"Project Health" angle (development activity, community size) has no
real stock-market equivalent and would make the crypto page feel less
like a stripped-down stock page and more like its own thing.

## 2. World Bank breadth — already scoped and shipped

The macro/geopolitical breadth ask is already answered concretely by
Pillar 4's 2026-09-24 expansion (see [ROADMAP.md](ROADMAP.md) and
[TODO.md](TODO.md)) — 9 economic indicators plus 6 Worldwide Governance
Indicators, live-tested for all 5 default countries, plus an open
~217-country search and a 4-country comparison view. Not duplicated
here; this section exists just to point at that work as the concrete
answer to "more country/macro/geopolitical data."

## 3. Finnhub: needs a fresh live re-check with a real key

The 2026-09-22 audit (see [TODO.md](TODO.md), "Stock fundamentals:
additional indicators") found `/stock/metric?metric=all` returns 133
real fields for a live stock, against 31 actually read by `script.js` at
the time — and several of those gaps were since closed (Risk/Efficiency
cards, PR #19). **This session could not re-run that audit** — it
requires a real Finnhub API key, which lives only in the gitignored,
local-only `config.js` and isn't available in this environment. The
methodology, so it's independently re-runnable:

```
grep -oE "metric\.[a-zA-Z0-9/&]+" script.js home.js | sort -u
```
diffed against the live response's own key list
(`Object.keys(response.metric)`) — whatever's in the live response but
not in that grep output is an unused, already-free field. Worth doing
this again before assuming the current 133-vs-31 gap is still accurate.

## 4. Twelve Data: server-side technical indicators

Twelve Data offers dedicated technical-indicator endpoints (RSI, MACD,
Bollinger Bands, and others) computed server-side, as an alternative to
`chart.js`'s current client-side computation of RSI/MACD from raw
candles. **Not yet live-tested against the free tier's rate limits in
this pass** — flagged as a candidate, not confirmed. If the free tier
covers it without extra cost, the main upside is more indicators (e.g.
Bollinger Bands, Stochastic) without writing more client-side math, at
the cost of more API calls per chart load (currently just one `time_series`
call per range switch).

## 5. Twelve Data: forex, confirmed available

**Confirmed live, 2026-09-24** — `/quote?symbol=EUR/USD` on Twelve
Data's public demo key returns a real, current forex quote (price,
open/high/low, 52-week range, % change). This resolves what used to be
an open question in TODO.md's homepage-brainstorm list ("would need to
confirm free-tier forex coverage before building") — it's confirmed
available. Finnhub's free tier still has zero forex coverage (unchanged,
documented in the root `CLAUDE.md`). A real candidate for a Forex
section — a placeholder tile already exists in Explore Products'
"Market Outlook" category reflecting this as a live-but-unbuilt gap, not
a dead end.

## 6. FRED: candidate additions beyond the current 8 series

The US Macro tab's `MACRO_SERIES` (home.js) currently covers Fed Funds
Rate, CPI inflation, unemployment, 10-year Treasury yield, 30-year
mortgage rate, M2 money supply, consumer sentiment, and WTI crude.
Standard, well-known FRED series codes not yet included (candidates,
**not live-tested this pass** — no FRED key available in this
environment either, same limitation as Finnhub above):

- `T10Y2Y` — 10-Year minus 2-Year Treasury yield spread (the classic
  "yield curve inversion" recession signal).
- `ICSA` — Initial jobless claims (weekly, a leading labor-market
  indicator).
- `HOUST` — Housing starts.
- `INDPRO` — Industrial production index.

All four are standard, long-published FRED series unlikely to have
changed codes, but should be confirmed with one live request each before
shipping, matching this app's own "confirmed, not assumed" standard.

## 7. Structural ideas (not a new data source, just using what exists better)

- **Cross-link Market Intelligence into the ticker page.** Right now the
  17-node/20-relationship map only lives on its own page — a ticker page
  for e.g. NVDA doesn't mention that it has 4 real, sourced relationships
  in `supplyChain.js`. A small "Also in Market Intelligence" card/link on
  the deep-dive page for any of the 17 nodes would surface existing data
  in a second, more contextual place, for zero new data cost.
- **Tooltip-coverage audit.** `definitions.js` explains most indicators
  via the `(?)` tooltip pattern — worth a pass checking every visible
  metric label actually has a matching tooltip entry, especially after
  the 2026-09-22 Risk/Efficiency additions and the 2026-09-24 options
  activity/volume columns.
- **Learn hub content audit.** A structured check of what topics
  `learn.js` already covers vs. the concepts referenced elsewhere in the
  app (e.g. does "Governance" in the new Macro tab section have a Learn
  entry explaining what a WGI score means beyond the inline one-liner?).

## What this doc is not

Not a commitment list — everything here is "worth investigating/
building," not "definitely happening." Items already confirmed live are
marked as such; items that need a live check with a real API key this
session didn't have access to are marked as candidates, not facts. Move
an item to [TODO.md](TODO.md) with a real shipping date once it's
actually built, same as every other feature in this project.
