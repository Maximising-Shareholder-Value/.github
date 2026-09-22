# To-Do

This is the working checklist — the actual, specific next steps, broken
down small enough that each one is a real thing someone could sit down
and do, not a vague goal. If [ROADMAP.md](ROADMAP.md) is "what are we
building and why," this file is "okay, so what do I actually do Monday
morning." Items get a `[x]` and a short note on how/when they were
finished rather than being deleted once done — that note then usually
gets folded into [HISTORY.md](HISTORY.md) too, so nothing about a
finished item is ever fully lost, just moved to where it belongs.

## Pillar 1: multi-asset-class indicators

- [x] **Audit which real ticker searches currently return N/A-heavy
      pages — done 2026-09-19, against the live production site (real
      data, not assumptions).** Tested AAPL (stock), VOO (index fund
      ETF), TLT (bond ETF), GLD/USO (commodity trust ETFs), UNG (futures-
      based commodity ETF), URA (uranium miners ETF), BTC/ETH (crypto) —
      **every one of them renders clean, with zero N/A in any visible
      card.** The one N/A found (AAPL's Insider Transactions table, 2
      instances) is a real, individual SEC Form 4 filing missing a
      `transactionPrice` field on 2 specific rows — correct behavior, not
      a bug. **Conclusion: the 2026-09-13 ETF/crypto redesign already
      solved this pillar for every asset type that has real free-tier
      data behind it** — commodity ETFs (both trust-structured like GLD
      and futures-based like UNG) were already covered by the same
      `getInstrumentType()` logic without any extra work needed.
- [x] Also confirmed: searching a symbol with genuinely zero coverage
      (tested a real Treasury CUSIP) degrades gracefully — stays on the
      home view with a status message, no crash, no broken dashboard.
- [x] ~~Clarify what "CDCs" meant in the original ask~~ — dropped,
      2026-09-19, per Jozsua ("let's ignore CDCs for now").
- [x] **Researched whether one source covers both bonds and options —
      done 2026-09-19, see [API_RESEARCH.md](API_RESEARCH.md) for the
      full comparison (EODHD, Alpaca, Tradier, Polygon/Massive).
      Conclusion: no free/hobby-accessible source covers both** — EODHD
      has both but paywalled (~$100/mo bundle); Alpaca has both but
      bonds specifically requires a business-level "Broker API"
      partnership, not reachable by a solo project. Treat as two
      separate decisions:
      - **Options: Alpaca's Trading API** — genuinely free, self-serve
        (email signup, free paper account, no funding/approval), real
        options chains. This replaces FlashAlpha as the lead candidate.
      - **Bonds: still no free path anywhere.** "Browse via bond ETFs"
        stays the answer unless the project ever wants to pay ~$100/mo.
- [x] **Alpaca backend proxy — done and live, 2026-09-21.** Jozsua
      created a free paper-trading account and shared the API Key
      ID/Secret. Added as Cloudflare secrets on `msv-api`
      (`ALPACA_API_KEY_ID`/`ALPACA_API_SECRET_KEY`), plus a new
      `/api/alpaca` route (`proxyAlpaca()` in `worker.js`, header-based
      auth) proxying `data.alpaca.markets/v1beta1`. Deployed and tested
      live: a real AAPL options chain with real bid/ask came back
      through the proxy. See msv-api's `CLAUDE.md` for the implementation
      detail.
- [x] **Options UI — shipped (v1), 2026-09-21.** Jozsua chose the
      simplified "few strikes" view explicitly, and asked for it kept
      light since he's not deeply familiar with options yet. Built: a new
      "Options" card on the ticker page (stocks + ETFs, hidden for
      crypto), showing the nearest expiration only and the ~7 strikes
      closest to the current price, bid/ask per call/put, the
      at-the-money row highlighted. A `(?)` tooltip explains calls/puts/
      strike/bid-ask in plain English (`options` key in `definitions.js`).
      Tested against real live data (real AAPL contracts, real bid/ask).
      **Deliberately left for later:** Greeks, implied volatility, more
      strikes/expirations — see [ROADMAP.md](ROADMAP.md)'s pillar
      breakdown for the "next up" note.

## ETF/index fund page enhancements (scoped 2026-09-19, partially shipped 2026-09-21)

**Shipped, no new data source needed:**
- [x] **Fund name + issuer** — a hand-curated lookup (`ETF_FUND_INFO` in
      `script.js`, ~50 tickers matching the home page's browse
      categories) supplies the real fund name (e.g. "Vanguard S&P 500
      ETF") and issuer (e.g. "Vanguard") that Finnhub can't provide for
      ETFs. Shows in the page title and a new "Fund Issuer" fact.
- [x] **Real "About" description for ETFs** — reuses the existing
      Wikipedia lookup, now falling back to describing the fund's
      *issuer* when the specific fund has no dedicated Wikipedia article
      of its own (confirmed most don't — SPY/QQQ/GLD do, VOO/IWM/ARKK
      don't), clearly disclosed as "about the issuer" rather than passed
      off as being about the fund itself.
- [x] **After-Hours price placeholder** — clearly labeled "sample," see
      [BLOCKERS.md](BLOCKERS.md) for why it can't be real yet.
- [x] Coverage limited to the curated list above — anything else
      searched still falls back to symbol-only display. Long-tail gap
      tracked in [BLOCKERS.md](BLOCKERS.md).

**PARKED, 2026-09-21** — confirmed dead end on every free tier (Jozsua
provided a real FMP key; tested live against the exact endpoints needed.
All three — ETF Holdings, ETF Info, ETF Sector Weighting — return
`HTTP 402`, gated to FMP's **Ultimate** (top) tier. Twelve Data's
`/statistics`, tested through the existing live proxy: same story,
paid-plan-only). See [BLOCKERS.md](BLOCKERS.md) for the full research
trail. **Jozsua's explicit call: park it rather than pay for it** — not
being actively pursued. Revisit only if a paid plan is deliberately
chosen later, not as a default "keep checking" background task.
- [ ] *(parked)* "About the fund" overview section (category, AUM, NAV,
      expense ratio, yield, legal type, YTD total return).
- [ ] *(parked)* Holdings + sector weighting section.
- [ ] Bid/ask and today's live volume have no free source anywhere, for
      any instrument type, confirmed — not solvable by a new
      ETF-specific vendor, this isn't an ETF-page problem specifically.
      Also effectively parked alongside the above (same root cause: no
      free data exists).

## Stock fundamentals: additional indicators — shipped 2026-09-22

Prompted by Jozsua asking "what other information can we put as part of
the multi asset indicator? can we put more information around risk,
performance, fund information, AUM, etc.?" — audited Finnhub's
`/stock/metric?metric=all` live (133 fields returned for a real stock)
against what `script.js` actually uses (31 fields, found via
`grep -oE "metric\.[a-zA-Z0-9/&]+..."`). **Stocks only** — the ETF metric
response was already confirmed exhausted in the 2026-09-19/21 ETF audit
above, so none of this applies there. Jozsua approved building all of it,
2026-09-22 ("can the risk and efficiency cards be built now? if yes,
please go ahead, as well as the rest") — shipped same day, msv-web PR #19.

- [x] **New "Risk" card** — Interest Coverage Ratio
      (`netInterestCoverageAnnual`), Long-Term Debt/Equity
      (`longTermDebt/equityAnnual` — literal `/` in the field name, needs
      bracket access), Dividend Payout Ratio (`payoutRatioAnnual`).
- [x] **New "Efficiency" card** — Asset Turnover (`assetTurnoverTTM`),
      Inventory Turnover (`inventoryTurnoverTTM`), Receivables Turnover
      (`receivablesTurnoverTTM`).
- [x] **Profitability card, extended** — added ROA (`roaTTM`, alongside
      existing ROE), ROI (`roiTTM`, not shown at all before), and 5-year
      margin trend (`netProfitMargin5Y`/`operatingMargin5Y`/
      `grossMargin5Y`).
- [x] **Growth card, extended** — added quarterly YoY growth
      (`epsGrowthQuarterlyYoy`, `revenueGrowthQuarterlyYoy`) alongside the
      existing annual growth figures.
- [x] **Valuation card, extended** — added Price-to-Sales (`psTTM`) and
      Price-to-Cash-Flow (`pcfShareTTM` — the operating-cash-flow-based
      variant, not the free-cash-flow `pfcfShareTTM` alternative).
- [x] New `definitions.js` tooltip entry for every new indicator (10
      total), matching the existing `{what, formula, high, low, sector}`
      shape.
- [x] Risk and Efficiency sections added to `NON_STOCK_HIDDEN_SECTIONS` —
      confirmed live (mocked ETF instrument type) that both correctly
      hide, same as Growth/Profitability.
- Verified against real field values captured live from the production
  Finnhub proxy (not fabricated), across all 6 affected sections, before
  shipping — see msv-web PR #19 for the full test trail.

## Embedded side-by-side comparison (scoped 2026-09-19)

A Compare feature already exists as its own page (`compare.js`) — this
is a **different** ask: a comparison section embedded directly on the
individual stock/ETF deep-dive page itself, showing similar assets in
the same class/industry/sector side by side without leaving the page.
Needs a design decision (which "similar assets" get picked
automatically — same-sector peers via Finnhub's `/stock/peers` for
stocks? same `BROWSE_CATEGORIES` bucket for ETFs?) before building.

## Pillar 5: explain-the-concept education layer

Scoped with Jozsua 2026-09-22: a dedicated **Learn hub** (not tooltip-
style — tooltips already exist per-indicator via `definitions.js` and
answer a narrower question). Target audience per Jozsua's own framing:
"people like me who want a load of nicely visualised information about
everything and all aspects before making an informed decision — this
information I'm providing them is the 'informed' part." Explicitly
flagged as a large content-generation task and agreed to ship in stages
— one category built and reviewed at a time, not one giant batch.

Five categories scoped (msv-web's `learn.js`, `LEARN_CATEGORIES`):
- [x] **The Basics — shipped 2026-09-22, msv-web PR #20.** Stocks, ETFs,
      Bond ETFs (bonds' stand-in, see the ETF section above), Crypto.
      Each topic: a plain-English explanation, a simple inline SVG/CSS
      visual (pie slice, basket, lend/borrow arrows, centralized-vs-
      decentralized network — no images, theme-colored so both light/dark
      work), a grounded real-ticker example, and a "why it matters here"
      callout tying back to the rest of the dashboard. Verified live
      (Playwright) in dark theme, light theme, and mobile width.
- [x] **Reading the Numbers — shipped 2026-09-22, msv-web PR #21.** 5
      topics: Valuation, Growth, Profitability & Efficiency, Financial
      Health & Risk, Dividends. Each explains what the matching ticker-
      page card actually measures (ties directly into the cards shipped
      in the "Stock fundamentals" section above), with its own visual —
      a two-bar comparison chart (reused for Valuation and Financial
      Health), an ascending bar chart (Growth), a shrinking funnel
      (margins), a turnover cycle diagram (Efficiency), and a growing
      quarterly-payment timeline (Dividends).
- [ ] **Macro & the Economy** — what a rate hike, inflation, or GDP growth
      actually does, and why it hits sectors differently (ties into the
      Macro dashboard/world map).
- [ ] **Options 101** — calls, puts, strikes, expiration, bid/ask spread
      (ties into the Options card; Jozsua flagged he's not familiar with
      options himself).
- [ ] **Putting It Together** — how to actually weigh all of the above:
      diversification, risk tolerance, reading the AI Outlook. Arguably
      the category that most directly answers Jozsua's stated goal for
      this whole pillar — don't skip it in favor of only the numbers-
      heavy categories.
- [ ] All four remaining categories currently render as "Coming soon"
      cards in the live UI (`LEARN_CATEGORIES` entries with an empty
      `topics` array) — the full intended shape of the hub is visible to
      users now, not a surprise addition later.

## Pillar 2+3: supply chain visualization pilot

- [ ] Pick the pilot theme (AI infrastructure is the natural first choice
      — it's Jozsua's own recurring research interest per the attached
      Claude conversations from 2026-09-19).
- [ ] Curate 5-10 companies and their real upstream/downstream
      relationships — this is a research task, likely LLM-assisted but
      needs human verification per company, not a live feed.
- [ ] Decide the data format (static JSON shipped with the frontend is
      the simplest starting point — no new backend needed).
- [ ] Design the actual visualization (this is the "sexy visual" ask —
      worth a dedicated design pass once the data shape is known, not
      before).

## Pillar 4: multi-country macro dashboard

- [x] **World Bank backend proxy — done and live, 2026-09-21.** Jozsua
      chose World Bank directly (broadest country coverage) rather than
      starting with OECD/DBnomics as originally suggested. No key/signup
      needed at all — confirmed it has zero CORS support of its own
      (same as FRED), so it's proxied via a new `/api/worldbank` route on
      `msv-api` (reuses the existing generic `proxy()` function). Tested
      live: real Singapore CPI inflation and real Indonesia GDP growth
      both came back correctly.
- [x] **Default countries decided, 2026-09-21:** Jozsua picked "Major
      global economies" — US, China, Germany (standing in for the EU),
      Japan, UK — over featuring his own footprint (Singapore/Indonesia/
      Australia) or both. A full country picker still covers everywhere
      else; these five are just what's shown before anyone searches.
- [x] **Macro dashboard UI — shipped (v1), 2026-09-21.** A country picker
      in the Macro tab: US stays on FRED (unchanged, monthly data);
      China/Germany/Japan/UK are backed by the already-live World Bank
      proxy. Indicators: GDP growth, inflation, unemployment, current
      account balance — a rate/yield indicator was tried first
      (matching the US tab), but World Bank's interest-rate series come
      back null for these advanced economies in recent years (confirmed
      live), so it was swapped for current account balance, which has
      real 2025 data for all 5 countries. Verified against real World
      Bank data for China/Germany/Japan (not just mocked) before
      shipping. **Deliberately not built yet:** a 2+ country comparison
      view, and a picker for countries outside the default 5 — both
      fast-follows, not blocking this first pass.
- [x] **World map tied to the macro dashboard — shipped, 2026-09-21.**
      Hovering a tracked country's real landmass on the Global Markets
      map now shows its index price plus a live World Bank macro
      snapshot (same indicators as the Macro tab above), so the map and
      the macro dashboard aren't two disconnected features. Removed the
      permanent price/% text under each map label in the process (now
      redundant with the sidebar list, which also gained country flags).
      Covers all 13 map countries, not just the 5 Macro-tab defaults —
      World Bank data is free/unlimited, so there was no reason to limit
      it to 5.
- [ ] OECD SDMX/DBnomics (callable directly from the browser, no proxy
      needed) remain an option to add later for countries/indicators
      World Bank doesn't cover well — not blocking, since World Bank
      alone already covers virtually every country.

## Pillar 6: AI research companion — validation step only, for now

- [ ] Ship a small set of **static** curated research threads (same
      shape as the two conversations that inspired this) as a taste test,
      before building any live LLM integration.
- [ ] Only after that: decide a cost model (who pays per query, any usage
      caps) before wiring up real API calls.

## Home page enhancements (not tied to a big pillar, but real asks)

- [ ] **Recently Viewed as a sidebar column.** Currently a horizontal row
      above the tabs (`renderRecentlyViewed()` in `home.js`) — Jozsua
      wants this reworked into a left- or right-hand column, with the
      ability to organize/categorize the entries rather than just a flat
      recency list (2026-09-19 ask). Needs a design decision on what
      "organize/categorize" means concretely (manual tags? auto-grouped
      by asset type? a watchlist rather than just recently-viewed?)
      before building — worth a quick check-in rather than guessing.
- [ ] **Compare, extended.** A Compare feature already exists
      (`compare.js`, up to 4 tickers side by side, reuses the same
      sector/traffic-light logic as the deep-dive page) — Jozsua's
      2026-09-19 ask adds "underlying assets" to what gets compared,
      which the current version doesn't do (e.g. an ETF/index fund's
      actual holdings/composition, not just its price stats). Needs
      research into whether Finnhub's free tier (or any source in
      [API_RESEARCH.md](API_RESEARCH.md)) exposes fund holdings at all
      before promising this.

## Homepage feature recommendations (brainstormed 2026-09-19, not yet chosen)

Asked for "an extensive range of recommendations" to consider adding —
these are options, not commitments. Roughly ordered by how cheap/easy
each would be given the app's existing zero-cost architecture:

- **Upcoming earnings calendar strip** — Finnhub's `/calendar/earnings`
  is already used per-ticker (deep-dive page); a homepage-wide version
  ("who reports this week") is the same endpoint, no new data source.
- **Economic calendar** (next Fed meeting, next CPI/jobs report date) —
  pairs naturally with the Macro tab; FRED doesn't provide calendar
  dates directly, would need a small curated/hand-maintained list rather
  than a live feed (dates are known well in advance, low maintenance).
- **A real watchlist**, separate from Recently Viewed — user manually
  adds/removes tickers, stored in `localStorage` (same zero-backend
  pattern already used for theme and recently-viewed). Natural pairing
  with the "Recently Viewed as a sidebar column" item above.
- **Sector performance heatmap** (not per-stock — per-sector, e.g. using
  the `XL*` sector ETFs already in the ETFs browse category) — cheap,
  reuses tickers already fetched or easily added.
- **"Did you know" rotating fact** tied to Pillar 5 (the education
  layer) — a small, free way to surface bite-sized learning content on
  every visit once that content exists.
- **Currency/FX strip** (USD/SGD, USD/AUD, etc.) — Twelve Data supports
  forex; would need to confirm free-tier forex coverage before building
  (Finnhub's free tier explicitly does NOT cover forex — see the root
  `CLAUDE.md` — so this would lean on Twelve Data or a new source).
- **Trending/most-searched tickers this week** — needs some form of
  shared counter across visitors, which the current architecture doesn't
  have (everything today is per-browser, no shared backend state) — the
  one item here that's a real architecture addition, not just more UI.

## Standalone items

- [x] CI: JS syntax check + Playwright smoke test on every PR — done
      2026-09-19, see [HISTORY.md](HISTORY.md).
- [x] Governance docs — done 2026-09-19 (originally as `msv-web/.github/`,
      **moved to this org-wide `.github` repo on 2026-09-21** so
      project-wide planning isn't siloed inside just one of the two code
      repos — see HISTORY.md's latest phase).
- [x] Home page: more explanatory copy + a "how to use" popup (paginated
      modal, not inline) — done 2026-09-19.
- [x] Home page: world map / ticker strip switched to placeholder data —
      done 2026-09-19, see the tradeoff note in [ROADMAP.md](ROADMAP.md).
- [x] Browse categories bumped ~50% more tickers each (12 → 18 per
      category) — done 2026-09-19.
- [x] "What's New" in-app popup (`changelog.js`, 🔔 in header) — done
      2026-09-21, see HISTORY.md Phase 10. **Keep this updated going
      forward, same discipline as HISTORY.md/TODO.md** — add a new dated
      entry to `CHANGELOG` in `changelog.js` whenever a real
      user-visible change ships.
- [x] Intro copy rewritten in a more professional/editorial tone — done
      2026-09-21.
- [x] Global Markets map/sidebar trimmed to countries-only (20 → 13
      tickers) — done 2026-09-21.
- [x] New "Commodities" browse category (18 tickers) — done 2026-09-21.
- [x] Governance doc intros simplified/elaborated — done 2026-09-21.
- [x] "What's New" badge swapped from an auto-popup to a quiet red dot
      on the bell, cleared on click — done 2026-09-21 (the auto-popup
      was more intrusive than intended).
- [x] **Six-pillars breakdown — done 2026-09-21**, see the "Pillar
      breakdown" section in [ROADMAP.md](ROADMAP.md) for the full detail
      (sub-parts, status, effort, open decisions per pillar).
- [ ] **Still open: which pillar(s) to actually start building, and at
      what pace.** The breakdown above is what's needed to make that
      call — waiting on Jozsua's decision, not yet started.
- [ ] Quagmire hub page link to $MSV — checked 2026-09-19, currently
      live and correctly pointing at
      `https://msv-web.jozsua-heng.workers.dev/` (verified via a real
      HTTP request, page title, and `wrangler deployments list`). Need
      Jozsua to clarify what "the new link on Cloudflare" refers to
      before changing anything — possibly a custom domain not set up
      yet, or a different Cloudflare account than the one currently
      deployed to.
- [ ] Custom domain for $MSV (optional, cosmetic — from the original
      "Moving forward with $MSV" list, not urgent).
- [ ] Cloudflare Workers Builds email notifications — Jozsua decided
      2026-09-21 to limit these to failures only (was getting one per
      PR, mostly "succeeded" noise). Can't be changed via the API token
      available here (confirmed — got a 403 on the Notifications
      endpoint; the wrangler OAuth token doesn't have that scope), so
      this needs Jozsua to do it himself in the Cloudflare dashboard
      (Notifications → the Workers Builds policy for msv-web/msv-api →
      select "Build failed" only). Not yet confirmed done.
