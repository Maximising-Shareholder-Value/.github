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
- [x] **Macro & the Economy — shipped 2026-09-22, msv-web PR #22.** 4
      topics: Interest Rates, Inflation, GDP Growth, Unemployment — each
      ties back to the real Macro tab/world map indicators.
- [x] **Options 101 — shipped 2026-09-22, msv-web PR #22.** 4 topics:
      Calls and Puts, Strike Price & Expiration, Premium/Bid/Ask, Why
      People Use Options — kept deliberately basic and cautious per
      Jozsua's own note that he's not familiar with options himself; the
      last topic is explicit that this stops at vocabulary, not a full
      trading education.
- [x] **Putting It Together — shipped 2026-09-22, msv-web PR #22.** 4
      topics: Diversification, Risk Tolerance, Reading the AI Outlook,
      Red Flags. This is the category that most directly answers Jozsua's
      stated goal for the whole pillar (helping someone actually decide,
      not just define terms) — built, not skipped in favor of only the
      numbers-heavy categories.
- [x] **Pillar 5 complete.** All 5 scoped categories are live — no
      "Coming soon" cards remain in `LEARN_CATEGORIES`. 21 topics total
      across the hub (4 + 5 + 4 + 4 + 4).

## Pillar 2+3: supply chain visualization pilot

- [x] **Pilot theme picked: AI infrastructure** — Jozsua's own recurring
      research interest, confirmed 2026-09-22.
- [x] **Company relationships researched, 2026-09-22** — see
      [SUPPLY_CHAIN_RESEARCH.md](SUPPLY_CHAIN_RESEARCH.md) for the full
      sourced dataset, each cited to an SEC filing, official company
      statement, or corroborated journalism, with a confidence rating.
- [x] **Made live, 2026-09-22 — msv-web PR #26, new "Supply Chain" tab.**
      Jozsua's instruction was direct: "just make the supply-chain
      research data set live in the app please" — taken as his review/
      sign-off on the research itself, rather than waiting for a separate
      line-by-line check. The open OpenAI/Anthropic question was
      resolved by including them: shown as real nodes, clearly labeled
      "No ticker — private company", rather than cut from the pilot. All
      17 nodes and 20 relationships transcribed verbatim from the
      research doc (not re-paraphrased) — every relationship card in the
      UI shows its own source and confidence rating rather than stating
      anything as flat fact, and the one row the doc flagged as
      needing careful handling (NVIDIA's real SEC-disclosed customer
      concentration vs. the journalist-*inferred* identities behind
      "Customer A/B/C/D") keeps that exact distinction visible in its
      own highlighted note in the live UI.
- [ ] **Visualization design** — the tab currently shows nodes as a grid
      and relationships as a card list (functional, ships the real data
      now), not the "sexy visual" node-graph originally envisioned for
      this pillar. Worth a dedicated design pass later if Jozsua wants
      the fuller visual treatment — not blocking, since the real data is
      already live and usable.
- [ ] Consider a JSON/data-file split if this pilot grows beyond one
      static JS file (`supplyChain.js` currently holds the data inline —
      fine at this size, worth revisiting if more industries/pilots get
      added later).

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

## Homepage overhaul v2: Seeking-Alpha-inspired persistent nav (scoped 2026-09-23)

Jozsua came back a day after the first overhaul shipped and asked for a
second pass, explicitly modeled on Seeking Alpha's structure (screenshots
attached) — "learn and adopt their best practices," not a literal copy.
Key asks: bring back a sidebar (this time persistent across every page,
not just the homepage), logo at the top, sectioned by dividers, and a
much longer nav list. Full mapping agreed with Jozsua before starting:

**Maps to existing features** (icon + new sidebar slot only): Home,
Market News, Learn, ETFs, Crypto, Macro, Watchlist, Compare, Market
Intelligence (→ the Supply Chain tab).

**Resolved using existing content, repositioned**: Stock Analysis (→ the
Winners/Losers/Most Active + browse-category tabs, moved out of the old
horizontal tab row into this nav item), Market Data (→ the Global
Markets world map + index strip), Sectors (→ the Sector Performance
heatmap).

**Explore Products** — built for real (not a placeholder): a directory
panel listing every destination in the app, including the Coming Soon
ones below, clearly marked which is which.

**New, genuinely not built yet — real "Coming Soon" placeholders, not
fake-functional UI:**
- [ ] Create Free Account — no auth/accounts system exists at all yet.
      This is a real, substantial future project (user accounts,
      sessions, a backend), not a quick add.
- [ ] Log In — same, depends on the above.
- [ ] Portfolio Builder — depends on accounts existing (a portfolio
      needs to be tied to someone) — blocked on the above two.
- [ ] Portfolio Health Check — same dependency.
- [ ] Performance — a NEW asset-class performance comparison (stocks
      vs. bonds vs. commodities vs. crypto returns over time), distinct
      from the Sectors heatmap. Doesn't depend on accounts — could be
      built standalone whenever it's prioritized.

**Explicitly not copied from Seeking Alpha, and why**: their "Rating"
column is a proprietary quant score with real analytical infrastructure
behind it — inventing a fake rating just to visually match would break
this app's "no fabricated data" standard, so it's dropped rather than
faked. Same reasoning for their "Trending" stock list, which is powered
by real search analytics this app doesn't have — any "trending"-style
module here needs to be honestly labeled for what it actually is (e.g.
curation order), not implied to be a real trending signal.

## Homepage overhaul — shipped 2026-09-22 (supersedes the old "Recently Viewed as a sidebar column" item below)

The 2026-09-19 ask to move Recently Viewed into a sidebar column got a
full concrete scope on 2026-09-22, once Jozsua saw the Learn hub live
and asked where it actually sits (buried in the same tab row as Winners/
Losers/browse categories) and wanted a fuller overhaul, drawing on
patterns from Seeking Alpha/Yahoo Finance/Google Finance/Webull/
TradingView. See HISTORY.md for the full build log across all phases.
**All 5 planned items shipped same day**, across 3 PRs plus one same-day
bugfix:

- [x] **1. Sidebar shell + Learn banner — msv-web PR #23.** Collapsible
      left sidebar: quick ticker search, Recently Viewed (moved from its
      old horizontal row into a vertical list), quick links to Learn/
      Compare/Macro. Learn promoted out of the tab row into its own
      banner card ("New to investing? Start here").
- [x] **Same-day fix — msv-web PR #24.** Jozsua flagged the collapsed
      sidebar "looked broken" — two real bugs found (the toggle button
      was being clipped by `overflow: hidden`, and the Quick Links icons
      were hidden entirely instead of staying visible as a proper icon
      rail). Both fixed; collapsed state is now a clean 3-icon rail.
- [x] **2. Watchlist — msv-web PR #25.** Real feature: a ☆/★ toggle on
      every ticker page, `localStorage`-backed (mirrors the existing
      Recently Viewed pattern exactly), sidebar list with remove (×) per
      entry. Zero extra API cost — name/symbol only, no live price.
- [x] **3. "Did you know" rotating tip — msv-web PR #25.** One Learn
      topic per day (date-seeded, stable all day), "Read more" jumps to
      Learn with that exact topic pre-expanded. Zero API cost.
- [x] **4. Sector performance heatmap — msv-web PR #25.** All 11 SPDR
      Select Sector ETFs (not just the 7 already in the ETFs browse
      category — needed the complete set), staggered quote calls (same
      40ms-apart pattern as the ranking-tab loader), cached per session.
- [x] **5. Economic calendar strip — msv-web PR #25.** Hand-maintained,
      real sourced dates — FOMC meetings from federalreserve.gov's
      published 2026 schedule, CPI release dates from bls.gov. Paired
      with a new **Earnings This Week** module (not originally scoped,
      added because it was a natural complement): one Finnhub
      `/calendar/earnings` call for the whole week, filtered to symbols
      this app already has a real name for.
- [x] Sidebar collapse design decision resolved: icon rail (Quick Links
      only — Quick Search/Watchlist/Recently Viewed need real text to be
      useful, so they hide when collapsed), state remembered via
      `localStorage`.
- [x] **Sidebar removed entirely, 2026-09-22 — msv-web PR #26.** Jozsua's
      call after seeing it live: "it looks so bad." Watchlist and
      Recently Viewed (real functionality) moved into the main grid as
      their own cards; Quick Search and Quick Links dropped rather than
      relocated, since they duplicated the header search box, the Learn
      banner, the Compare button, and the Macro tab. The homepage is now
      one flowing column again, just with a much fuller grid than before
      the overhaul started.

## Home page enhancements (not tied to a big pillar, but real asks)

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
