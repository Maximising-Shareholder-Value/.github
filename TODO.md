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
- [x] **Options UI expanded to the free-tier ceiling, 2026-09-24.** Jozsua
      asked to take Pillar 1 to 100% done. An expiration picker (pill
      row) now lets a visitor switch between every expiration the initial
      fetch already returned, instead of only the nearest one — no
      re-fetch, that data was already being thrown away. Default view
      stays the same light 9 strikes; a "Show all strikes" toggle reveals
      the full already-fetched ±15% band. A new "Show last trade & volume"
      toggle surfaces `latestTrade.p`/`dailyBar.v` — both already present
      in every Alpaca snapshot response and simply unused before this.
      Zero new network calls per page view. **Greeks/implied volatility
      confirmed genuinely blocked** (not just deferred) — see the new
      [BLOCKERS.md](BLOCKERS.md) entry, dated the same day. With that
      confirmed, Pillar 1 is now as complete as the free data tier
      allows — individual bonds remain the one other permanent gap.

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
- [x] **Pillar 4 taken to 100%, 2026-09-24** — Jozsua asked for the
      remaining gaps closed, plus a "more country-related, macro and
      geopolitical" data push. `/api/worldbank` was already confirmed a
      fully generic passthrough proxy (msv-api), so all of this was
      frontend-only, no backend deploy:
      - **Open country picker** — a debounced free-text search over
        World Bank's full ~217-country list (fetched once, cached),
        alongside the 5 "Featured" quick-picks (kept, not replaced).
      - **2+ country comparison view** — a toggle lets up to 4 countries
        (same cap as the Compare feature) be selected and shown side by
        side in a table, reusing the same World Bank fetch logic.
      - **Expanded indicator breadth** — the Macro tab (not the map
        popup, which stays small on purpose) now also shows GDP per
        Capita, Trade Balance, Government Debt, Total Reserves, and
        Population, live-tested against all 5 default countries before
        shipping (confirmed non-null for every one).
      - **Geopolitical dimension, genuinely new territory** — World
        Bank's Worldwide Governance Indicators (source database 3, not
        the default WDI database) added as a "Governance" sub-section:
        Voice & Accountability, Political Stability, Government
        Effectiveness, Regulatory Quality, Rule of Law, Control of
        Corruption. **Important gotcha found and confirmed live:** the
        real indicator codes are prefixed `GOV_WGI_` (e.g.
        `GOV_WGI_CC.EST`) — the bare `CC.EST`/`PV.EST`-style codes
        sometimes seen referenced elsewhere don't exist in World Bank's
        default catalog and return a real "not found" error if queried
        as-is; this was live-tested (all 6 indicators × all 5 default
        countries, zero nulls) before shipping, not assumed. One added
        Political Stability line also now appears in the map's hover
        popup, alongside the existing 4 economic indicators.
      - **Smarter world map** — a "Color by: Market / GDP Growth /
        Inflation" toggle now tints each of the 13 tracked countries'
        real landmass paths by indicator intensity (a genuine
        choropleth), reusing the same `color-mix()` technique already
        used for the sector heatmap tiles — not just the existing hover
        popup.
      - **São Paulo map dot fixed** — Jozsua reported it was mispositioned.
        Confirmed directly (via `getBBox()`/`isPointInFill()` on the
        live SVG) that the lat/lon value was correct but this specific
        basemap's own Brazil landmass polygon is itself drawn offset from
        where the map's lon/lat formula expects it — a basemap
        data-quality quirk, not a bad coordinate. Fixed with a manual
        pixel-offset correction on just the dot (`dotDx`/`dotDy`),
        verified visually against the live map (the dot now lands
        directly on Brazil's coastline, confirmed via `isPointInFill`),
        rather than touching the 1.3MB hand-authored basemap SVG itself.

## Pillar 6: AI research companion — validation step only, for now

- [ ] Ship a small set of **static** curated research threads (same
      shape as the two conversations that inspired this) as a taste test,
      before building any live LLM integration.
- [ ] Only after that: decide a cost model (who pays per query, any usage
      caps) before wiring up real API calls.

## Homepage overhaul v2: Seeking-Alpha-inspired persistent nav — complete 2026-09-23 (msv-web PRs #27, #28, #29)

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

**Shipped, 2026-09-23 — msv-web PR #27:**
- [x] Persistent `.app-shell` sidebar — visible on every page (ticker
      deep-dive, Compare, everywhere), not just Home. Logo at top, 3
      nav groups separated by dividers, 18 items total.
- [x] Stock Analysis / Market Data / Sectors / Market Intelligence /
      Watchlist / Market News all routed to their real content (existing
      tabs + smooth-scroll, or existing always-visible home cards).
- [x] Explore Products — a real directory of every destination, Coming
      Soon ones clearly marked, built the same day (no new data needed).
- [x] New `#placeholderView` — properly hides the rest of the app for
      Coming Soon pages, rather than showing empty content next to a
      live world map. Every existing view-toggle function (`goHome`,
      `loadTicker`, `loadCryptoTicker`, `showCompareView`,
      `goToHomeTab`) updated to hide it too, so nothing can get stuck
      showing two views at once.

**Real "Coming Soon" placeholders, not fake-functional UI — still
genuinely not built, UI-only for now:**
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
- [x] **Rest of the original v2 spec — shipped 2026-09-23, msv-web PR
      #29:**
  - Hero redesign — two columns, headline/description on the left, a
    real "Create a free account" panel on the right (email input +
    button, routes to the Create Free Account page above). No fake
    "Continue with Google/Apple" buttons — those would specifically
    imply real OAuth that doesn't exist.
  - Market News rebuilt as a compact two-column list — "Top Headlines"
    / "Latest News", not "Trending" — this app has no real trending/
    search-analytics signal the way Seeking Alpha's own does, so that
    label would have implied a signal that doesn't exist. "Top
    Headlines" is honestly just Finnhub's own returned order for the
    first few items; "Latest News" is the same data re-sorted by
    timestamp.
  - Winners/Losers/Most Active now render as a compact table (Symbol/
    Price/Change), matching the reference screenshots' dense list
    style. No "Rating" column — Seeking Alpha's is a proprietary quant
    score with real analytical infrastructure behind it; faking one
    would have broken this app's no-fabricated-data standard. The
    existing Heatmap view-toggle option is untouched.
  - Also shipped, separately, same day: sidebar icons replaced with
    hand-drawn line-style SVG (was emoji — Jozsua's feedback: "nicer
    looking, simplified and aesthetic"), logo badge shrunk ~25%
    (msv-web PR #28).

**The full homepage overhaul v2 spec is now complete.**

**Explicitly not copied from Seeking Alpha, and why**: their "Rating"
column is a proprietary quant score with real analytical infrastructure
behind it — inventing a fake rating just to visually match would break
this app's "no fabricated data" standard, so it's dropped rather than
faked. Same reasoning for their "Trending" stock list, which is powered
by real search analytics this app doesn't have — any "trending"-style
module here needs to be honestly labeled for what it actually is (e.g.
curation order), not implied to be a real trending signal.

## Homepage v3: routed pages, Market Intelligence showcase, one logo, Explore Products retaxonomy — shipped 2026-09-24

Jozsua came back after reviewing the live v2 homepage in person with a
large batch of asks: sidebar clicks should feel like leaving the page,
not scrolling down it ("I'm just prepping the structure, I don't want
everything on the homepage"); Watchlist should come off the homepage
since it's already in the sidebar; Market Intelligence needed a real
visual showcase; the header had a redundant second `$MSV` logo that also
looked wrong in light mode; and Explore Products needed real icons and a
much richer category structure with a new "Premium" line item.

- [x] **Router (`ROUTES`/`navigateTo`, home.js)** — replaced the old
      `NAV_ACTIONS` scroll-to-section map with a real router: every one
      of the 18+ sidebar/Explore destinations now shows a focused view
      (`showHomeFocused(sectionKey)`, driven by `data-home-section` tags
      on homepage cards in index.html) and updates the URL via the
      History API (`pushState`/`popstate`), so back/forward and
      refresh-to-a-specific-page work. Deliberately stayed one
      `index.html` (no build step, no per-page markup duplication) rather
      than real separate HTML files — same technique already used for
      the ticker/Coming-Soon views, just generalized. **Production note:**
      `msv-web/wrangler.jsonc` needed `assets.not_found_handling:
      "single-page-application"` added so a hard refresh on e.g. `/macro`
      doesn't 404 at the edge — confirmed against Cloudflare's current
      Workers static-assets docs before adding, since this is the one
      real deploy-risk item in this whole batch.
- [x] **Watchlist off the homepage** — moved out of `.home-grid` into its
      own card, only shown when the sidebar's Watchlist item is focused
      (not in the default "full homepage" view). Zero JS change to
      `renderWatchlist()` itself.
- [x] **Market Intelligence showcase + rename** — the tab itself was
      still internally labeled "Supply Chain" even though the sidebar
      already said "Market Intelligence" (a naming mismatch from the
      2026-09-23 sidebar build) — the tab title is now "Market
      Intelligence" everywhere user-facing (internal id/CSS class names
      kept as `supply-chain`, renaming those would be pure churn). A new
      homepage teaser card (below the category tabs, standalone, per
      Jozsua's explicit layout ask) shows 5 real company logos (NVDA,
      TSM, AMD, MSFT, GOOGL) via Finnhub's `/stock/profile2` `logo`
      field — the exact same field already used for the ticker
      deep-dive page's own logo, so zero new API integration. A "See the
      full picture →" button and the sidebar item both lead to the full
      17-node/20-relationship page.
- [x] **Single $MSV logo** — the header's separate `.brand-lockup` copy
      (fixed `#06120d` dark chip, "looks the same in both themes" by
      design) is removed entirely; only the sidebar badge remains,
      click-to-home wiring moved onto it. That badge's own hardcoded
      colors were the actual root cause of "looks weird in light mode" —
      swapped to the theme's own `--accent`/`--accent-contrast` tokens
      (the same pairing `#searchBtn` already uses) so it's still a solid
      brand-colored badge, just one that correctly follows the active
      theme instead of staying fixed.
- [x] **Explore Products retaxonomy** — icons switched from emoji to the
      sidebar's own inline SVGs (cloned at render time, zero duplicated
      markup, guaranteed pixel-identical); the flat 17-tile grid became 6
      named categories (Get Started / Stock Analysis / Market Outlook /
      Market Intelligence & Data / Portfolio Tools / Learn & Premium) per
      Jozsua's explicit request for more categories/subcategories, with
      genuinely new Coming Soon destinations added to fill it out (Stock
      Ideas, Stock Sentiment, Analyst Upgrades & Downgrades, Stock
      Screener, Precious Metals, Forex) rather than just regrouping the
      existing 17 tiles.
- [x] **"Premium" added to the sidebar** — a new Coming Soon destination
      (Group 1, right after Explore Products), no pricing/scope decided
      yet.

**Verified live against a local build** (real click-through of all 19
sidebar/Explore destinations, URL-per-destination, back/forward, and a
homepage-focus visibility check) before considering this done — see the
session's own testing notes; a follow-up production smoke check (hard
refresh on a non-home path) is still needed after the next `msv-web`
deploy to confirm the `wrangler.jsonc` change works end-to-end.

## Homepage v5: Market Data / Sectors / ETFs / Crypto pages — shipped and deployed 2026-09-27

- [x] Market Data: 42 countries, hover card, country profile, global risk
      dashboard, "colour by" map, dot fixes.
- [x] Sectors: 11 sectors + 52 industries, equal tiles, data table, detail panel.
- [x] ETFs: 42 categories, ~290 live-checked ETFs, search.
- [x] Crypto: full dashboard (see HISTORY.md).
- [x] Market Intelligence: ripple view, industry chips, research table.
- [x] Sidebar: Crypto grouped into its own section between Macro and
      Portfolio Builder, with two "Soon" placeholders (Bitcoin Cycles,
      Crypto News) previewing what the React POC already has.
- [x] **Deployed to production 2026-09-27** (`msv-web` commit `f33dea5`) —
      re-verified live: nav order, Sectors tiles, Crypto table, ETF chips
      all render with real data. **Known minor issue found during this
      check**, not a regression from this batch — see "Deploy: config.js
      404 becomes a console error" below.
- [ ] Re-verify the curated ticker lists every few months — companies get
      acquired/renamed (`sectors.js` reps, `etfs.js`, `crypto.js` ETF/stock lists).
- [ ] Placeholder industries in Market Intelligence (semiconductors, EV &
      batteries, energy, defence, pharma…) need real researched relationships
      before they're shown as anything but illustrative.
- [ ] Macro tab: differentiate from Market Data (see recommendations given
      2026-09-27: Macro = one country's economy over time, indicators as
      charts, policy/rates/inflation narrative; Market Data = cross-country
      and market-risk snapshot). Jozsua confirmed 2026-09-30 he's happy to
      go ahead — needs a scoping pass (does the Macro tab's own multi-
      country compare mode get trimmed in favour of Market Data's, and
      real time-series charts added?) before building, since it changes
      existing UI, not just a label.
- [x] **Market Intelligence company panel simplified — 2026-09-30.** The
      "click a company, see it below" panel had every relationship row
      showing name, badge, figure, description and source all at once.
      Now collapsed behind a "Why? ▾" toggle per row, same pattern as the
      Researched Relationships table below it. Deployed same day.
- [x] **TradingView chart widget added and deployed — 2026-09-30.** New
      "MSV Chart / TradingView" toggle on the ticker page. See BLOCKERS.md
      "Standing watch-items" for the non-commercial licensing constraint —
      revisit before any monetization.

## Stock screener — MVP scoped 2026-09-30, building with free-tier data only

Jozsua wants a comprehensive Webull-style screener as part of making the
app "a complete package." A screener was explicitly ruled out at project
start (Finnhub free tier's 60/min cap can't support per-ticker calls
across a large universe) — that constraint is still real, so this MVP
works within it rather than waiting on a paid data source:

- [x] **v1 — shipped and deployed 2026-09-30** (`msv-web` e52483f). Ended
      up using `BROWSE_CATEGORIES`' four stock categories (not
      `RANKING_STOCK_SYMBOLS` + `sectors.js` reps as originally sketched
      above — simpler, and those four categories alone already give ~70
      deduped stocks). Filters: category, market cap bucket, price range,
      today's up/down, max P/E. Sortable columns, click a row to open its
      ticker page. See HISTORY.md for the full build/verification notes.
- [x] **v2 — tested live 2026-09-30, confirmed blocked, not pursuing.**
      Jozsua provided a fresh FMP key; `/stable/company-screener` returns
      the same "Restricted Endpoint" error as the ETF holdings/info
      endpoints (see BLOCKERS.md). No free path to a broader-market
      screener via FMP. **Went a different direction instead, same day:**
      embedded TradingView's free Screener widget (real whole-market US
      stock coverage) as a second tab alongside the MSV Screener MVP —
      see "TradingView widgets" below.
- [x] **FMP's `/profile` endpoint IS genuinely free and useful, unlike
      the screener/holdings endpoints — shipped 2026-09-30.** Real ETF
      fund name/description/website/ISIN/beta for any ticker, wired into
      the ticker page (not the Screener). See BLOCKERS.md's "Full
      official fund name + issuer" item (now resolved) and HISTORY.md.

## TradingView widgets — both shipped 2026-09-30

- [x] **Chart widget** (ticker page) — "MSV Chart / TradingView" toggle,
      also gives crypto tickers a working chart for the first time.
- [x] **Screener widget** — "MSV Screener / TradingView" toggle on the
      Screener page, real whole-market US stock coverage (`market:
      "america"`), the practical answer once FMP's screener endpoint was
      confirmed blocked (see above).
- [ ] **Standing reminder:** both are free/non-commercial-only per
      TradingView's terms — see BLOCKERS.md's "Standing watch-items,"
      revisit before any monetization. Not an open task, just don't
      forget it exists.

## React proof-of-concept — approved and built 2026-09-27; Phase 1 (build pipeline) deployed 2026-09-30

Jozsua approved rebuilding the Crypto page in React + TypeScript (Vite) as
a proof of concept on 2026-09-27, then liked it and asked for it to be
"flooded with more data." Lives in `msv-web/react-poc/` — own
`package.json`/`node_modules` (gitignored), listed in `msv-web/.assetsignore`
so it never deploys. Committed to git for a real history, but the site
itself still ships as the plain HTML/JS it always has.

- [x] 8 tabs: Overview (market strip, Fear & Greed, trending, market
      breadth, Altcoin Season Index, market-cap dominance donut, movers
      with matched-news-or-data-signal "why"), Markets (table + coin
      profile + futures/derivatives), Exchanges, DeFi (categories, chain
      TVL, top protocols, top yield pools), Stablecoins (with live peg
      tracking), **Bitcoin Cycles** (Rainbow Chart, Stock-to-Flow model,
      halving schedule), **News** (live Finnhub crypto feed + a curated,
      dated Regulation & Adoption tracker — CLARITY Act, GENIUS Act, MiCA,
      UAE/Hong Kong — each fact re-verified live via web search before
      writing, not from memory), Learn.
- [x] Favourites, auto-refresh, shareable `?coin=`/`?tab=` URLs, dark/light
      toggle, a hover crosshair on the price chart.
- [x] Found and fixed a real honesty bug while building: the "why did this
      coin move" news-matching first used a plain substring search, which
      matched a coin named "Quant" to an unrelated article about
      "quantum" cryptography. Fixed with word-boundary regex matching.
- [x] **Phase 1 — build pipeline proven in production, 2026-09-30**
      (`msv-web` 39ce4c3). `npm run build` outputs to a new sibling
      `msv-web/react-crypto/` folder; deployed and linked from the live
      Crypto page as a clearly labelled beta. See HISTORY.md for the full
      writeup and the verification steps taken before calling it done.
- [ ] **Phase 3, page 1 (next) — port real Crypto functionality into this
      pipeline**, replacing the linked-beta stopgap: make `/react-crypto/`
      (or wherever it ends up) the actual Crypto page, retire `crypto.js`.
      Order after that: Sectors → ETFs → Screener → Market Data → Market
      Intelligence → Learn → Home → the ticker deep-dive page (last) →
      the sidebar/router shell itself (only once every page is React).
      Not started.
- [ ] Port Crypto Cycles (renamed 2026-09-30 from "Bitcoin Cycles" — same
      feature, Jozsua's call to avoid confusion with the separate Macro
      tab) and Crypto News into the vanilla site (the two sidebar
      placeholders added 2026-09-27) once the migration approach
      is decided.

## Deploy: config.js 404 becomes a console error — found and fixed same day (2026-09-27)

Production re-verification after the Homepage v5 deploy found `index.html`'s
always-present `<script src="config.js">` tag (empty/absent on purpose in
production — see msv-web/CLAUDE.md's "Config / secrets") now throws a real
`Uncaught SyntaxError: Unexpected token '<'` in the browser console. This
predated today's batch: it's a side effect of `wrangler.jsonc`'s
`not_found_handling: "single-page-application"` (added 2026-09-24), which
makes a missing `config.js` return the full HTML app shell with a 200
instead of a plain 404 — and a `<script>` tag that receives HTML instead
of JS throws a parse error instead of silently failing.

**Fixed** (`msv-web` commit `d12c217`, deployed same day): `index.html`
now only requests `config.js` when `location.hostname` is
`localhost`/`127.0.0.1`/empty (same check as `IS_LOCAL_DEV`), via
`document.write` from a tiny inline script placed where the old
`<script src="config.js">` tag was. Production never needed the file's
variables at all — every reference to them turned out to be either
`typeof`-guarded or sitting behind an `IS_LOCAL_DEV &&` short-circuit
that's `false` in production — so the fix doesn't touch `.assetsignore`
or the SPA fallback. Verified: local dev still loads `config.js`
unchanged, the Playwright smoke test still passes, and a real check
against the live production URL confirms `config.js` is no longer
requested and there are zero console errors.

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

## Homepage feature recommendations (brainstormed 2026-09-19)

Asked for "an extensive range of recommendations" to consider adding —
were options, not commitments, roughly ordered by how cheap/easy each
would be given the app's existing zero-cost architecture. **Status
re-checked 2026-10-02 (Jozsua: "built all the low-hanging fruit"):**
every item cheap enough to build zero-cost had, in fact, already been
built in earlier sessions without this list ever being checked off —
only the FX strip was actually still open. Corrected below so this list
doesn't send a future session re-investigating things that already ship.

- [x] **Upcoming earnings calendar strip** — already live, homepage
      `#earningsCalendarCard` ("Earnings This Week", `home.js`'s
      `loadEarningsCalendar()`/`loadEarningsCalendar()`'s Finnhub call).
- [x] **Economic calendar** — already live, homepage `#econCalendarCard`,
      backed by the hand-maintained `ECON_CALENDAR_EVENTS` list.
- [x] **A real watchlist** — already live (`script.js`'s
      `getWatchlist()`/`toggleWatchlist()`, `home.js`'s
      `renderWatchlist()`, the ☆ on every ticker page).
- [x] **Sector performance heatmap** — already live, homepage
      `#sectorHeatmapCard`, 11 sector ETFs.
- [x] **"Did you know" rotating fact** — already live, homepage
      `#didYouKnowCard` (`learn.js`'s `renderDidYouKnowTip()`) — actually
      pulls from real Learn hub lesson topics (deterministic by date, a
      new one each day) rather than the raw indicator-tooltip
      definitions this bullet originally envisioned; better content than
      planned, and click-through jumps straight to the full Learn
      article.
- [x] **Currency/FX strip** — shipped 2026-10-02, see HISTORY.md. 6 major
      pairs (EUR/USD, GBP/USD, USD/JPY, USD/SGD, USD/AUD, USD/CHF) via
      one batched Twelve Data `/quote` call, confirmed live against the
      real `msv-api` proxy before building (Finnhub still has zero forex
      coverage).
- [ ] **Trending/most-searched tickers this week** — needs some form of
      shared counter across visitors, which the current architecture
      doesn't have (everything today is per-browser, no shared backend
      state) — the one item here that's a real architecture addition,
      not just more UI. Not cheap; deliberately not attempted in the
      2026-10-02 low-hanging-fruit pass.
- [ ] **Investor-style line/bar graphs directly on the homepage** —
      flagged 2026-09-24 (Jozsua: "I want to see a bit more graphs on
      the homepage"). Recorded as a future intent, not scoped or built
      yet — the homepage doesn't have much of its own time-series data
      to chart today (most numeric data lives on the per-ticker
      deep-dive page, not the homepage itself). Worth revisiting once
      there's more homepage-native data to visualize (e.g. once the
      [DATA_EXPANSION_RECOMMENDATIONS.md](DATA_EXPANSION_RECOMMENDATIONS.md)
      ideas below start landing). Not cheap/well-scoped enough for the
      2026-10-02 pass either.

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
- [ ] **Custom domain for $MSV — bumped up in priority, 2026-09-30.**
      Still cosmetic (not blocking anything), but Jozsua asked to keep
      this high on the list rather than indefinitely parked. Needs: (1)
      a domain name — Jozsua doesn't have one yet and hasn't decided
      where to buy one, (2) Cloudflare dashboard → Workers & Pages →
      msv-web → Settings → Domains & Routes → Add Custom Domain. Steps
      are ready; just waiting on a domain being picked/bought.
- [ ] Cloudflare Workers Builds email notifications — Jozsua decided
      2026-09-21 to limit these to failures only (was getting one per
      PR, mostly "succeeded" noise). Can't be changed via the API token
      available here (confirmed — got a 403 on the Notifications
      endpoint; the wrangler OAuth token doesn't have that scope), so
      this needs Jozsua to do it himself in the Cloudflare dashboard
      (Notifications → the Workers Builds policy for msv-web/msv-api →
      select "Build failed" only). Not yet confirmed done.

## Jozsua's 2026-10-01 feature list — triaged, two batches shipped

Sent as one long list; sequenced into groups rather than attempted all at
once (session-usage-limit risk flagged up front, per Jozsua's standing
instruction). Group 1 (quick wins + the chart bug) and the low-hanging
half of Group 2 (moderate/no-new-data items) shipped the same session —
see HISTORY.md's "Chart honesty fix + layout density pass" and
"Low-hanging-fruit batch" entries for the detail. Not yet deployed to
production.

- [x] Chart 1D/4H dead-space bug (the "greyed out, unusable" report) —
      fixed 2026-10-01.
- [x] 1W chart x-axis showing time-only labels with no date — fixed
      2026-10-01 (found while investigating the above, not originally
      reported as a separate bug).
- [x] Bigger charting section (in-house chart canvas + TradingView-sized
      sub-panels) — 2026-10-01.
- [x] Font size -5% (via the existing `--page-zoom` variable) — 2026-10-01.
- [x] Narrower left sidebar, Seeking-Alpha-style — 2026-10-01.
- [x] More Screener columns — 2026-10-01: 52W High/Low, Beta, Dividend
      Yield, Avg Volume, all from data already fetched.
- [x] More stocks per homepage category — 2026-10-01: each of the 7
      BROWSE_CATEGORIES grew by 4-6 real, live-verified tickers.
- [x] ETF categorization by issuer — 2026-10-01: new "By Issuer" toggle
      on the ETFs page, 15 issuers (Vanguard, BlackRock/iShares, State
      Street/SPDR, Schwab, JPMorgan + 10 more) each with a blurb, derived
      from existing fund names so it can't drift out of sync — see
      HISTORY.md's "ETF By-Issuer view" entry.
- [x] **Feasibility research done — 2026-10-01**, see
      API_RESEARCH.md's "Prediction markets & 'who's holding/trading
      what'" section for the full writeup. Three separate verdicts:
  - [x] **Polymarket — shipped 2026-10-01.** New Prediction Markets page
        (`predictionMarkets.js`), 6 tabs via `tag_id` filtering (found
        `tag_slug` is silently broken on the live API — real bug caught
        before shipping). No msv-api proxy needed. Verified live: 24
        real markets per tab, including live Oct 2026 Fed-meeting odds.
        Not deployed yet.
  - [ ] **Michael Burry / 13F institutional holdings — buildable, but a
        real multi-piece build.** SEC EDGAR is free and official;
        confirmed live against Scion Asset Management's actual filings.
        Needs two new msv-api proxy routes (SEC Archives isn't
        CORS-enabled; OpenFIGI for CUSIP→ticker mapping isn't either)
        plus real XML parsing — scope as its own small project, not a
        quick add. **Not Burry-specific** — Jozsua clarified 2026-10-01
        this should cover any notable manager's 13F (Buffett, Ackman,
        Wood, etc.), same mechanism per manager via their CIK.
  - [ ] **Nancy Pelosi / congressional trading — no good free API found,
        don't build yet.** House Stock Watcher and Senate Stock Watcher
        (the usual free answer) are both confirmed dead. Quiver
        Quantitative has no free API tier. See BLOCKERS.md's new
        "Congressional stock trading data" entry — revisit later or pay
        for Quiver if this becomes a priority.
- [x] **Community/subreddit "what people are saying" page — researched
      2026-10-01, declined.** Not a "no free tier" situation — the
      platforms are actively closing: Reddit announced 2026-09-30 it's
      shutting down RSS (Nov 13, 2026) and its whole public API (March
      2027) for paid-only access; Stocktwits has new developer
      registrations closed indefinitely. See BLOCKERS.md's new
      "Community/subreddit sentiment data" entry. If still wanted,
      suggested the Crypto page's hand-curated "Regulation & Adoption
      tracker" pattern instead of a live feed — periodic research, not
      an automated integration. Also flagged: "summarize" needs a real
      LLM API call (real cost), different from every other free-tier
      integration in this app.
- [x] Bigger sector→top-ETFs popups — 2026-10-01: clicking a sector or
      industry now also shows other ETFs tracking the same market
      (cross-referenced from etfs.js's ETF_CATEGORIES, not a new hand-
      curated list) — see HISTORY.md's "other ETFs tracking this
      market" entry. 60/63 sectors+industries covered.
- [ ] **Not started — moderate, needs new UI:** world map's country list
      made a dropdown by default instead of a secondary view (Jozsua
      attached a reference image, but it didn't actually show the
      described UI — need a real screenshot/example before building
      this); more homepage sections; stock page 1D price/% change
      (likely already shown on the ticker deep-dive page and the dense
      quotes table — need Jozsua to point at the specific surface that's
      missing it before building something redundant).
- [ ] **Not started — professional-look design pass on the in-house
      chart** (distinct from the dead-space bug fix above): gridlines,
      candlestick styling, general visual polish so it reads as trustworthy
      as the TradingView widget next to it.
- [ ] **Not started — needs real research, not just more UI:** market
      news page expansion (way more categories) + an IPO calendar — same
      "confirm a free data source exists first" discipline as the other
      research items above.
- [ ] **Not started — content writing, no data risk:** Learn section
      stock-picking page — what indicators/tools people actually use for
      technical and fundamental analysis, and for finding lower-risk
      picks.
- [ ] React migration status, answered directly (not a to-do): Phase 3
      page 1 (Crypto) shipped 2026-10-02 — see the 2026-10-02 section
      below. Sectors is next.

## 2026-10-02

- [x] Page-wide zoom reduced further, `--page-zoom` 1.045 → 0.95 —
      2026-10-02, Jozsua asked for the whole site scaled down a bit more.
      Same knob as the 2026-09-21 (+10%) and 2026-10-01 (-5%) changes.
- [x] **React migration Phase 3, page 1 — Crypto shipped and deployed,
      2026-10-02.** Jozsua confirmed scope and pacing: migrate pages one
      at a time, each built, reviewed by Jozsua in the browser, then
      committed/pushed/deployed before starting the next — not attempted
      all at once. `crypto.js` retired; the sidebar's Crypto, Crypto
      Cycles, and Crypto News items now navigate to `/react-crypto/`
      (with `?tab=cycles`/`?tab=news` for the latter two) instead of the
      old vanilla page or a "Soon" placeholder. Also fixed while this was
      a real destination for the first time (not a beta opened in a new
      tab): the React page's theme toggle now shares the main site's
      `stockDashboardTheme` localStorage key instead of its own separate
      one, and got a "Back to $MSV" link in place of the old beta banner.
      Verified live against the real production deploy (response bodies,
      not just status codes). **Known gap:** the React page has no
      sidebar of its own yet, so leaving it means using that back link —
      acceptable with one page migrated, worth revisiting once more are.
      See `msv-web/CLAUDE.md`'s React migration section for the detail.
- [ ] **React migration — next up: Sectors**, then ETFs → Screener →
      Market Data → Market Intelligence → Learn → Home → ticker
      deep-dive → sidebar/router shell last. Not started.

## 2026-10-03

- [x] **CI green again** — fixed two load-time errors (`macro.js`
      formatter reference, forex strip timing). Commit `4276a9b`.
- [x] **Sectors in React — built, not yet deployed.** Tiles for all 63
      sectors/industries, detail panel, related-ETF table with YTD/1Y/beta/
      volume. Reachable at `/react-crypto/?page=sectors` after a build.
- [ ] **Sectors — not done yet:** "Open XLI page" button (the vanilla page
      opens the ticker in place; the React build has no way to do that
      until the ticker page moves to React). AUM and expense ratio columns
      are blocked on paid data (BLOCKERS.md).
- [ ] **Sectors — decision needed:** link the sidebar's Sectors item to the
      React page, replacing the vanilla one, once reviewed in the browser.
- [ ] **World map dropdown** — still waiting on a real screenshot of the
      wanted UI.
- [ ] **Next React page after Sectors:** ETFs → Screener → Market Data → … (unchanged order).
- [x] **Research: IPO calendar + market news sources (2026-10-03)** — Finnhub `/calendar/ipo` and `/news?category=general` both live-checked through the existing proxy; see API_RESEARCH.md. Building the pages is still open.
- [x] **World map / country picker — built 2026-10-03 from the written
      note (no screenshot).** The Market Data directory is now a grouped
      dropdown by default, with a "Full list" toggle back to the old view.
      Needs a look from Jozsua: if the layout isn't what was meant, say so.
- [x] **"Open XLI page" button — built 2026-10-03.** The React Sectors page
      links to `/?ticker=XLI`; the main site opens that ticker on load.
      Works on the live site only after the next deploy.
- [x] **IPO calendar page — built 2026-10-03 (React, `?page=ipo`).** Finnhub
      `/calendar/ipo`, live. Shows listings from two weeks back to 30/60/90
      days ahead, with a "likely SPAC" flag from the company name (a guess) and
      a hide-SPACs toggle. Not deployed yet.
- [x] **Market news page — built 2026-10-03 (React, `?page=news`).** Finnhub
      `/news` in four categories (general, mergers, crypto, forex) with a
      search box. Forex is thin on Finnhub. Not deployed yet.
- [x] **ETFs page in React — built 2026-10-03 (`?page=etfs`).** All 42
      categories under 8 families, live prices per category, a search across
      all 290 funds, and the By Issuer view. Fund links open the main site's
      ticker page via `?ticker=`. Not deployed yet. Not shown: AUM, expense
      ratio, holdings (paywalled). The fund profile panel from the vanilla page
      (FMP) isn't ported yet.
- [x] **Stock Screener in React — built 2026-10-03 (`?page=screener`).** The
      96 stocks from the four stock categories, with filters for category,
      market cap, price, today's move and P/E, and sortable columns. A cold load
      takes about 5–6 minutes (three calls per stock, paced for the free tier),
      and the table fills in as results arrive. Not deployed yet. The TradingView
      whole-market screener stays on the vanilla page.

## 2026-10-03 — deployed to the live site

- [x] **Deployed:** the React Sectors, ETFs, Stock Screener, IPO calendar and
      Market News pages are live at `/react-crypto/?page=…`.
- [x] **Sidebar switched:** Sectors, ETFs, Stock Screener and Market News in the
      sidebar now open their React pages (Crypto already did).
- [x] **Market Data in React — built and pushed (`b6a9c98`), not yet deployed (`?page=market-data`).** Now includes the world map, country profile and risk dashboard. Earlier note, superseded:
      the country picker (dropdown and full list), market hours and open/closed
      status, live country-ETF prices. Not done: the world map, the World Bank
      macro charts and the risk dashboard. The sidebar's Market Data item stays
      on the vanilla page until those are ported.
- [ ] **Still to port:** Market Intelligence, Learn, Home, ticker deep-dive page,
      sidebar/router shell, and the rest of Market Data (above).
- [ ] **Still open:** AUM and expense ratio (paywalled); chart polish (needs a
      target from Jozsua); the vanilla Sectors/ETFs/Screener code is now unused
      but not yet deleted.

## 2026-10-03 — status after Market Data

Still open:
- [ ] Deploy the Market Data React page and sidebar switch (Market Data item still points at the vanilla page).
- [ ] Market Intelligence, Learn, Home, ticker deep-dive page, sidebar/router shell in React.
- [ ] Delete the unused vanilla Sectors/ETFs/Screener code after review.
- [ ] Add IPO calendar page to the sidebar (page is live but not linked).
- [ ] AUM and expense ratio (paywalled); chart polish (needs a target).
- [ ] Custom domain, Workers Builds email decision, Quagmire link, pillar choice (see older sections).
- [x] Deployed Market Data React page and switched the sidebar item (2026-10-03, version 871684e5).

## 2026-10-03 — React migration status (latest)

- [x] Home fully in React (`?page=home`), Explore (`?page=explore`), Learn, Market Intelligence, Market Data (full), Sectors, ETFs, Screener, IPO, News: built and deployed (`952e8ad`).
- [x] Shared sidebar drawn in React on every React page (AppSidebar, 23 items from index.html).
- [ ] Main-site router and sidebar still vanilla. The React sidebar links to React pages where they exist and to the vanilla routes elsewhere.
- [ ] Ticker deep-dive page (script.js, ~103 KB plus chart, valuation, invest, compare) not started. It's the last big piece; the router's full move depends on it.
- [ ] Cleanup of the vanilla code for moved pages, after review.
- [ ] Crypto markets table: CoinGecko /coins/markets returns 403 through the proxy right now (affects live React Crypto too).
- [~] Ticker deep-dive page in React, first slice (`?page=ticker&symbol=AAPL`): header, price, ranges, company facts and valuation. Still on the main site: growth and profitability, financial health, dividends and risk, financial statements, ownership, insider transactions, SEC filings, options, chart, recommendations and earnings, news and peers, and the hover tooltips (definitions.js). ETF and crypto layouts also still on the main site.
- [x] Ticker page in React, second slice: valuation, growth, profitability, financial health, efficiency, risk, dividends and momentum, with tooltips and sector traffic lights (`?page=ticker`). Still on the main site: price chart, financial statements, ownership, insider transactions, SEC filings, options, recommendations and earnings, news and peers; ETF and crypto layouts.
- [x] Price chart on the React ticker page: ranges 1D to 5Y, line and candles, 20/50-day averages, hover readout, volume and RSI. Still on the main site: MACD and support/resistance.
- [x] Ticker page in React: analyst recommendations, recent earnings, company news and peers. Still on the main site: MACD and support/resistance, financial statements, ownership and insider transactions, SEC filings, options; ETF and crypto layouts; the comparison and "what if I'd invested" tools.
- [x] Ticker page in React: ownership note and insider trades. Still on the main site: MACD and support/resistance, financial statements, SEC filings, options; ETF and crypto layouts; comparison and "what if I'd invested" tools.
- [x] Ticker page in React: financial statements summary (last four quarters). Still on the main site: MACD and support/resistance, SEC filings, options; ETF and crypto layouts; comparison and "what if I'd invested" tools.
- [x] Ticker page in React: SEC filings with descriptions and EDGAR links. Still on the main site: MACD and support/resistance, options; ETF and crypto layouts; comparison and "what if I'd invested" tools.
- [x] Ticker page in React: options chain. Still on the main site: MACD and support/resistance; ETF and crypto layouts; comparison and "what if I'd invested" tools; the "real-life example" paragraphs.
- [x] Ticker page in React: ETF layout (fund details, performance, trading activity, chart, options, filings). Still on the main site: crypto layout; MACD and support/resistance; comparison and "what if I'd invested" tools; the "real-life example" paragraphs.
- [x] Ticker page in React: crypto layout (CoinGecko detail; the homepage crypto table fixed to use it, since the markets list is blocked on the proxy); chart MACD and support/resistance.
- [x] "What if you'd invested?" calculator and the Compare page in React. Chart indicators (MACD, support/resistance) done. Still to do: the "real-life example" paragraphs under each section; the main-site router (sidebar and page switching) moving to React; removing the vanilla code for pages that have moved.
- [x] Real-life example paragraphs under each ticker-page section (stock, ETF and crypto).

## 2026-10-03 — the main site now runs on React

- [x] Prediction Markets, the "Coming soon" pages, and Macro ported to React.
- [x] Main-site router moved: `/` and every sidebar path open their React page (index.html is now a redirect; the old homepage is kept as legacy.html).
- [ ] Review the React site in a browser, page by page, before deleting the vanilla code (legacy.html and the old JS files).
- [ ] Remove the vanilla code once reviewed.
- [ ] Market Data and Macro figures: check a few against the old pages.
- [ ] The React Crypto page's coin table still uses the markets list, which the proxy refuses (403). The homepage works around it; the Crypto page doesn't yet.
- [ ] Decisions still open: custom domain, Workers Builds emails, TradingView licence, which pillar next, chart polish target, world map screenshot.
