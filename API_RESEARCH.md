# Finance & Macro Data API Research

Researched 2026-09-19 for the roadmap's multi-asset-class (pillar 1) and
multi-country macro (pillar 4) work. Where possible, claims here are
**confirmed by a live request**, not just taken from a blog post — this
repo's own `CLAUDE.md` has a strong "confirmed directly, not assumed"
culture around free-tier claims (they change, and secondhand summaries
get them wrong often), so this doc keeps that standard.

## Shortlist: free APIs worth having in the toolbox (generous limits, ranked)

Jozsua asked to keep this list tight rather than adding vendors freely.
In order of how generous the free tier actually is:

1. **Finnhub** (already in use) — 60 calls/min (~86,400/day theoretical).
2. **Alpaca** (researched, not yet integrated) — free paper account,
   **1,000 calls/min** on the Basic market data plan. The most generous
   limit found in this whole research pass. Covers options; does NOT
   cover bonds (see the bonds/options note below).
3. **Twelve Data** (already in use) — 800/day, 8/min.
4. **OECD SDMX / DBnomics** (researched, not yet integrated) — free, no
   API key at all, no documented strict rate limit found, CORS-friendly
   (callable straight from the browser, no proxy needed).
5. **CoinGecko** (already in use) — public tier, no key needed, generous
   enough for this app's crypto usage.
6. **World Bank** (researched) — free, no key, but needs a msv-api proxy
   (no CORS).
7. **FRED** (already in use) — free, needs a key, no meaningful limit
   issue at this app's scale.
8. **Financial Modeling Prep (FMP)** — 250 calls/day. The **least**
   generous of the bunch by raw request count, but the only real lead
   for ETF/fund-specific data (see below) — and that kind of data
   changes slowly (daily at most), so 250/day can stretch far with
   caching, unlike live quotes.

## Currently in use

| Source | Used for | Free tier | CORS (browser-callable?) |
|---|---|---|---|
| Finnhub | Stocks, ETFs, quotes, recs, earnings, news | 60 calls/min | Yes (confirmed, already used directly in local dev) |
| Twelve Data | Price charts | 800/day, 8/min | Yes (confirmed, already used directly in local dev) |
| CoinGecko | Crypto | Public tier, no key needed for basic use | Yes (confirmed, already used directly in local dev) |
| FRED | US-only macro (rates, CPI, unemployment, etc.) | Free, needs a key | **No** — this is why it's the one API always proxied through msv-api, even in local dev |

## Candidates for multi-asset-class coverage (pillar 1)

| Source | What it adds | Free tier | Notes |
|---|---|---|---|
| **Financial Modeling Prep (FMP)** | Broad fundamentals + bonds/commodities reference data alongside prices | 250 calls/day, 500MB/30-day bandwidth cap | Broadest single alternative if Finnhub's free tier gap on bonds ever needs a second source rather than just substituting bond ETFs (current approach) |
| **EOD Historical Data (EODHD)** | Global exchange coverage, long historical data, has an options add-on | Free tier exists but is limited; options data is a paid add-on | Better fit as a future paid upgrade than a free-tier swap |
| **Alpha Vantage** | General market data | 5 calls/min, 15-min delay | Rate limit is too low to be a serious Finnhub replacement, not worth adopting for its own sake |
| **FlashAlpha** | **Options chains** (bid/ask, IV, Greeks, open interest) | Has a free tier (exact limits not confirmed — check before building on it) | The only *free* options-with-Greeks source found; everything else (Intrinio, Databento, EODHD's options add-on) is paid. Worth a spike before committing pillar 1's options coverage to it. |

**Reality check on bonds and options specifically:** neither has a clean
free live-data source. Individual bonds have zero coverage on Finnhub's
free tier (confirmed, documented in `CLAUDE.md` already) — the existing
"show bond ETFs instead" approach on the home page browse category is
still the pragmatic answer, not a gap to fix with a new API. Options are
similar: FlashAlpha is the only free option, unverified at production
scale. Recommendation: don't block pillar 1 on solving these two — ship
real indicators for commodities/index funds/bonds-as-ETFs first (all
achievable with existing Finnhub/Twelve Data access), and treat options
as a distinct, smaller follow-up once FlashAlpha's actual limits are
confirmed by a live test.

### Follow-up: is there ONE source for both bonds and options? (2026-09-19)

Asked specifically — researched four more candidates looking for a
single vendor covering both, to avoid juggling two separate
integrations. Short answer: **no free or self-serve-accessible source
covers both.** Every vendor that has both bundles them differently:

| Source | Bonds | Options | Verdict |
|---|---|---|---|
| **EODHD** | Real corporate + government bond data via ISIN/CUSIP | Via a paid marketplace add-on | Both exist, but **both are paid** — the "ALL-IN-ONE" bundle (EOD + Fundamentals + Calendar + Bonds) is $99.99/month, options is a separate paid add-on on top. Free tier (20 calls/day) includes neither. |
| **Alpaca** | Real US Treasury bills + 500+ corporate bonds | **Free** — confirmed live via their docs: full options trading + real-time/historical options data through the standard self-serve Trading API, enabled by default on a free paper account (just an email signup, no funding, no approval) | **Bonds require the separate "Broker API," which needs a business partnership/application — not reachable by a solo hobby project.** Options, however, is genuinely free and self-serve. |
| **Tradier** | Not offered at all | Real-time data requires a *funded* live brokerage account; sandbox gives delayed data only | Doesn't solve either half cleanly for a free/hobby setup |
| **Polygon.io (now branded Massive)** | **Not offered at all** — confirmed via their live pricing page (stocks, options, indices, currencies, futures — no bonds/fixed-income product exists) | Free tier exists for stocks (5 calls/min) but options-specific free tier wasn't disclosed on the pricing page | Options-focused only, no bonds story at any price |

**Practical recommendation:** treat bonds and options as two separate
decisions, not one:
- **Options** — **Alpaca's Trading API is the answer**, and it's
  actually better than the FlashAlpha lead from the first pass: fully
  free, self-serve (email signup, free paper account, no funding or
  approval needed), real-time and historical options chains. Update:
  supersedes the FlashAlpha spike as the next step if/when options
  coverage gets built.
- **Bonds** — still a genuine free-tier dead end everywhere checked.
  The only way to get real individual-bond data is to either pay EODHD
  ~$100/month (which would also happen to unlock options from the same
  vendor, undercutting the case for Alpaca) or pursue a business-level
  partnership (Alpaca Broker API) — neither fits a hobby project's
  zero-cost architecture. **"Browse via bond ETFs instead" remains the
  right call** unless real revenue ever justifies a paid data bill.

## Fund overview, holdings & sector weighting for ETFs (2026-09-19)

Jozsua wants a proper ETF/index-fund "about" section — category (e.g.
Large Blend), fund family, net assets/AUM, NAV, expense ratio, yield,
legal type — plus holdings and sector weighting. None of this is
fund-classification data; it's a genuinely different data domain from
price/return data.

**Confirmed live (2026-09-19) that Finnhub's ETF-specific endpoints are
premium-gated, not free:**

```
GET /etf/profile, /etf/holdings, /etf/sector, /etf/country
→ {"error":"You don't have access to this resource."}
```

on all four, tested against `VOO` through the real production proxy.
Also confirmed live that `/stock/metric` for an ETF (`VOO`, all metrics)
contains **zero** yield or today's-volume fields — every field currently
rendered for ETFs is already everything Finnhub's free tier has to give
on this front. So this isn't "we're leaving free data on the table" —
it's a genuine new-data-source problem.

| Source | What it claims to offer | Free tier | Confirmed? |
|---|---|---|---|
| **Financial Modeling Prep (FMP)** | Dedicated endpoints matching this exactly: ETF & Fund Holdings, ETF Sector Weighting, ETF Country Weighting, and an "ETF Information" endpoint with expense ratio + AUM | 250 calls/day, no credit card to sign up (email + password only) | **Not confirmed** — FMP's docs pages block automated fetching, and it's common for vendors to gate exactly this kind of specialized dataset behind a paid "Starter" tier even when basic quotes/profiles are free. Could not verify without an actual key. |
| **Twelve Data** (already in use) | Has a documented "ETF" product and a separate "Fundamentals" API mentioning fund family/category/expense ratio/AUM | 800/day, 8/min (same key already in use for charts) | **Not confirmed** — same issue, fundamentals-type data is commonly a paid-tier add-on even on APIs with an otherwise-generous free plan. |
| **EODHD** | Same ETF fundamentals glossary (fund family, category, holdings, sector weights) | Free tier is 20 calls/day, very likely excludes this | Already established as paywalled for this specific data (see the bonds/options section above) |

**What this means concretely:** there's a real, named candidate (FMP has
exactly the right endpoint shapes) but **the free-tier status is
unconfirmed** — the docs are unscrapable and secondhand sources don't
agree. This needs the same treatment as Alpaca did: **sign up for a free
FMP key (just an email, no card) and test the specific endpoints
live** before building anything on top of it. Same for Twelve Data's
fundamentals endpoints, since that key already exists — worth testing
before adding a new vendor at all.

## Candidates for multi-country macro (pillar 4)

FRED is Federal Reserve data — **US only** by definition. Two genuinely
free, multi-country alternatives were found and CORS-tested live today:

| Source | Coverage | Free tier | CORS | How to integrate |
|---|---|---|---|---|
| **World Bank Open Data API** | Any country, GDP/inflation/unemployment/20,000+ indicators | Free, **no API key at all** | **No** — confirmed via a live request from a `localhost` origin, no `Access-Control-Allow-Origin` header came back | Needs a new proxy route on msv-api, same pattern as FRED today |
| **OECD SDMX API** | ~38 OECD member countries, GDP/CPI/trade/labor | Free, **no API key at all** | **Yes** — confirmed live: the response echoed back `access-control-allow-origin: <the calling origin>` | Can be called **directly from the browser**, no backend proxy needed — cheaper to build than FRED was |
| **DBnomics** | Aggregates ECB, IMF, World Bank, OECD, Eurostat, FRED into one API | Free, no key needed for basic queries | **Yes** — confirmed live, same CORS-echo behavior as OECD | Worth considering as a single integration point instead of wiring up World Bank/OECD/ECB separately — trades a little flexibility for a lot less integration work |

**Recommendation when pillar 4 starts:** prototype against OECD SDMX or
DBnomics first specifically *because* they don't need the msv-api proxy
detour — that's a real complexity reduction versus how the FRED tab was
built. Fall back to World Bank (proxied) only for countries/indicators
OECD doesn't cover.

## Prediction markets & "who's holding/trading what" (researched 2026-10-01)

Jozsua asked for two things in one request: Polymarket/prediction-market
odds on the site, and a page showing what specific public figures hold
and trade — his examples were Michael Burry (a hedge fund manager, whose
holdings come from **13F filings**) and Nancy Pelosi (a member of
Congress, whose trades come from **STOCK Act disclosures**) — explicitly
compared to r/tradewithcongress. These are three genuinely different
data problems, researched separately below. Every claim here was
confirmed with a live request (`curl -I` with an `Origin` header for
CORS, or a real API call) on 2026-10-01, not taken from a blog post —
several blog/aggregator claims below turned out to be wrong or stale
when checked directly, which is exactly why this project checks live.

### Prediction markets: Polymarket — ready to build, no blockers found

**The strongest result of this whole research pass.** Polymarket's two
public APIs were both confirmed live, with zero authentication needed
for read-only market data:

| API | What it gives | Free tier | CORS | Rate limit (confirmed from official docs, not a blog) |
|---|---|---|---|---|
| **Gamma API** (`gamma-api.polymarket.com`) | Market questions, descriptions, end dates, liquidity, 24h volume | Free, no key | **Yes** — confirmed live, `access-control-allow-origin: *` | 300 req/10s (`/markets`), 500 req/10s (`/events`), 900 req/10s combined |
| **CLOB API** (`clob.polymarket.com`) | Live order book, prices, midpoints (the actual "probability" numbers) | Free, no key | **Yes** — confirmed live, `access-control-allow-origin: *` | 1,500 req/10s (`/price`, `/book`), 500 req/10s for the batch versions |

Both can be called **directly from the browser**, same tier as Finnhub/
Twelve Data/CoinGecko/FMP today — no new msv-api proxy route needed,
which makes this cheaper to integrate than FRED or World Bank were.
Confirmed real, finance-relevant markets exist with real volume, not
just novelty bets — e.g. live-pulled today: "Will the Fed increase
interest rates by 25 bps after the October 2026 meeting?" ($742k 24h
volume) sitting right next to "Will there be no change in Fed interest
rates after the October 2026 meeting?" — this slots naturally next to
the existing Macro (FRED) tab. Rate limits are IP-based via Cloudflare,
per the official docs, generous enough that this app's existing
throttle patterns (dataUtils.js) would comfortably stay well under them.

**Recommendation: build this one first** of the three — it's the only
one with zero open questions.

### Michael Burry / 13F institutional holdings — buildable, needs real backend work

13F is a quarterly SEC filing required of any investment manager with
over $100M in US equity assets — this is the *official*, free, primary
source for "what does [hedge fund manager] hold," confirmed live against
Michael Burry's actual firm, Scion Asset Management (CIK 0001649339):

- **`data.sec.gov/submissions/CIK##########.json`** — list of a
  manager's filings. **Free, no key, CORS-enabled** (confirmed:
  `access-control-allow-origin: *`) — callable directly from the
  browser. Scion's real filing history came back correctly, most recent
  13F-HR dated 2025-11-03.
- **The actual holdings table** lives in a separate `infotable.xml`
  inside each filing, under `www.sec.gov/Archives/edgar/data/...` — NOT
  the same host as above. Confirmed **no CORS header** on this one (it
  sits behind SEC's Akamai bot-protection, visible in the response's
  `_abck`/`bm_sz` cookies), so this half **needs an msv-api proxy route**,
  same pattern as FRED. SEC's fair-access policy requires a descriptive
  `User-Agent` header on every request (used `MSV research
  contact@example.com` for this test) and asks for roughly ≤10 req/sec.
- **Real data confirmed**: Scion's latest 13F genuinely lists Halliburton
  call options, Lululemon, Molina Healthcare, NVIDIA and more — exactly
  the kind of concentrated, well-known-for-contrarian-bets portfolio
  Burry is known for. Not a mock.
- **The catch: 13F identifies holdings by CUSIP, not ticker.** NVIDIA's
  real CUSIP from Burry's filing (`67066G104`) was tested against
  **OpenFIGI** (Bloomberg's free CUSIP→ticker mapping API) and correctly
  resolved to `NVDA`. OpenFIGI is free but **not CORS-enabled either**
  (confirmed: no `access-control-allow-origin` header) — second proxy
  route needed. Unauthenticated rate limit confirmed live at **25
  requests/minute** (`ratelimit-limit: 25`, `ratelimit-policy: 25;w=60`)
  — tight for a filing with 20-50+ positions; a free OpenFIGI signup
  raises this substantially ("map hundreds of thousands of instruments,"
  per their own docs) and costs nothing, same low-friction pattern as
  Twelve Data/CoinGecko/FMP's optional keys already in this app.

**What building this actually requires:** two new msv-api proxy routes
(SEC Archives + OpenFIGI), real XML parsing of the 13F infoTable format
(not JSON — more parsing work than any current integration), and a
curated list of which managers' CIKs to track (13F only covers managers
who chose to register that CIK publicly in a findable way — "search by
manager name" isn't a clean API, the CIK has to be looked up once,
similar diligence to how ETF tickers are hand-verified in this app).
**Feasible, but a real multi-piece build**, not a quick add — scope as
its own small project if greenlit, not bundled into a "low-hanging
fruit" pass.

### Nancy Pelosi / congressional trading (STOCK Act disclosures) — no good free API found

This is the weakest result of the three. The two sources that come up
first in every search (**House Stock Watcher**, **Senate Stock
Watcher** — historically the standard free/open answer for this) are
**both confirmed dead**: `housestockwatcher.com` and
`senatestockwatcher.com` (and `www.`/`api.` subdomain variants) all fail
DNS resolution entirely (`curl` error 6, "couldn't resolve host") — not
down temporarily, gone. Several 2026-dated blog posts still cite these
as live and working, which they are not — a direct example of why this
project doesn't trust secondhand summaries for data-source claims.

What was actually checked:

| Source | Verdict |
|---|---|
| **Quiver Quantitative** | Confirmed **no free API tier** — API pricing starts at $30/month (Hobbyist plan); the free tier is for their website dashboard only, a different product from the API. Ruled out. |
| **Disclosed Capitol** (disclosedcapitol.com) | Claims a free tier (signup + API key required) limited to the most recent 90 days of trades, vague/undocumented rate limits. Small, unestablished vendor — not independently verified beyond reading their own docs page, and no track record to judge reliability against. |
| **Lambda Finance** | Free tier exists but capped at 50 API calls/month per their own claim — too low to be useful for a live page. |
| **Capitol Trace** (capitoltrace.com) | Returned **HTTP 403 Forbidden** when checked directly — couldn't even confirm what it offers. |
| **Apify-hosted scrapers** (several) | Pay-per-use through Apify's platform, not free; also third-party scrapes of the official filings rather than a primary source. |

**The only genuinely authoritative, zero-cost source is the government
itself** — the House Clerk's disclosure portal
(`disclosures-clerk.house.gov`) and the Senate's eFD system
(`efdsearch.senate.gov`), both confirmed reachable live. Neither offers
a structured JSON API — they're built for a human filling out a search
form and reading a PDF/HTML filing, not for programmatic access. Scraping
either one is a real, separate research/build task (parsing fragile
HTML or PDFs, and the Senate system in particular is known for
session/agreement-gating that complicates straightforward scraping) —
not evaluated further here since it's beyond what "confirm a free data
source exists" was meant to answer.

**Recommendation: do not build this yet.** Either (a) wait and recheck
periodically in case a genuinely free, reliable, well-documented API
appears (this space turns over fast — two of the standard answers from
even a year ago are already dead), (b) revisit Disclosed Capitol once it
has more of a track record, or (c) if Jozsua wants this badly enough to
pay, Quiver Quantitative's $30/month Hobbyist API is the most
established paid option found. Not a "no," just a "not free yet."

**Not just Burry** — Jozsua clarified (2026-10-01) he used Burry as one
example, not the target: the real ask is a general "notable figures"
holdings/trades page. This doesn't change the research above — 13F
covers ANY manager who registered a CIK (Burry/Scion, Buffett/Berkshire
Hathaway, Ackman/Pershing Square, Wood/ARK Invest, etc. all file the
same way), so building it as "a curated list of tracked managers, same
mechanism per manager" rather than "a Burry page" was already the right
shape — just worth stating explicitly before anyone scopes it as a
one-person feature.

## Community/sentiment data: subreddits & similar (researched 2026-10-01)

Jozsua asked whether a "community page" summarizing subreddit discussion
(what people are saying, buy/sell chatter) is feasible. Checked two
realistic candidates — **verdict: don't build this right now, for two
different reasons.**

### Reddit — actively shutting down third-party API access, not just restrictive

This was already known to require OAuth and have tightened access (free
tier capped at ~100 req/min, non-commercial only, and Reddit's
"Responsible Builder Policy" — updated 2025-11-11 — closed self-service
app registration; every new OAuth client now needs manual approval with
a real chance of silent rejection). Confirmed directly: the old
"append `.json` to any reddit.com URL" trick, still cited in a lot of
tutorials, now returns a flat **403** — no anonymous access at all.

**But the decisive finding is newer than any of that**, reported
2026-09-30 (literally the day before this research) and confirmed
against the primary report, not a secondhand summary: **Reddit is
shutting down RSS feeds entirely on November 13, 2026, and closing
public API access altogether by March 2027** — existing approved
developer apps must re-register by January 12, 2027 just to avoid
losing access before the final cutoff. Reddit's own stated reasoning is
monetizing the same content through paid AI-licensing deals instead
($43M in "other revenue" in Q2 2026, +24% YoY) — the direction is
explicitly toward commercial-only access, not a free/hobby tier that's
merely inconvenient to get into. Building a new integration on this
right now means building on a platform with a published shutdown date a
few months out, independent of whatever Jozsua personally has to do to
get approved in the meantime.

### Stocktwits — a finance-native alternative, but also closed to new developers

Stocktwits (a Twitter-like feed specifically for stock/crypto chatter,
closer to what a $MSV "what are people saying" feature would actually
want than general Reddit) has a legacy public endpoint
(`api.stocktwits.com/api/2/streams/symbol/{TICKER}.json`) that still
returns real, live, unauthenticated messages — confirmed live against
AAPL. **But:** it is **not CORS-enabled** (would need an msv-api proxy),
and Stocktwits' own official developer portal states they are
**currently not accepting new API registrations** while reviewing their
whole API program, with no stated end date. The endpoint that works
today is informal/unsupported, not a sanctioned path for a new
integration — could be cut off without notice.

### Recommendation

**Don't build a live, automated community-sourced page right now** —
both realistic sources are either actively closing (Reddit) or already
closed to new developers (Stocktwits). If Jozsua wants "what people are
discussing" content without a live feed, the precedent already exists
in this app: the Crypto page's "Regulation & Adoption tracker" is
hand-curated with web-search-verified, dated facts, refreshed
periodically rather than pulled live — the same approach could cover
"what the community's saying" as an occasional research pass instead of
an automated feature. **Separately worth flagging:** "summarize them"
specifically would need either an LLM call from msv-api (a real
per-request cost, a different category of spend than every other free-
tier integration in this app) or Claude doing one-off manual
summarization during a session (not a live in-app feature) — worth
deciding which before this is revisited, independent of the data-source
blocker above.

## Sources

- [Best Free Stock Market APIs and Data Tools in 2026 (DEV Community)](https://dev.to/nexgendata/best-free-stock-market-apis-and-data-tools-in-2026-a-developers-honest-comparison-1926)
- [12 Must-Have Financial Market APIs for Real-Time Insights in 2026 (HackerNoon)](https://hackernoon.com/12-must-have-financial-market-apis-for-real-time-insights-in-2026)
- [Best Financial Data APIs in 2026 (nb-data)](https://www.nb-data.com/p/best-financial-data-apis-in-2026)
- [Options Chain API - Real-Time Greeks, IV, Open Interest (FlashAlpha)](https://flashalpha.com/articles/options-chain-api-real-time-greeks-open-interest)
- [Options Chain Data Providers: Free and Real-Time Sources (Oyamori)](https://oyamori.com/learning/options-chain-data-providers/)
- [FMP API Review: Pricing, Free Tier & Limits (2026) (Find My Moat)](https://www.findmymoat.com/tools/financial-modeling-prep-fmp)
- [5 Free APIs for Global Economic Data in 2026 (DEV Community)](https://dev.to/sotwdata/5-free-apis-for-global-economic-data-in-2026-no-api-key-needed-1ocf)
- [Indicator API Queries (World Bank Data Help Desk)](https://datahelpdesk.worldbank.org/knowledgebase/articles/898599-indicator-api-queries)
- [OECD SDMX API documentation](https://data.oecd.org/api/sdmx-ml-documentation/)
- CORS support for World Bank/OECD/DBnomics: confirmed directly via live `curl` requests with an `Origin` header on 2026-09-19, not taken from any of the above sources (none of them documented it clearly).
- Finnhub `/etf/*` endpoints being premium-gated, and `/stock/metric` having no yield/volume fields for ETFs: confirmed directly via live `curl` requests through the production msv-api proxy on 2026-09-19.
- [ETF & Fund Holdings API (FMP)](https://site.financialmodelingprep.com/developer/docs/stable/holdings)
- [ETF Sector Weighting API (FMP)](https://site.financialmodelingprep.com/developer/docs/stable/sector-weighting)
- [Do You Need a Credit Card for Financial Modeling Prep? (FMP)](https://site.financialmodelingprep.com/education/other/do-you-need-a-credit-card-to-use-financial-modeling-prep)
- [Twelve Data | ETF APIs](https://twelvedata.com/etf)
- [Twelve Data | Fundamental Data API](https://twelvedata.com/fundamentals)
- [Tradier Market Data docs](https://docs.tradier.com/docs/market-data)
- [Alpaca Fixed Income docs](https://docs.alpaca.markets/us/docs/fixed-income) — confirms Broker-API-only gating for bonds
- [Alpaca Options Trading docs](https://docs.alpaca.markets/us/docs/options-trading) — confirms free self-serve paper-account access
- [Alpaca expands fixed income to corporate bonds (Alpaca blog)](https://alpaca.markets/blog/alpaca-expands-fixed-income-offering-to-include-corporate-bonds/)
- [Massive (Polygon.io) pricing](https://massive.com/pricing) — confirms no bonds/fixed-income product exists
- [Polymarket API rate limits (official docs)](https://docs.polymarket.com/api-reference/rate-limits)
- Polymarket Gamma/CLOB CORS support and live market data: confirmed directly via `curl -I` with an `Origin` header and real market queries on 2026-10-01, not taken from any third-party guide.
- [SEC EDGAR company filings API (data.sec.gov)](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) — CORS and real filing data confirmed directly against Scion Asset Management's (Michael Burry) actual CIK on 2026-10-01.
- [OpenFIGI API](https://www.openfigi.com/api) — CUSIP-to-ticker mapping confirmed directly against a real CUSIP from Burry's own 13F filing; rate-limit headers confirmed live, not from docs alone.
- [Quiver Quantitative API pricing](https://www.quiverquant.com/premium-vs-api/) — confirms no free API tier, $30/month minimum.
- `housestockwatcher.com` / `senatestockwatcher.com` confirmed dead (DNS resolution failure) via direct `curl` on 2026-10-01, despite several 2026-dated blog posts citing them as live — a reminder that secondhand data-source claims in this space age out fast.
- [Reddit is killing RSS feeds and ending public API access because of AI bots (TechCrunch, 2026-09-30)](https://techcrunch.com/2026/09/30/reddit-is-killing-rss-feeds-ending-public-api-access-because-of-ai-bots) — primary report on the RSS (Nov 13, 2026) and public API (March 2027) shutdown dates.
- [Responsible Builder Policy (Reddit official)](https://support.reddithelp.com/hc/en-us/articles/42728983564564-Responsible-Builder-Policy) — referenced via the TechCrunch report and corroborating search results; the page itself returned 403 when fetched directly.
- `reddit.com/r/*.json` (the old no-auth trick) confirmed returning a flat 403 via direct `curl` on 2026-10-01 — OAuth is mandatory, no anonymous path exists anymore.
- [Stocktwits for Developers](https://api.stocktwits.com/developers) — states new API registrations are currently closed pending a program review, no end date given.
- Stocktwits' legacy public symbol-stream endpoint confirmed live (real AAPL messages) but not CORS-enabled, via a direct `curl` with an `Origin` header on 2026-10-01.
