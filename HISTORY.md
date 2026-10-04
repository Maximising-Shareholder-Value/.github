# Project History

This is the story of $MSV, told in order, one dated entry at a time —
"first this happened, then that, then this." If you weren't around for
any of it (a new contributor, or just catching up), you should be able to
read this top to bottom and understand how the project got to where it
is today, without needing to ask anyone or dig through old messages.

Each entry answers: what changed, roughly when, and why it mattered —
not the fine technical detail of *how* it was built (that lives in each
repo's own `CLAUDE.md` instead, next to the actual code it describes).
Think of this file as the project's diary, and `CLAUDE.md` as its
technical manual.

Nothing here is guessed or reconstructed from memory — every entry is
checked against the real, actual git history (including the original
repo this project split from), so the dates and order can be trusted.

## Phase 1 — Built as one repo (2026-07-30 to 2026-08-27)

$MSV started as a single combined repo
([JozsuaHeng/Maximising-shareholder-value](https://github.com/JozsuaHeng/Maximising-shareholder-value),
now archived) containing both the frontend and the Cloudflare Worker
proxy, built in a fast, iterative run over about a month:

- **Jul 30** — initial commit: the core stock dashboard (search, deep-dive
  page, plain-English tooltips).
- **Jul 31** — first public deployment attempt (Cloudflare Pages
  Functions), then corrected to a real Cloudflare Worker; rate-limit
  resilience and a chart session-gap rendering bug fixed.
- **Aug 1** — lazy-loaded home page categories; ticker search autocomplete
  and side-by-side comparison view added.
- **Aug 4** — home page rebuilt as tabs; FRED Macro tab and CoinGecko
  crypto tab added; chart range semantics fixed; insider transactions
  added; a background pre-warming layer (KV + cron) added.
- **Aug 6-7** — a dense stretch of fixes and features: specific
  fetch-error messages, the 1W chart layout bug, candlestick toggle, the
  "What If You'd Invested?" calculator, self-explanatory SEC filings, a
  varied Outlook headline, **the KV/cron pre-warming layer removed again**
  in favor of simpler on-demand edge caching, a full homepage overhaul,
  the ranking universe trimmed to control API cost, and the first real
  geographic world map (replacing an earlier decorative graticule-only
  version).
- **Aug 8-27** — several rounds of world map redesign (callout boxes →
  floating stroke-outlined text → the current drop-shadow text approach),
  the sidebar ticker list briefly expanded to 30 then reverted to 20 after
  it broke the map's layout, a market breadth strip added, more macro
  indicators added to the Macro tab.
- **Sep 13** — ETF and crypto instrument types redesigned (previously
  rendered as a wall of "N/A" using the stock-shaped sections).

This phase is where almost all of the app's actual product surface got
built. See each current repo's `CLAUDE.md` for the technical detail
behind any of these — most of it is still directly relevant.

## Phase 2 — Split into the Maximising-Shareholder-Value org (2026-09-15)

**Sep 15** — the combined repo was split into two independently
deployable pieces once the project moved under a dedicated GitHub org and
outside collaboration became a real possibility:

- **[msv-web](https://github.com/Maximising-Shareholder-Value/msv-web)** —
  the frontend.
- **[msv-api](https://github.com/Maximising-Shareholder-Value/msv-api)** —
  the Cloudflare Worker backend/proxy.
- The old combined repo was archived; its old combined Worker deployment
  was deleted once the split was verified working.
- Git history was **not** carried over — both new repos started fresh.
  (This file exists partly to recover that lost continuity for anyone
  reading only the new repos.)
- `API_BASE_URL` (in `script.js`) was introduced as the one seam that
  lets the frontend call a separately-deployed backend instead of
  assuming same-origin `/api/xxx` paths.

## Phase 3 — Deployment hardening (2026-09-15 to 2026-09-18)

- **Sep 16** — `API_BASE_URL` pointed at the actually-deployed `msv-api`
  Worker (`https://msv-api.jozsua-heng.workers.dev`), verified with a live
  quote coming back through it.
- **Sep 18** — `wrangler.jsonc` added for static-assets-only deployment
  (msv-web has no server-side code of its own — it's a static site that
  calls out to msv-api).

## Phase 4 — CI introduced (2026-09-19)

- The first automated merge gate: a GitHub Actions workflow running a
  `node --check` syntax pass on every tracked JS file, plus a Playwright
  smoke test that loads the real home page and searches a ticker
  end-to-end (external APIs mocked via `page.route()`, so it needs no
  real API key and isn't at the mercy of rate limits).
- Landed as [PR #1](https://github.com/Maximising-Shareholder-Value/msv-web/pull/1),
  merged via admin override since branch protection requires 1 approving
  review and, as a solo project, there's no second person to approve it —
  worth revisiting once there's a real second contributor.
- Also surfaced that **Cloudflare Workers Builds** is already
  auto-connected to msv-web and runs its own build check on every PR —
  discovered via the PR's checks, not something that had been
  deliberately wired up in this conversation.

## Phase 5 — Vision and roadmap defined (2026-09-19)

Jozsua laid out a much larger long-term direction for the app — supply
chain visualization (upstream/downstream company relationships), proper
indicators for every asset class instead of stock-shaped sections showing
N/A, a country-comparable macro/political dashboard, a general
"explain the concept" education layer, and an embedded AI research
companion modeled on how he actually researches trades himself. This was
run through a structured impact/effort prioritization pass — see
[ROADMAP.md](ROADMAP.md) for the resulting sequencing and
[TODO.md](TODO.md) for the concrete next actions.

## Phase 6 — Governance docs + homepage rework (2026-09-19)

This `.github/` folder was created as the single place to find project
context, history, the roadmap, and open API research — see
[README.md](README.md) for the index. The home page also got a content
pass (more explanatory copy, a "how to use" section) and the world
map/ticker strip's live Finnhub calls were replaced with placeholder data
so the homepage looks finished without spending free-tier API budget on
every single visit — see the note in [ROADMAP.md](ROADMAP.md) for the
tradeoff this involves and when to reverse it.

## Phase 7 — Home page polish round 2 (2026-09-19)

Same-day follow-up after Phase 6 shipped: the intro copy was made more
casual (dropped the formal "(via ETFs)" parenthetical for a lighter
tone), the inline "how to use" `<details>` accordion was replaced with a
proper paginated popup modal (5 slides, prev/next + dot navigation,
reusing the same overlay pattern as the existing indicator-tooltip
popup), and every browse category (Trending Tech, Blue Chip, Dividend
Payers, Growth, ETFs, Bond ETFs) got ~50% more tickers (12 → 18 each).
Two new feature ideas were also scoped into [TODO.md](TODO.md): a
Recently-Viewed sidebar column with categorization, and extending the
existing Compare feature to cover a fund's underlying holdings, not just
its price stats.

## Phase 8 — Pillar 1 audit: mostly already solved (2026-09-19)

Before building anything for the "multi-asset-class indicators" pillar,
ran a real audit against the live production site (not local assumptions)
across a stock, an index-fund ETF, a bond ETF, two commodity trust ETFs,
a futures-based commodity ETF, a sector ETF, and two crypto tickers.
Every single one rendered with zero N/A in any visible card — the
2026-09-13 ETF/crypto redesign already generalized cleanly to commodity
ETFs without any extra work being needed. The one N/A found (2 instances
in AAPL's Insider Transactions table) turned out to be a real missing
field in a specific SEC Form 4 filing, not a bug. Net result: this pillar
turned out to be far closer to "done" than the original roadmap assumed
— see [TODO.md](TODO.md) for what's genuinely left (individual bonds and
options, which have no free-tier data source at all, not a rendering gap
to fix).

## Phase 9 — Alpaca (options) + World Bank (multi-country macro) backends live (2026-09-21)

Jozsua created a free Alpaca paper-trading account (no funding, no ID
verification needed — that's only required for a live account) and
shared the API Key ID/Secret. Both keys were added as Cloudflare secrets
on `msv-api` and a new `/api/alpaca` route was added to `worker.js`
(header-based auth, its own `proxyAlpaca()` function since Alpaca
doesn't use the query-param-key pattern the other four APIs share). A
second new route, `/api/worldbank`, was added at the same time — World
Bank needs no signup or key at all, so this was zero setup on Jozsua's
side, just backend work. Both were deployed and tested against **real**
upstream data before merging: a live AAPL options chain with real
bid/ask came back through `/api/alpaca`, and real Singapore CPI
inflation + Indonesia GDP growth came back through `/api/worldbank`.
Shipped as [msv-api PR #1](https://github.com/Maximising-Shareholder-Value/msv-api/pull/1)
— the first PR in that repo since the org split.

This closes the backend half of two roadmap items (options data for
pillar 1, multi-country macro data for pillar 4) — the frontend UI for
either (an options view on the ticker page, a country-selectable macro
dashboard) is still unbuilt, tracked in [TODO.md](TODO.md).

## Phase 10 — Changelog popup, copy rewrite, world map cleanup, Commodities category (2026-09-21)

A same-day follow-up batch after Phase 9:

- **"What's New" popup added** (`changelog.js`, 🔔 icon in the header) —
  a plain-English, dated list of recent changes, shown automatically the
  first time a returning visitor loads the app after something new
  ships (skipped entirely on a genuinely first-ever visit), and
  reopenable anytime via the bell icon. Answers Jozsua's ask for a way
  to see "what's been added, edited, changed" without needing to read
  git history.
- **Intro copy rewritten** in a more confident, editorial register
  (closer to financial-media writing like Seeking Alpha) — replaced the
  earlier casual/joke-heavy version.
- **Global Markets map and sidebar strip are now countries only.**
  `MARKET_TICKERS` used to also carry US indexes (QQQ/DIA/IWM),
  commodities (GLD/USO), and regional baskets (EFA/EEM) — none of which
  answer "which country's market is open," which is what that panel is
  for. Trimmed from 20 entries to 13, one per exchange, matching
  `worldMarkets.js`'s `EXCHANGES` list exactly.
- **New "Commodities" browse category** (18 tickers — precious metals,
  energy, agriculture, industrial metals, broad commodity baskets) — the
  indexes that got removed from the map (QQQ/DIA/IWM/EFA/EEM) already
  lived in the existing "ETFs" category, so only the commodities needed
  a genuinely new home.
- **Governance doc introductions rewritten** to be simpler and more
  explained (this file, `ROADMAP.md`, `TODO.md`, and the folder's
  `README.md` index) — per Jozsua's specific ask for plainer language.

## Note: the "six pillars" build request (2026-09-21)

Jozsua also asked to "build the six pillars" from the roadmap in this
same message. That's the entire long-term vision — including the
supply-chain visualization (real per-company research), an AI research
companion (a real cost decision), and a multi-country dashboard UI — in
one shot, which runs directly against both the roadmap's own recommended
staged sequence and Jozsua's own earlier "let's work through it slowly"
instruction. Flagged back to him rather than either attempting all six
at once (would produce shallow, rushed results across the board) or
quietly building only a fraction without saying so. See
[TODO.md](TODO.md) for how this got scoped down into an actual next step.

## Phase 11 — Badge fix + six-pillars breakdown (2026-09-21)

Two follow-ups after Jozsua saw Phase 10 live:

- **The What's New auto-popup was swapped for a quiet red badge** on the
  bell icon instead — a full-screen popup firing on every returning
  visit after a change shipped was more intrusive than intended. The
  badge clears the moment the bell is actually clicked, and (like the
  popup before it) still doesn't show anything on a genuine first-ever
  visit. Tested directly across three scenarios (first visit, returning
  visitor with an older "last seen" date, clicking the bell) rather than
  assumed correct.
- **The six roadmap pillars were broken down in detail** — see the
  "Pillar breakdown" section of [ROADMAP.md](ROADMAP.md), added in
  response to Jozsua asking for exactly that after the "build all six"
  request from Phase 9/10 was flagged as too large to attempt in one
  pass. Each pillar now has its real sub-parts spelled out, what's
  already done vs. open, a rough effort read, and the specific decisions
  still needed before building further — meant to give Jozsua what he
  needs to actually choose a starting point, rather than another vague
  restatement of the same six one-liners.

## Phase 12 — Pillar 1 shipped (v1), ETF page enrichment, BLOCKERS.md (2026-09-21)

The largest single batch of the day, after Jozsua reviewed the pillar
artifact and greenlit pillar 1 specifically:

- **Options view shipped.** A new "Options" card on the ticker page
  (stocks + ETFs) — nearest expiration only, the 7 strikes closest to
  the current price, bid/ask per call/put, the at-the-money row
  highlighted, a plain-English `(?)` tooltip explaining calls/puts/
  strike/bid-ask. Deliberately simplified per Jozsua's explicit request
  ("not the most familiar with options... keep it light"). Backed by
  the Alpaca proxy from Phase 9. Verified against real live contract
  data before shipping (a genuine test bug was caught and fixed along
  the way — Alpaca's option symbols encode expiration/strike/type in the
  contract symbol itself, e.g. `AAPL260921C00250000`, which needed a
  small parser).
- **ETF pages got real fund names, issuers, and descriptions** — a
  hand-curated `ETF_FUND_INFO` lookup (Finnhub gives nothing usable for
  ETFs, confirmed) covering the ~50 tickers already in the home page's
  browse categories, plus a fallback in the Wikipedia description lookup
  for when a specific fund has no article of its own (most don't) but
  its issuer does. Caught and fixed a real bug in testing: a bare
  "Vanguard" search resolved to the Wikipedia article about the military
  formation term, not the fund company — fixed with a small
  disambiguation override.
- **`BLOCKERS.md` added** — a single consolidated record of everything
  that's genuinely not buildable right now and exactly why (individual
  bonds, bid/ask, today's volume, after-hours prices, most ETF fund
  stats, the Cloudflare notification permission gap), created
  specifically so Jozsua doesn't have to keep re-asking about the same
  blockers, per his direct feedback that this had happened a few times.
- **The six-pillar visual now also lives in the actual repo/site**, not
  just as an external Claude artifact link — `roadmap.html` at the repo
  root, reachable at `/roadmap.html` on the live deployment (not linked
  from the main nav). The Claude artifact was refreshed to match.
- Cloudflare Workers Builds notifications: Jozsua chose "failures only";
  logged in [TODO.md](TODO.md) since it needs him to change it himself
  in the Cloudflare dashboard (no API permission available here).

## Phase 13 — Governance docs moved to this repo, FMP tested and parked (2026-09-21)

- **This `.github` repo became the real governance/PMO hub.** The five
  docs above (ROADMAP/TODO/HISTORY/BLOCKERS/API_RESEARCH) used to live
  in `msv-web/.github/`, which shared a name with this repo and caused
  real confusion — Jozsua expected *this* repo to be the project's
  planning home, and kept finding it empty while the actual content was
  siloed inside the frontend repo. Moved here; `msv-web/.github/` now
  holds only its CI workflow.
- **The FMP key Jozsua provided was tested live against the exact ETF
  endpoints needed** (Holdings, Info, Sector Weighting) — all three
  gated to FMP's Ultimate tier, confirmed by real `HTTP 402` responses,
  not guessed. Twelve Data's fundamentals endpoint, tested the same way
  through the existing proxy, is equally paid-only. **Jozsua's call:
  park the rich ETF stats (NAV/AUM/expense ratio/holdings/sector
  weighting) rather than pay for a data plan** — the key was not stored
  anywhere since there's currently nothing free-tier for it to unlock.
- Also this session: the site got a 10% overall zoom bump and wider
  content containers (1400px → 1600px) on desktop, and the "What's New"
  popup now shows a real date+time per entry instead of just a day.
- Clarified for Jozsua: the "Global Markets" world map and the "Macro"
  tab are two different things — pillar 4 (multi-country macro) is about
  the Macro tab, not the map.
- Discussed (not built) replacing the homepage's Market News section
  with supply-chain content — recommendation was to build that on the
  ticker page instead, tied to a specific company, once pillar 2+3
  actually has data to show; revisit featuring it on the homepage after.

## Phase 14 — Pillar 4 shipped (v1): multi-country macro dashboard (2026-09-21)

Same day, after the ETF-stats parking decision — Jozsua asked "what
should I do now," was given a clear recommendation (macro dashboard: the
only pillar item where every blocking decision was already made and
nothing new was needed), and said yes.

Built and shipped in one pass: a country picker in the Macro tab. US
stays on FRED (unchanged); China, Germany, Japan, and the UK are backed
by the World Bank proxy that had been sitting live-but-unused since
Phase 9. Landed on 4 indicators — GDP growth, inflation, unemployment,
current account balance — after live-testing showed World Bank's
interest-rate series (which would have matched the US tab's Fed Funds
Rate more closely) come back null for advanced economies in recent
years; current account balance was the replacement that actually had
real 2025 data across all 5 countries. Verified against real World Bank
data for China, Germany, and Japan individually before shipping, not
just assumed to work because the US path already did.

This closes pillars 1 and 4's first iterations in the same session —
both were "backend ready, just needs UI" items, which is exactly why
they were tackled before pillars 2/3/5/6, all of which need real
curation or design work first.

## Phase 15 — World map tied to the macro dashboard (2026-09-21)

Same day, immediately after pillar 4 shipped — Jozsua asked for the map
and the new macro dashboard to actually connect to each other, rather
than being two disconnected features that happened to both exist.

Removed the price/% line that used to sit permanently under every map
label (it duplicated the sidebar list one-for-one) and replaced it with
a hover interaction on each tracked country's real landmass — confirmed
live that `worldmap.svg` carries a per-country CSS class on its land
paths (e.g. `class="land coast jp"`), which made real per-country hover
detection straightforward rather than needing a redesigned map. The
hover popup shows the country's index price plus a live World Bank
macro snapshot (reusing the same indicators/fetch logic pillar 4 just
shipped). Hong Kong has no separate landmass shape at this map's
resolution (confirmed: zero SVG matches for any "hk" class) — its
marker dot is the hover target there instead. Sidebar list also gained
country flags.

A real bug was caught and fixed during testing: initially the hover
listeners were being re-attached to the map's country shapes on every
30-second re-render, which would have piled up duplicate listeners
forever since those SVG paths are part of a persistent, cached root
element (unlike the marker layer, which is destroyed and rebuilt each
render). Fixed by guarding the shape-hover attachment to run exactly
once via a dataset flag.

## Phase 16 — New Risk/Efficiency cards, deeper Profitability/Growth/Valuation (2026-09-22)

Jozsua asked what other multi-asset indicator info could be surfaced
(risk, performance, fund info, AUM). Answered by auditing Finnhub's
`/stock/metric?metric=all` live — it returns 133 fields for a real stock,
of which the app only used 31 (found by grepping `script.js` for every
`metric.` field reference and diffing against the live response). Jozsua
approved building all of the genuinely new fields found, same message
("can the risk and efficiency cards be built now? if yes, please go
ahead, as well as the rest").

Shipped in msv-web PR #19:
- New **Risk** card: Interest Coverage Ratio, Long-Term Debt/Equity,
  Dividend Payout Ratio.
- New **Efficiency** card: Asset Turnover, Inventory Turnover,
  Receivables Turnover.
- **Profitability** extended: ROA, ROI (neither shown before), plus
  5-year average gross/operating/net margins.
- **Growth** extended: quarter-over-quarter YoY growth, alongside the
  existing annual and 5-year figures.
- **Valuation** extended: Price/Sales, Price/Cash-Flow.

Stocks only — the ETF metric response was already confirmed exhausted in
the 2026-09-19/21 ETF audit (see Phase 12/13), so none of this applies
to ETF or crypto pages; both new sections were added to
`NON_STOCK_HIDDEN_SECTIONS` and confirmed live (mocked ETF instrument
type) to correctly hide. Every new indicator got its own `definitions.js`
tooltip entry, matching the existing plain-English format.

Tested against real field values captured live from the production
Finnhub proxy (not fabricated) across all 6 affected sections before
shipping — zero console/page errors, tooltips confirmed working. Full
CI (syntax check, Playwright smoke test, Cloudflare Workers Build) green
on the PR before merge.

## Phase 17 — Learn hub started: Pillar 5, "The Basics" shipped (2026-09-22)

Same day. Jozsua asked to start on Pillar 5 (the education layer),
framing the goal directly: "people like me who want a load of nicely
visualised information about everything and all aspects before making
an informed decision — this information I'm providing them is the
'informed' part." Apps like Seeking Alpha were cited as "a little
complicated" — the bar here is simpler and more visual, not less
thorough.

Scoped as a dedicated **Learn hub** (not tooltip-style — `definitions.js`
tooltips already exist and answer a narrower "what's this one number"
question) with 5 categories: The Basics, Reading the Numbers, Macro &
the Economy, Options 101, Putting It Together. Flagged upfront as a
large content-generation task and deliberately staged — build and ship
one category, get the format reviewed, then continue — rather than
writing all 5 in one uninterrupted batch.

Shipped in msv-web PR #20: **The Basics** (Stocks, ETFs, Bond ETFs,
Crypto) — 4 topics, each with a plain-English explanation, a simple
inline SVG/CSS visual (pie slice for stock ownership, a basket for
ETFs, lend/borrow arrows for bonds, a centralized-vs-decentralized
network comparison for crypto — no images, built from basic shapes so
they're both easy to verify correct and theme-colored automatically),
a grounded real-ticker example, and a "why it matters here" callout
linking back to the actual dashboard sections that concept explains.
The other 4 categories render as "Coming soon" cards in the live UI
right now, so their existence and scope is visible today rather than
being a surprise later.

Verified live via Playwright in dark theme, light theme, and at mobile
width (390px) — all render correctly, zero console/page errors. Full CI
green before merge.

## Phase 18 — Learn hub, "Reading the Numbers" shipped (2026-09-22)

Same day, immediately after Phase 17 — Jozsua confirmed the Basics
format and said to continue. Second of the 5 scoped Learn categories,
shipped in msv-web PR #21: **Reading the Numbers**, 5 topics (Valuation,
Growth, Profitability & Efficiency, Financial Health & Risk, Dividends),
each explaining what the matching real ticker-page card actually
measures, deeper than the `(?)` tooltips go, with its own visual — a
reusable two-bar comparison chart (price-vs-earnings for Valuation,
bills-vs-cash for Financial Health), an ascending bar chart (Growth), a
shrinking funnel showing revenue narrowing down through gross/operating/
net margin (Profitability), a turnover cycle diagram (Efficiency), and a
growing quarterly-payment timeline (Dividends) — all built from basic
SVG/CSS shapes, no images.

One design pass during testing: the turnover cycle diagram's "Assets"/
"Sales" labels initially sat too close to the connecting arcs in a
close-up screenshot — tightened the spacing before merging rather than
shipping it cramped.

3 categories remain: Macro & the Economy, Options 101, Putting It
Together (still visible as "Coming soon" in the live UI).

## Phase 19 — Pillar 5 complete: final 3 Learn categories shipped (2026-09-22)

Same day. Jozsua said to go ahead with the remaining 3 categories in one
message that also raised a much larger homepage-overhaul ask (see below)
— explicitly sequenced: finished the self-contained Learn content first
since it didn't depend on any of the overhaul's design decisions, and
flagged the combined scope as too large for one uninterrupted batch per
the standing "flag before heavy batches" instruction, rather than
attempting everything at once.

Shipped in msv-web PR #22:
- **Macro & the Economy** (4 topics): Interest Rates, Inflation, GDP
  Growth, Unemployment — each ties back to the real Macro tab/world map
  indicators already live.
- **Options 101** (4 topics): Calls and Puts, Strike Price & Expiration,
  Premium/Bid/Ask, Why People Use Options — deliberately basic and
  cautious (Jozsua flagged he isn't familiar with options himself); the
  closing topic is explicit that this is vocabulary, not a full trading
  education.
- **Putting It Together** (4 topics): Diversification, Risk Tolerance,
  Reading the AI Outlook, Red Flags — arguably the category that most
  directly answers Jozsua's original framing for this whole pillar
  ("this information I'm providing them is the 'informed' part"), built
  rather than skipped in favor of only the numbers-heavy categories.

**Pillar 5 is now complete** — all 5 scoped categories are live (21
topics total), no "Coming soon" cards remain. New reusable visual
primitives added along the way: a cause-and-effect chain, several-
inputs-converging-to-one-output, a "4 out of 100" dot grid, a two-column
icon comparison, a strike-price ladder, a two-pie diversification
comparison, and a risk-tolerance spectrum bar — all basic SVG/CSS
shapes, no images, verified live via Playwright with zero console
errors before merge.

## Homepage overhaul — scoped 2026-09-22, phase 1 shipped same day

Jozsua asked, in the same message as the above: where does the Learn
tab even live right now (answer: buried in the same tab row as Winners/
Losers/browse categories, which undersells it), and requested a fuller
homepage overhaul — Learn should NOT sit as a peer of those other tabs,
plus a new collapsible left-hand sidebar.

Scoped via a check-in rather than guessing at a full layout blind (this
touches navigation, the map, search, and every existing tab — expensive
to redo if the direction is wrong). Jozsua's answers:
- **Sidebar contents**: quick ticker search, Recently Viewed (moved from
  its current horizontal row into the sidebar), quick links to Learn/
  Compare/Macro, AND a Watchlist placeholder (new functionality — a
  manually-curated, localStorage-based list of tracked tickers) — picked
  all of the above, plus "anything else you recommend".
- **Learn's new placement**: a prominent banner card near the top of the
  homepage (e.g. "New to investing? Start here"), not inside the tab row
  and not just a sidebar link.
- **Extra features approved**: a "Did you know" rotating tip pulling
  from Learn's own content, a sector performance heatmap (reusing sector
  ETFs already in the app), and an economic calendar strip (hand-
  maintained, pairs with the Macro tab). Explicitly declined: leaving it
  at just the sidebar + Learn placement — Jozsua wants the extras too.

Flagged to Jozsua as needing its own dedicated pass rather than being
bundled into the same session as the Learn content — planned build
order: (1) the sidebar shell + Learn banner as one focused PR first,
since that's the riskiest/most structural part and everything else
layers on top of it, (2) the Watchlist feature, (3) the "Did you know"
tip (cheapest — just surfaces existing Learn content), (4) the sector
heatmap, (5) the economic calendar strip.

### Phase 1 shipped same day — msv-web PR #23

Repositioned existing functionality only, deliberately no new data
fetches, so this structural change stayed bounded and testable on its
own:
- New collapsible left sidebar: a lighter quick search (calls
  `loadTicker()` directly — no autocomplete, since `autocomplete.js` is
  wired to the single header `#tickerInput` only), quick links to
  Learn/Compare/Macro, a "Watchlist — Soon" placeholder, and Recently
  Viewed (moved from its old horizontal row into a vertical sidebar
  list). Collapse state persists via `localStorage`.
- Learn promoted out of the tab row entirely into its own banner card
  ("New to investing? Start here") above Global Markets — removed from
  `buildTabs()`'s tab list; `switchTab("learn")` works identically,
  only the entry point changed.
- Real bug caught in testing: the sidebar's Recently Viewed section
  wasn't hidden when the sidebar collapsed (it hadn't been wrapped in
  the same `.sidebar-body` class the other sections use), so its
  content visibly overflowed the collapsed rail — fixed before merge.

Verified live via Playwright: sidebar collapse + persistence across
reload, sidebar search loading a real ticker page, all 3 quick links
navigating correctly, Learn confirmed out of the tab row, Recently
Viewed populating correctly, dark/light theme, and mobile width (390px,
sidebar stacks above main content). Zero console/page errors. Phase 2
(Watchlist, sector heatmap, economic/earnings calendars, "Did you know"
tip) not started yet — see TODO.md's "Homepage overhaul" section.

## Supply chain pilot: AI infrastructure researched (2026-09-22)

Same day, run in the background (a research-only subagent, no code/file
access) while the homepage overhaul was being built, per Jozsua's
explicit go-ahead. Produced a sourced dataset of 13 candidate pilot
nodes and ~19 real company relationships (supplier/customer/investor/
manufacturing-partner), each cited to an SEC filing, official company
statement, or corroborated journalism with a confidence rating — see
[SUPPLY_CHAIN_RESEARCH.md](SUPPLY_CHAIN_RESEARCH.md) for the full
dataset and sourcing methodology.

**Explicitly not live-ready** — per this app's "no fabricated data"
standard, the dataset needs Jozsua's own review pass before it's treated
as ready to inform any visualization work, the same discipline used for
every other data source in this app (confirmed live, not just assumed
from a blog post). One specific row is already flagged in the doc as
needing careful handling if it ships: NVIDIA's own SEC filings disclose
real customer concentration (high confidence), but the specific company
*identities* behind "Customer A/B/C/D" are journalist/analyst inference,
not confirmed by NVIDIA — that distinction needs to survive into any
final UI, not get flattened into a flat "confirmed fact" list.

## Homepage overhaul complete: phase 2 + a same-day sidebar bugfix (2026-09-22)

Same day, right after phases 1 and the supply-chain research. Jozsua
looked at the shipped sidebar and flagged it looked broken when
collapsed, then asked for phase 2 to ship in full: "do phase 2 and make
it live, make everything live, i want to see it all."

**Bugfix first — msv-web PR #24.** Two real bugs in the phase-1
collapse implementation, found by actually looking closely at the
collapsed state rather than assuming the earlier screenshot review was
enough: `overflow: hidden` was clipping the toggle button (deliberately
positioned half outside the sidebar's edge), and the collapse rule hid
the Quick Links icons entirely instead of keeping them as a functional
icon rail — collapsed showed nothing but an empty strip with a floating,
half-cut-off toggle. Fixed: collapsed state now shows a clean, centered
3-icon rail (Learn/Compare/Macro, with `title` hover labels), no
clipping, and only the sections that genuinely need real text (Quick
Search, Watchlist, Recently Viewed) hide when collapsed.

**Phase 2 — msv-web PR #25**, all 5 items from the planned build order:
- **Watchlist**: a real ☆/★ toggle on every ticker page next to the
  company name, `localStorage`-backed (`stockDashboardWatchlist`,
  mirroring the existing Recently Viewed key/pattern exactly), sidebar
  list with a remove (×) per entry. No live price shown for watchlisted
  symbols — same zero-cost, name/symbol-only design Recently Viewed
  already uses.
- **"Did you know" tip**: one Learn hub topic surfaced per day (seeded
  by today's date, not random — stable all day, a new one tomorrow),
  clicking "Read more" jumps to the Learn tab with that exact topic
  pre-expanded and scrolled into view. A real technical constraint
  surfaced while building this: `home.js` calls its own `initHome()`
  immediately as soon as it finishes loading, which is BEFORE `learn.js`
  loads (per the script tag order) — so the tip's render function had to
  live in `learn.js` itself and self-initialize at that file's own
  bottom, rather than being called from `initHome()`, to avoid a
  "LEARN_CATEGORIES is not defined" error.
- **Economic Calendar**: hand-maintained, genuinely real dates — FOMC
  meeting dates confirmed live via web search against
  federalreserve.gov's own published 2026 schedule, CPI release dates
  against bls.gov's — not guessed or extrapolated from training data,
  consistent with this app's sourcing discipline everywhere else.
- **Earnings This Week** (a natural addition alongside the Economic
  Calendar, not originally itemized separately): one Finnhub
  `/calendar/earnings` call covering the whole upcoming week — confirmed
  live that this endpoint returns literally every company reporting in
  the range, including many unnamed micro-caps, so results are filtered
  down to symbols this app already has a real company name for
  (`RANKING_STOCK_SYMBOLS` + `BROWSE_CATEGORIES`) before rendering.
- **Sector Performance heatmap**: all 11 SPDR Select Sector ETFs (the
  ETFs browse category only had 7 of these, picked for general browsing
  — the heatmap needed the complete standard set), staggered quote calls
  40ms apart (same pattern already used for the ranking-tab loader),
  fetched once and cached for the session.

New `.home-grid` bento-style layout added to hold the three small cards
plus the wide heatmap, between the world map and the existing browse
tabs.

Verified live via Playwright with realistic mocked data across all 5
modules in one pass: Did You Know navigation/auto-expand confirmed
working end-to-end, Earnings Calendar's known-symbol filter confirmed
excluding a planted unknown test symbol while keeping real ones, Sector
Heatmap rendering all 11 tiles colored by performance, Watchlist's full
add → sidebar-shows-it → remove round trip confirmed. Also confirmed
graceful degradation (a generic empty-array mock response correctly
falls back to "couldn't load" messaging instead of erroring) and mobile
width (390px, single-column, no horizontal scroll). Zero console/page
errors across every check. Full CI green on both PRs before merge.

**The homepage overhaul scoped earlier today is now fully shipped** —
sidebar, Learn banner, and all 5 phase-2 modules are live in production,
verified via a direct cache-busted fetch against the deployed site
immediately after each merge (not just assumed from a green CI check).

## Sidebar removed; Supply Chain pilot made live (2026-09-22)

Same day, right after phase 2. Jozsua looked at the finished overhaul
and gave two direct instructions: "get rid of the left hand side
collapsable bar, it looks so bad" and "just make the supply-chain
research data set live in the app please" — msv-web PR #26.

**Sidebar removal.** Rather than just deleting it wholesale, the real
functionality inside it (Watchlist, Recently Viewed) was relocated into
the main homepage grid as its own cards, alongside the phase-2 modules
already there. Quick Search and Quick Links were dropped entirely rather
than relocated — both duplicated things that already exist elsewhere
(the header's own search box, the Learn banner, the Compare button, the
Macro tab), so there was nothing worth preserving there. `home.js`'s
`initHomeLayout()` shrank down to just wiring the Learn banner's click
handler. The homepage is back to one flowing column, just with a much
fuller grid of real content than before this whole overhaul started.

**Supply Chain pilot made live.** A new "Supply Chain" tab in the same
tab row as Winners/Losers/Macro. Jozsua's "just make it live" was taken
as his sign-off on the SUPPLY_CHAIN_RESEARCH.md dataset (rather than
requiring a separate, slower line-by-line review pass first) — a
reasonable read given he'd been saying "make everything live, I want to
see it all" consistently through this whole session. The open OpenAI/
Anthropic question from that doc was resolved by including them:
real nodes in the UI, clearly labeled "No ticker — private company,"
rather than cut from the pilot to keep to ticker-only entities. All 17
nodes and 20 relationships were transcribed verbatim from the research
doc into `supplyChain.js` — re-read directly from the source doc during
implementation specifically to avoid any transcription drift, given the
"no fabricated data" stakes of getting this wrong. Every relationship
card in the live UI shows its own source and a High/Medium confidence
badge rather than stating anything as flat fact, and the one row the
research doc specifically flagged as needing careful handling — NVIDIA's
real, SEC-disclosed customer concentration vs. the journalist-*inferred*
(not NVIDIA-confirmed) identities behind "Customer A/B/C/D" — keeps that
exact distinction visible in its own highlighted note in the shipped UI,
verified via a close-up screenshot review before merge.

This is deliberately the functional version, not the "sexy visual"
node-graph originally envisioned for this pillar (nodes as a clickable
grid, relationships as a card list) — real data live now, with the
fuller visual treatment left as a later, separate design pass rather
than blocking on it.

Verified live end-to-end against the actual deployed production site
(not just CI) immediately after merge: sidebar confirmed absent from the
DOM, Watchlist/Recently Viewed confirmed still fully functional from
their new grid-card homes, all 17 Supply Chain nodes and 20
relationships confirmed rendering with zero console errors, tickers in
both the node grid and relationship cards confirmed linking to real
ticker pages.

## Homepage overhaul v2: persistent sidebar, Seeking-Alpha-inspired (2026-09-23)

Next day. Jozsua came back with a much more detailed spec than either
prior homepage pass — three reference screenshots of Seeking Alpha's own
layout attached, explicit instruction to "learn and adopt their best
practices," and a full 18-item nav list with exact ordering and grouping
(account actions / main sections / personal tools), each item marked
whether it should be a genuinely new "Coming Soon" build.

Before touching code, worked through every nav item against what this
app actually has and proposed a concrete mapping — confirmed with
Jozsua via a single check-in rather than guessing and risking a rebuild
in the wrong direction a third time. Also answered a direct question
along the way: of the six roadmap pillars, four are substantially
shipped (options/multi-asset, macro dashboard, the Learn hub complete,
and supply chain data now live though not yet the "sexy visual"
treatment); Pillar 6 (AI research companion) hasn't been started at all.

Shipped in msv-web PR #27:
- **A genuinely persistent sidebar** — structurally different from
  yesterday's (which lived inside `#homeView` and only showed on the
  homepage). This one wraps the *entire app* in a new `.app-shell` flex
  container, with the sidebar as a true sibling of the header and every
  view — visible on the ticker deep-dive page, Compare, everywhere. The
  24px page padding that used to sit on `body` moved to a new
  `.app-body` wrapper so the sidebar itself can run flush to the
  viewport edge and full height, matching how Seeking Alpha's own nav
  behaves.
- **Existing features repositioned, not rebuilt**: Learn, ETFs, Crypto,
  Macro, and the Supply Chain tab (relabeled "Market Intelligence")
  route through the existing tab system plus a smooth-scroll down to the
  content card. Market Data, Market News, Sectors, and Watchlist point
  at parts of the home page that were already always-visible — clicking
  them just scrolls there. Stock Analysis absorbs the Winners/Losers/
  Most Active/browse-category tabs that used to sit in a bare horizontal
  row with no real heading of their own.
- **Explore Products, built for real**: a directory listing every
  destination in the app, Coming Soon ones clearly badged — not a
  placeholder itself, since it needed no new data to build properly.
- **Five real "Coming Soon" pages** (Create Free Account, Log In,
  Performance, Portfolio Builder, Portfolio Health Check) — a new
  `#placeholderView` section that genuinely hides the rest of the app,
  rather than showing an empty "coming soon" box next to a live,
  distracting world map. Every existing view-toggle function across the
  codebase (`goHome`, `loadTicker`, `loadCryptoTicker`, `showCompareView`,
  `goToHomeTab`) was updated to also hide this new view, closing off the
  same "two views visible at once" bug class that had to be fixed twice
  during yesterday's build.
- Explicitly **not** copied from Seeking Alpha: their proprietary Quant
  Rating column (would require fabricating a score this app has no real
  data behind) and their "Trending" stock list (powered by real search
  analytics this app doesn't have) — both would have broken the site's
  no-fabricated-data standard just to look more similar.

A real (if minor) investigation during testing: an Explore Products
screenshot appeared to show "Coming Soon" badges on the wrong tiles
(Macro, Compare) instead of their actual owners (Performance, Portfolio
Builder). Rather than trusting the screenshot, pulled precise DOM
bounding boxes for every tile and badge before concluding anything — the
badges were correctly positioned entirely within their own tiles the
whole time; the visual proximity to the neighboring tile's corner had
just made the screenshot easy to misread. A good reminder to verify
against the DOM, not just eyeball a render, before calling something a
bug.

Verified live against the actual deployed production site immediately
after merge (not just CI): 18 nav items confirmed present, Market
Intelligence confirmed still routing correctly to all 17 Supply Chain
nodes, zero console errors.

**Still queued, not part of this PR** (recorded in TODO.md): the hero
section redesign (tagline + description + a "Create Account" CTA block),
a compact Market News split into Trending/Latest columns, and a Winners/
Losers/Most Active table restyle inspired by the reference screenshots.

## Homepage v2 completed: icons, hero, compact news, movers table (2026-09-23)

Same day, two more rounds. Jozsua looked at the freshly-shipped sidebar
and gave quick, direct feedback: the emoji icons "not... nicer looking,
simplified and aesthetic," and the logo needed to be smaller. Then, in
the same breath, said to keep going with the rest of the queued v2 work.

**Icons — msv-web PR #28.** All 18 emoji replaced with hand-drawn line-
style SVG icons — simple geometric shapes (circles, rects, lines, basic
paths), the same "basic shapes" approach already proven for the Learn
hub's inline visuals earlier this session. No icon font or library
pulled in; everything stays inline, consistent with this app's plain-
HTML/no-build-step philosophy. All icons use `currentColor` so hover/
active states apply automatically. Logo badge shrunk roughly 25% in
font-size and padding.

**Hero, news, movers table — msv-web PR #29**, the last 3 items from the
original v2 spec:
- **Hero**: rebuilt into two columns — headline and description on the
  left (unchanged copy, new heading), a "Create a free account" panel on
  the right (email input + button, routes to the Create Free Account
  Coming Soon page). Deliberately no "Continue with Google/Apple"
  buttons — those specifically imply real OAuth integration that
  doesn't exist, which would have been a more misleading placeholder
  than a plain, honestly-labeled CTA.
- **Market News**: rebuilt as a compact two-column list, replacing the
  old hero-card-plus-image-grid layout. Labeled "Top Headlines" and
  "Latest News" — deliberately NOT "Trending", since this app has no
  real trending/search-analytics signal the way Seeking Alpha's own
  does; using that word would have implied a signal that doesn't exist.
  "Top Headlines" is honestly just Finnhub's own returned order for the
  first few items; "Latest News" is the same data re-sorted strictly by
  timestamp.
- **Movers table**: Winners/Losers/Most Active now render as a compact
  table (Symbol/Price/Change) instead of a card-tile grid, matching the
  density of the reference screenshots. No "Rating" column — Seeking
  Alpha's is a proprietary quant score backed by real analytical
  infrastructure; inventing one here just to look similar would have
  broken this app's no-fabricated-data standard. The existing Heatmap
  view-toggle option was left untouched.
- Cleaned up now-dead code left behind by these replacements
  (`buildNewsCard`, `buildGrid`, `NEWS_CATEGORY_COLORS`, unused chip CSS).

Verified live against the actual deployed production site immediately
after each merge: hero CTA confirmed routing to the Coming Soon page,
both news columns confirmed present with correct content, all 18 SVG
icons confirmed rendering, zero console errors throughout.

**The full homepage overhaul v2 — every item from Jozsua's original
Seeking-Alpha-inspired spec — is now complete and live**, across PRs
#27, #28, and #29.

## Homepage v3, Pillars 1 & 4 to 100%, Explore Products retaxonomy (2026-09-24)

Jozsua reviewed the live v2 homepage in person and came back with a large
batch of asks in one sitting — a genuine architecture change (sidebar
navigation) alongside several feature/polish requests and two
pillar-completion pushes. All frontend-only (`msv-web` +
`msv-org-github`); confirmed zero `msv-api` changes were needed anywhere
in this batch, since `/api/worldbank` and `/api/alpaca` were already
generic passthrough proxies.

**Routed navigation (`home.js` `ROUTES`/`navigateTo`).** Sidebar/Explore
clicks used to scroll to a homepage section; Jozsua explicitly wanted
them to feel like leaving the page instead ("I don't want everything on
the homepage"). Replaced the old scroll-based `NAV_ACTIONS` with a real
router — every destination shows a focused view (`showHomeFocused`,
driven by new `data-home-section` tags) and updates the URL via the
History API, so back/forward and refresh-to-a-page work, without
splitting into real separate HTML files (stayed one `index.html`, no
build step, reusing the same full-view-swap technique the ticker/Coming
Soon pages already used). `msv-web/wrangler.jsonc` gained
`assets.not_found_handling: "single-page-application"` so a hard refresh
on a non-home path doesn't 404 once deployed — confirmed against
Cloudflare's current docs before adding.

**Homepage content.** Watchlist moved off the always-visible homepage
grid (already reachable via the sidebar). A new Market Intelligence
teaser card — 5 real company logos via Finnhub's `/stock/profile2`
(zero new API integration, same field the ticker page's own logo
already uses) — now sits standalone below the category tabs, linking to
the full 17-node/20-relationship page. The tab itself, previously
mislabeled "Supply Chain" even though the sidebar already said "Market
Intelligence," is now consistently named everywhere user-facing.

**One $MSV logo.** The header's separate fixed-color logo copy is gone;
only the sidebar badge remains. Its own hardcoded near-black colors
turned out to be the actual cause of "looks weird in light mode" (a
fixed dark chip on a now-white page) — fixed by switching it to the
theme's own `--accent`/`--accent-contrast` tokens, the same pairing
`#searchBtn` already used, so it now correctly follows the active theme.

**Explore Products.** Emoji icons replaced with the sidebar's own inline
SVGs, cloned at render time for pixel-identical icons with zero
duplicated markup. The flat 17-tile grid became 6 named categories (Get
Started / Stock Analysis / Market Outlook / Market Intelligence & Data /
Portfolio Tools / Learn & Premium), with genuinely new Coming Soon
destinations added to fill it out rather than just regrouping existing
tiles (Stock Ideas, Stock Sentiment, Analyst Upgrades & Downgrades,
Stock Screener, Precious Metals, Forex). A new "Premium" item was added
to the sidebar itself.

**Pillar 1 → 100% within free-tier limits.** The Options card gained an
expiration picker and an expandable full-strike table (both reslicing
data the original fetch already pulled — zero new network cost) plus an
optional last-trade-price/volume view (fields already present in every
Alpaca snapshot, simply unused before). A live check of Alpaca's free
feed confirmed no `greeks`/`impliedVolatility` field exists at all —
recorded as a permanent blocker (BLOCKERS.md) rather than left an open
question, closing out the pillar's only remaining ambiguity.

**Pillar 4 → 100%.** Indicator breadth expanded (GDP per Capita, Trade
Balance, Government Debt, Total Reserves, Population) plus a new
Governance dimension — World Bank's Worldwide Governance Indicators,
never referenced anywhere in this codebase before. A real gotcha found
and confirmed live: the actual indicator codes are prefixed `GOV_WGI_`
(e.g. `GOV_WGI_CC.EST`), not the bare `CC.EST`-style codes that don't
exist in World Bank's default catalog. All 6 governance indicators
live-tested against all 5 default countries before shipping (zero
nulls). Also shipped: an open ~217-country search picker, a 4-country
comparison view, and a "smarter" world map — a Color-by toggle
(Market/GDP Growth/Inflation) that tints real country landmasses by
indicator intensity, a genuine choropleth reusing the sector heatmap's
own `color-mix()` technique. Separately, the São Paulo map dot — reported
mispositioned — was traced to this basemap's own Brazil landmass polygon
being drawn offset from where the map's lon/lat formula expects it (not
a bad coordinate); fixed with a hand-calibrated pixel offset, confirmed
landing directly on Brazil's coastline via `isPointInFill()`.

**Data-expansion research.** A new [DATA_EXPANSION_RECOMMENDATIONS.md](DATA_EXPANSION_RECOMMENDATIONS.md)
answers Jozsua's "how do we flood this app with data on free tiers"
ask — the single highest-value confirmed gap found: CoinGecko's
`/coins/{id}/market_chart` (confirmed live, no key needed) would give
crypto tickers a real historical price chart, which they currently have
none of at all. Also confirmed live: Twelve Data's free tier does
support forex quotes (resolving a question TODO.md's homepage-brainstorm
list had left open) — Finnhub's own forex coverage remains zero.
Finnhub's own field-usage gap and candidate new FRED series are flagged
as needing a fresh live check with a real API key, which this session
didn't have access to (the local `config.js` holding real keys is
gitignored and wasn't populated here) — the methodology is documented so
it's independently re-runnable rather than asserting stale numbers as
current fact.

Verified via a local build before considering this done: all 19 sidebar/
Explore destinations click through correctly with the right URL and
focused content; the official Playwright smoke test and a `node --check`
syntax pass across every `.js` file both stayed green throughout.

## Deploy incident: .git and node_modules briefly public (2026-09-24)

Same-day follow-up to the phase above, discovered while deploying it.
`msv-web/wrangler.jsonc`'s `assets.directory` is `"./"` (repo root) and
no `.assetsignore` file existed yet — the first deploy of the day
uploaded the entire repo as public static assets, including `.git/`
(full history, refs, branch names) and
`node_modules/.cache/wrangler/wrangler-account.json` (Cloudflare account
ID + email, not an auth token). Confirmed live via a direct request to
`/.git/config` returning real file content (HTTP 200) before the fix,
and confirmed again after — this wasn't assumed, it was checked both
ways. Fixed same-day with a tracked `.assetsignore` excluding `.git`,
`.github`, `node_modules`, `.wrangler`, `test-results`,
`playwright-report`, `config.js`, and `wrangler.jsonc` itself; verified
fixed by checking the response **body** rather than status code (the
`not_found_handling: single-page-application` setting added the same
day makes every path return HTTP 200 regardless, so a status-code-only
check can't distinguish "excluded" from "still exposed" — this tripped
up the first verification attempt before the right check was used).
Documented permanently in `msv-web/CLAUDE.md`'s new "Deploy safety"
section so this can't be silently reintroduced.

No evidence of an actual credential leak — the exposed wrangler-account
file held only an account ID and email, not a token, and `config.js`
(the file that would hold real API keys) is gitignored and was never
committed, so it was never part of what got uploaded either way.

## Homepage v4: dependency diagram, dense tables + heatmaps, sidebar fixes (2026-09-26)

Another in-person review round. **Sidebar bug root cause:** the earlier
"10% bigger" `body { zoom: 1.1 }` made the sidebar's `100vh` height render
10% taller than the screen, cutting off its bottom (the API usage panel).
Fixed by dividing the zoom back out (`--page-zoom`) and splitting the
sidebar into a scrolling nav plus a pinned footer (API usage + a short
educational-use disclaimer). Logo lockup enlarged with the full name
beside it. **Market Intelligence** left the tab row and became its own
container: a left-to-right SVG dependency web (AI labs → cloud →
AI infrastructure → chips & power gear → foundry & memory → tools & raw
materials) with arrows pointing at what each company depends on; click a
bubble for details, click again for its stock page. The 20 researched
relationships stay as solid "sourced" links; the power/cooling and
raw-material layers plus a few well-known chip-design ties are the app's
own structural reasoning, drawn dashed and labelled "Inferred" (not part
of the SUPPLY_CHAIN_RESEARCH.md pass — no specific contracts or figures
claimed). **Stock tabs:** Winners/Losers/Most Active and every browse
category now use one dense sortable table (crypto-style) plus a detailed
heatmap; browse categories now fetch live quotes on demand instead of
being name-only chips. Home defaults to Trending Tech. **Calendars:**
economic calendar expanded to 25 upcoming Fed/CPI/PPI/jobs/JOLTS/GDP/PCE
dates read directly off the FOMC, BLS and BEA schedule pages, each row
linking to its source; earnings widened to notable reporters with
estimates and Nasdaq links. Explore Products compacted and expanded to 52
products/sub-products.


## Homepage v5: Market Data, Sectors, ETFs and Crypto become real research pages (2026-09-27)

Jozsua's review of v4 asked for the site to be "extremely detailed" and
more engineer-friendly. Shipped locally (frontend only — `msv-api` needed
no change):

- **Market Data:** 13 → 42 countries (all BRICS members, Indonesia,
  emerging and frontier markets), each tagged with its group and a
  live-verified country ETF where one exists. Richer hover card; clicking
  a country (map or list) opens a country profile below the map (KPIs vs
  peer group, history charts, governance scores, similar economies, live
  market data). New global risk dashboard (VIX, rates, inflation, credit,
  country risk) from free FRED/World Bank series, using stated rule-of-
  thumb bands. Map "colour by" modes. Mumbai/Johannesburg dots fixed by
  re-deriving the map projection from country bounding boxes instead of
  assuming a standard one.
- **Sectors:** 11 sectors + 52 industries/themes, equal-size heatmap
  tiles, a sortable all-data table, and a per-sector/industry detail panel
  (price, performance vs the S&P 500, drivers, curated representative
  companies — labelled as not live holdings).
- **ETFs:** 42 categories (~290 ETFs, all live-checked) across US,
  sectors, themes, income, global, bonds, commodities/mining, and
  crypto/volatility/leveraged; search box; the old top pills removed.
- **Crypto:** market overview, Fear & Greed gauge, trending, sortable
  top-100 with sparklines, coin profile with price chart, categories,
  DeFi TVL, stablecoins, crypto ETFs/stocks, Crypto 101.
- **Market Intelligence:** chain/ripple view, industry category chips
  (AI live, others placeholders with illustrative names only), the 20
  researched relationships redesigned as a scannable table.
- **Fixes:** sidebar is one continuous scroll; the stock category pills
  now show only on stock tabs; homepage/hero type slightly smaller.
- Free-tier lessons: the World Bank throttles bursts (fixed with a
  4-at-a-time limiter + retry); CoinGecko's free plan returns empty
  community/developer data; ~20 curated company tickers had been
  delisted/acquired and were pruned or renamed (SQ→XYZ, FI→FISV).

## Homepage v5 deployed; React proof-of-concept built and grown (2026-09-27)

Jozsua asked to review the above before deploying ("go through the nitty
gritty details"), then approved it later the same day, along with a
second, separate ask: try rebuilding one page in React as a first step
towards migrating the whole frontend.

- **React + TypeScript proof of concept** (`msv-web/react-poc/`, Vite —
  own `package.json`, not deployed, listed in `.assetsignore`): the Crypto
  page rebuilt as components. Jozsua liked it and asked for it to be
  "flooded with more data," so it grew from a straight rebuild into 8
  tabs: Overview (market breadth, Altcoin Season Index, a market-cap
  dominance donut, and a "today's movers" panel that explains *why* a
  coin moved — either a genuinely matched real news article or a plainly
  labelled data signal, never a guess), Markets, Exchanges, DeFi,
  Stablecoins (with live peg-deviation tracking), a new **Bitcoin Cycles**
  tab (a Bitcoin Rainbow Chart and a Stock-to-Flow model, both heavily
  caveated as informal community models, not predictions), a new **News**
  tab (a live Finnhub crypto feed plus a hand-checked, dated Regulation &
  Adoption tracker — CLARITY Act, GENIUS Act, MiCA, UAE/Hong Kong — every
  fact re-verified live via web search rather than pulled from memory,
  since crypto policy moves too fast to trust a training cutoff), and
  Learn. Caught and fixed a real bug along the way: the news-matching
  logic first used a plain substring search, which wrongly matched a coin
  named "Quant" to an unrelated article about "quantum" cryptography —
  fixed with word-boundary regex matching before it shipped.
- **Sidebar:** Crypto grouped into its own section (Crypto, plus two new
  "Soon" placeholders — Bitcoin Cycles, Crypto News) between Macro and
  Portfolio Builder, previewing where the POC's richer content is headed
  once it's ported into the real site.
- **Deployed to production** the same day (`msv-web` commit `f33dea5`),
  after Jozsua's review. Re-verified live: Sectors, ETFs, Crypto, and the
  new sidebar section all render with real data. One small pre-existing
  console error was found during this check (not caused by this batch,
  see TODO.md) and **fixed and redeployed the same day** (`d12c217`):
  `index.html` now only requests the local-dev-only `config.js` on
  `localhost`/`127.0.0.1`, so production never triggers the SPA
  fallback's HTML-served-as-JS parse error in the first place.
- Still open: which page migrates to React next, and giving the site a
  real build step so a React page can actually ship to production (today
  proved the pattern locally, not in production).

## Market Intelligence company panel simplified — 2026-09-30, not yet deployed

Jozsua reviewed the "click a company, see it visualised below" panel
(`renderMiPanel()` in `msv-web/supplyChain.js`) and asked for it to be
easier to read. Each relationship row (`miEdgeRowHtml()`) used to show
the company name, a "Sourced · [confidence]" or "Inferred" badge, a
figure, the full description paragraph, and the source citation all at
once — dense on first glance, especially with several rows per column.

**Fix:** each row now shows just the company name and a plain
"Sourced"/"Inferred" tag by default, with a "Why? ▾" toggle that reveals
the figure, description, confidence level, and source underneath — same
expand-on-click pattern already used by the Researched Relationships
table lower on the page (`renderMiResearch()`), so the interaction is
consistent across the tab rather than a new pattern. No data or research
content removed, just tucked behind one click. Verified `node --check`
passes; not yet deployed — needs a local look via `npx http-server -p
8917` (or Live Server) before shipping, per the usual preview-before-
deploy step.

## TradingView chart widget added — 2026-09-30, deployed (`msv-web` c6e541c)

Jozsua asked for "powerful chart capability like TradingView." Rather than
try to rebuild TradingView's feature set in `chart.js`, embedded their
free "Advanced Real-Time Chart" widget (`tradingview.js`, new file) as a
second chart source on the ticker deep-dive page, toggled via a "MSV
Chart / TradingView" switch above the Price Chart card. No API key or
signup needed — TradingView supplies its own data feed, so it's zero cost
against Finnhub/Twelve Data quotas. As a side benefit, crypto tickers
(Finnhub format `BINANCE:BTCUSDT`, which matches TradingView's own
`EXCHANGE:PAIR` syntax) now get a working chart for the first time — the
custom chart.js chart has never supported crypto candles.

**License constraint, important if $MSV ever monetizes:** confirmed via
TradingView's own terms of service — the free widget requires its
attribution bar to stay visible (enforced with bans/legal action) AND
restricts free use to **non-commercial** sites: "we do not permit
commercial usage of any of our services or APIs [without] separate
agreement." Fine today since $MSV has no subscriptions/ads. If that
changes, this needs either a paid TradingView agreement or falling back
to the in-house chart.js chart (zero licensing risk, already built).
Documented in `tradingview.js`'s file header and the UI's own disclosure
text so this isn't forgotten later.

Verified before deploying: `node --check`, the existing Playwright smoke
test (passes), and a manual screenshot check of both toggle states.

## Stock Screener MVP shipped — 2026-09-30, deployed (`msv-web` e52483f)

Jozsua asked for a comprehensive, Webull-style screener as part of making
$MSV "a complete package." Built the MVP version scoped in TODO.md the
same day: a new Screener page/nav item (`screener.js`) with filters for
category, market cap bucket, price range, today's up/down, and max P/E,
sortable columns, click a row to open the real ticker page. This flips
the existing "Stock Screener" Coming Soon placeholder to live.

**Deliberately not a whole-market screener** — reuses the app's existing
curated ~70-stock universe (`BROWSE_CATEGORIES`' four stock categories,
deduped), same constraint noted in root `CLAUDE.md` since project start:
Finnhub's free tier has no bulk screener endpoint and caps at 60 calls/
min shared across every visitor. Each ticker needs 3 calls (quote,
metric, profile2 for market cap — added `fetchProfileCached` to
`dataUtils.js`, 24h TTL). Queued as individual jobs rather than bundled
per-ticker behind `Promise.all`, so the throttle (concurrency 2, 1.2s
gap) governs the real call rate directly instead of tripling it. Renders
progressively (skeleton with "--" appears immediately, rows fill in as
data lands) rather than blocking on a spinner — verified this actually
works via a manual functional test with mocked Finnhub responses,
including confirming the P/E and market-cap filters correctly narrow
results against realistic data shapes before deploying.

**Upgrade path, not yet built:** a broader, real whole-market version is
possible if Financial Modeling Prep's Stock Screener endpoint turns out
to be genuinely free-tier accessible — this project has been burned by
FMP's docs overstating free access before (the ETF endpoints), so that
needs a live test with a fresh key before it's trusted. Tracked in
TODO.md, waiting on Jozsua to provide a key.

## React migration Phase 1 shipped — 2026-09-30, deployed (`msv-web` 39ce4c3)

Jozsua approved starting the React/TypeScript migration plan and asked to
begin with Phase 1: prove a React page can actually ship to production,
before migrating any real page's functionality. Done — live at
[msv-web.jozsua-heng.workers.dev/react-crypto/](https://msv-web.jozsua-heng.workers.dev/react-crypto/),
linked from the real Crypto page as a clearly labelled beta.

**How it works:** `react-poc/`'s `npm run build` now outputs to a new
sibling folder `msv-web/react-crypto/` (`vite.config.ts`'s `outDir`),
instead of the default `react-poc/dist/` which `.assetsignore` excludes
wholesale (source, `node_modules`, config — none of that should ever be
public). `base: "/react-crypto/"` makes the built `index.html`'s asset
URLs resolve correctly from that path. `react-crypto/` is gitignored, a
build artifact like any other — **there's no CI/CD auto-build**, `npm run
build` must be run by hand before `npx wrangler deploy` any time
`react-poc/` changes and the live beta should reflect it (documented in
`react-poc/README.md`'s new "Deploying it" section).

**Verified before calling this done** (same discipline as every other
deploy-safety check in this project): a real browser check confirmed live
data renders with zero console errors, AND `react-poc/`'s actual source/
config/`node_modules` were confirmed still NOT publicly fetchable
afterward — checked the response *body*, not just status code, since the
SPA fallback returns 200 for literally any unmatched path.

Also added: a `react-build` CI job (typecheck + `vite build` on every PR,
build-only so no live API key is needed) alongside the existing
`syntax-check`/`smoke-test` jobs, so a broken React build fails before
merge instead of only being discovered at deploy time.

**Next: Phase 3, page 1 — port real Crypto functionality into this
pipeline.** The POC already exists; the work here is making it the *real*
Crypto page (not just a linked beta) and retiring `crypto.js`. Not started.
Full page-by-page order after that: Sectors → ETFs → Screener → Market
Data → Market Intelligence → Learn → Home → the ticker deep-dive page
(biggest/riskiest, last) → finally the sidebar/router shell itself.

## TradingView Screener + FMP ETF profiles + FMP usage bar — 2026-09-30, deployed

Jozsua provided a fresh FMP API key and asked for three things: embed the
TradingView Screener widget, use FMP to build out more ETF data if
possible, and add an FMP usage bar to the sidebar alongside the others.

**Live-tested the fresh key first** (`msv-api` cdddf72 for the backend
route, `msv-web` 3098ccf for the frontend): `/stable/company-screener`,
`/stable/etf/holdings`, and `/stable/etf/info` all still return
"Restricted Endpoint" — same paywall as the 2026-09-21 test, confirmed
again rather than assumed carried over. `/stable/profile`, however,
genuinely works for any ticker (also true back on 2026-09-21, per
BLOCKERS.md, just never acted on) — real fund name, a genuinely fund-
specific description, website, ISIN/CUSIP, beta.

- **TradingView Screener widget** — a "MSV Screener / TradingView" toggle
  on the Screener page, same pattern as the chart toggle but a different
  embed mechanism (TradingView's newer self-initializing `embed-widget-
  screener.js`, config as a `<script>` tag's JSON text content, not a
  constructor call — config keys confirmed from the widget's own loader
  script before using them). `market: "america"` gives real whole-market
  US stock coverage, the practical answer once FMP's screener endpoint
  was confirmed blocked.
- **FMP ETF profiles** — `fetchFmpEtfProfile()` (`msv-web/script.js`)
  calls the new `/api/fmp` proxy route (`msv-api/worker.js`, `FMP_API_KEY`
  Cloudflare secret, 24h cache given FMP's tight 250/day free budget) for
  ANY ETF ticker, not just the ~60 in the curated `ETF_FUND_INFO` list.
  Real description takes priority over the Wikipedia-issuer-fallback
  (skips that fetch entirely when FMP succeeds); real website/ISIN shown
  as new facts. The curated list + Wikipedia fallback stays in place for
  when FMP has no key or fails for a given ticker — not replaced, just no
  longer the only source. Does NOT unlock NAV/AUM/expense ratio/holdings/
  sector weighting — those stay exactly where they were (parked).
- **FMP usage bar** — a new "FMP (daily) x / 250 per day" row in the
  sidebar's API usage panel, reusing the existing generic table-driven
  renderer and the daily-count `localStorage` pattern already built for
  Twelve Data's 800/day cap — no new UI code needed beyond one config row.

**Verified before deploying:** `node --check` on every changed file, the
existing Playwright smoke test, two manual functional tests with mocked
Finnhub/FMP responses (confirmed the ETF page renders FMP's real
description/ISIN/website, and the sidebar usage panel shows the new FMP
row), and — since this sandbox can't reach TradingView's real CDN to
render a full visual proof locally — a direct DOM check confirming the
Screener widget's script tag is constructed exactly right (correct `src`,
correct JSON config keys). Both pieces then re-verified against the real
deployed production site with a live browser: the ETF page shows a real
FMP-sourced description/ISIN for QQQ, and the TradingView Screener
widget's toolbar (Overview/Filters/General dropdowns) renders live —
zero console errors either way.

## Chart honesty fix + layout density pass — 2026-10-01

Jozsua sent a long list of feature requests for $MSV; the first batch
tackled was the quick, low-risk layout items plus one real bug, chosen
together so the session wouldn't stall on the bigger unknowns (Polymarket/
congressional-trading data, which need their own feasibility research
first — tracked separately below).

**Found while investigating "the 1D/3D/7D ranges look greyed out and
unusable" (Jozsua's report, imprecise range names but a real bug
underneath):** screenshotted the live site's chart at every range before
touching code (`msv-web` `chart.js`/`index.html`/`style.css`). Two real
problems, not one:

- **1D/4H wasted roughly half the chart as dead space.** The axis was
  deliberately extended to the full 4am-8pm session window so there'd be
  room to shade "this is when pre/after-hours would be" bands — but
  Twelve Data's free tier has no real data in those windows, so the bands
  were empty, and on a real screen the actual price line was squeezed
  into a sliver of the chart. Jozsua's "unusable" report is correct: a
  disclosure feature that eats half the chart for zero information isn't
  worth it. Removed the axis extension and the shading entirely — 1D/4H
  now bound tightly to the real data like every other range, replaced the
  swatch legend with one plain sentence disclosing the same free-tier
  limitation without taking up chart space for it.
- **1W's x-axis labels showed time-of-day only ("9:30 AM" … "3:55 PM"),
  no date** — `formatChartDate()` decided time-vs-date formatting off a
  raw `series.intraday` flag, true for 1W's 30-minute bars same as any
  single-session range, but 1W actually spans 5 different days. Every
  label looked like it belonged to the same single day. Fixed by reusing
  the same "single session vs multi-day" distinction `getXMapper` already
  computed for its own axis-positioning decision — 1W labels now show
  real dates (Sep 24, Sep 25, Sep 28…).

**Also shipped in the same pass** (the "quick wins" Jozsua asked for
first): the in-house price chart canvas grown 340px→460px (sub-panels
scaled up to match) since TradingView's embedded widget next to it was
already 500px and made the in-house one look small by comparison; sidebar
width 264px→220px for a denser, Seeking-Alpha-style nav; the page-wide
`--page-zoom` variable (the same knob used for the 2026-09-21 "+10%
bigger" request) taken from 1.1→1.045 for the "decrease font size 5%"
ask — the only global scale control this page has, so it moves spacing
and images proportionally too, not just type.

**Verified before calling it done:** real production screenshots (before)
confirmed the actual bug visually rather than just from reading the code;
a local Playwright harness with mocked Twelve Data responses shaped like
5 real trading days (after) confirmed both fixes render correctly —
full-width price line on 1D/4H, real per-day dates on 1W — plus
`node --check` on the edited JS. Not yet deployed to production; Jozsua
reviews before a `wrangler deploy`.

**Still open from Jozsua's same request list** (deliberately not started
this session): Polymarket/prediction-market data, a Burry/congressional-
trading-style holdings page (needs feasibility research first, same
discipline as the options/bonds research in API_RESEARCH.md), IPO
calendar + expanded news categories, ETF-by-issuer grouping, world-map
country-dropdown-as-default, bigger sector ETF popups, more homepage/
screener/stock-page content, and a Learn-section stock-picking guide.

## Low-hanging-fruit batch: screener columns + more browse tickers — 2026-10-01

Second pass the same day, after the chart fix/layout batch — Jozsua asked
to start on the "not started" items, low-hanging fruit first.

- **Screener: 5 new columns** (52-Week High, 52-Week Low, Beta, Dividend
  Yield, Avg Volume 10-Day) — genuinely free, since `ensureScreenerData()`
  already fetches a full `/stock/metric` response per ticker and these
  fields were just sitting there unused. No new network calls, no slower
  page load.
- **BROWSE_CATEGORIES grown again** (`home.js`) — each of the 7 homepage
  categories gained 4-6 more real tickers (Trending Tech/Blue Chip/
  Dividend Payers/Growth +6 each, ETFs +6, Bond ETFs/Commodities +4
  each), same zero-cost-at-list-stage pattern as the 2026-09-19 12→18
  bump. **Every new ticker was live-verified against the real production
  `msv-api` proxy before being added** (`/stock/profile2` for stocks —
  real company name came back for all 24; `/quote` for ETFs — real
  nonzero price for all but one) — one candidate (NIB, cocoa) came back
  all-zero/invalid and was swapped for WOOD (timber & forestry) instead,
  confirmed live. Same discipline as sectors.js/etfs.js's curated lists,
  done because this project has a real history of delisted/renamed
  tickers slipping into hand-written lists (EA, CMA, MRO, CTRA, ABB, SQ→XYZ,
  FI→FISV, all documented in etfs.js's/sectors.js's own history).
- **Side effect, documented not hidden:** the Screener's universe grew
  ~70 → ~94 tickers (it draws from 4 of these same categories), so its
  cost estimate updated too (~210 → ~282 Finnhub calls for a cold first
  load) — same throttle protects it, just a slightly longer first load,
  not a new risk category.

Not deployed yet, same as the earlier batch today — Jozsua reviews
before `wrangler deploy`.

## ETF By-Issuer view — 2026-10-01

Third pass the same day, continuing down the "low-hanging fruit" list.
Jozsua's exact ask: "I know Vanguard has VOO/VOOG, Schwab has
SCHG/SCHB/SCHD, JPMorgan has JEPI/JEPQ — I want this visualised in a
simple to see and understand and intuitive way."

Added a "By Category" / "By Issuer" toggle to the ETFs page (`etfs.js`).
Issuer is **derived from each fund's own name string** rather than
hand-tagging all ~290 tickers a second time — this file already writes
names consistently as "Vanguard X", "iShares Y", "SPDR Z" (confirmed by
inspection), so the by-issuer view can never drift out of sync with the
by-category data above it. A sanity check script (run before calling this
done, not just trusted by eye) confirmed **222 of 256 unique tickers
(87%) matched to a real issuer**, sorted into 15 issuer cards each with a
short blurb (Vanguard 32 funds, BlackRock/iShares 72, State Street/SPDR
32, Schwab 6, JPMorgan 3, Invesco 19, and 9 smaller ones). The unmatched
13% are smaller/niche issuers (Alerian, Amplify, abrdn, Sprott,
KraneShares, United States Commodity Funds, etc.) — deliberately left out
of this view rather than guessed at or mislabeled. One small override
was needed: the 11 S&P sector SPDRs are written as short names
("Technology", not "SPDR Technology") in their own category since that
category's blurb already says SPDR — handled with an explicit
ticker-level override rather than changing those existing display names.

Verified with a live browser check (not just reading the code): the
toggle switches views, all 15 issuer cards render with correct counts,
clicking an issuer (tested Charles Schwab) opens the same dense
quotes-table component used everywhere else in the app.

Not deployed yet, same as the day's earlier batches — Jozsua reviews
before `wrangler deploy`.

## Sectors: "other ETFs tracking this market" — 2026-10-01

Fourth pass the same day. Jozsua's original ask, back when he first saw
the Sectors page: "currently one ETF is representing one sector or
subsector — upon clicking it, can it show the top ETFs in that market?
Oh, it opens up below, so that's great, let's display more in the
industry."

Added a new section to the sector/industry detail panel (`sectors.js`,
`renderSectorDetail`), right below the existing price/performance block:
"Other ETFs tracking this sector/industry." Rather than hand-curating a
second ETF list per sector, it cross-references `etfs.js`'s
`ETF_CATEGORIES` for other funds already tracked on the same theme —
e.g. clicking Technology (tracked by XLK) now also shows VGT, FTEC, IYW,
SMH, SOXX, IGV, FDN, each with a live price. Verified the match rate with
a standalone data script before touching the UI: 60 of 63 sector/industry
entries (95%) found a matching multi-fund category; the 3 that don't
(Communication Services sector itself, the Entertainment and Autos
industries) just don't show the section, rather than showing something
thin or wrong.

One real bug caught by that same verification pass, before it shipped:
the naive version matched the `sec-spdr` category first for every S&P
sector (since that category also contains XLK etc.), which would have
shown "other ETFs" that were actually just the other 10 unrelated S&P
sectors' SPDRs — not useful, and not what was asked for. Fixed by
excluding that one category from this specific lookup.

**Explicitly not ranked by market cap or AUM** — Jozsua's original
phrasing asked for that, but this project confirmed back in September
that AUM/assets are paywalled on every free data source checked (see
BLOCKERS.md), so the list is disclosed as "this app's own curated order,
not a ranking" rather than faking a sort order off data that isn't
actually available.

Verified with a live browser check: clicking the Technology sector tile
renders the new table with 7 real alternative ETFs, correct live
prices/%, no console errors.

Not deployed yet, same as the day's earlier batches — Jozsua reviews
before `wrangler deploy`.

## Feasibility research: Polymarket + "who's holding what" — 2026-10-01

Fifth pass the same day, moving into the two items from Jozsua's
2026-10-01 list that explicitly needed research before any UI: live
prediction-market odds, and a page showing what public figures
(Michael Burry, Nancy Pelosi — his named examples) hold and trade. Full
detail in [API_RESEARCH.md](API_RESEARCH.md); the short version, three
separate verdicts since these turned out to be three different data
problems:

- **Polymarket: clear to build.** Both the Gamma API (market metadata)
  and CLOB API (live prices/order book) confirmed free, CORS-enabled
  (no msv-api proxy needed — callable straight from the browser, same
  tier as Finnhub/Twelve Data today), and generously rate-limited, all
  verified with live requests against the official docs, not a
  third-party summary. Real finance-relevant markets confirmed live,
  e.g. two live Fed-rate-decision markets with ~$700k+ 24h volume each —
  a natural fit next to the existing Macro/FRED tab.
- **Michael Burry (13F filings): buildable, real work.** SEC EDGAR
  confirmed free and official, tested directly against Scion Asset
  Management's actual filings (real holdings came back: Halliburton
  calls, Lululemon, Molina Healthcare, NVIDIA). The submissions API is
  CORS-enabled; the actual holdings documents are not (sit behind
  Akamai bot protection) and 13F uses CUSIP identifiers, not tickers —
  needs two new proxy routes (SEC Archives, OpenFIGI for CUSIP→ticker)
  plus real XML parsing. Scoped as its own project if greenlit.
- **Nancy Pelosi (congressional trading): no good free source, don't
  build yet.** House Stock Watcher and Senate Stock Watcher — the
  standard free answer for this, cited as current in several 2026-dated
  blog posts — are both confirmed **dead** (DNS failure, not just down).
  Quiver Quantitative has no free API tier ($30/month minimum). A few
  smaller vendors claim free tiers but are unestablished or too limited
  to be useful. Logged as a new entry in
  [BLOCKERS.md](BLOCKERS.md) rather than left unresolved in chat.

**Discipline note worth keeping:** every claim above was checked with a
live `curl`/API call before being written down, including ones that
contradicted what search results and blog posts said (the two Stock
Watcher sites especially) — consistent with this project's standing
"confirmed directly, not assumed" rule for data-source claims.

## Prediction Markets page shipped; community/Reddit page researched and declined — 2026-10-01

Sixth pass the same day. Jozsua gave the explicit go-ahead to build
Polymarket (researched earlier the same day with no blockers found),
asked for a feasibility check on a subreddit-sourced "community page,"
and clarified the Burry research was meant generally ("notable figures,"
not just him).

**Prediction Markets page shipped** (`predictionMarkets.js`, new sidebar
nav item, new `ROUTES`/`EXPLORE_DIRECTORY` entries in home.js): live
odds from Polymarket's Gamma API, 6 tabs (Trending, Finance, Economy &
Fed, Crypto, Business, Politics) via `tag_id` filtering. One real bug
caught during the build: `tag_slug` (the parameter name several
third-party guides use) is **silently ignored** by the live API —
tested directly, it returned the same unfiltered top-by-volume results
regardless of the slug passed. Switched to `tag_id` (an undocumented-
by-slug-name integer), found by paginating Polymarket's full 2,100+-tag
list and verified live per tag before trusting it. No msv-api proxy
needed — confirmed CORS-enabled earlier the same day, same tier as
CoinGecko. Verified with a live browser check against the real API (not
mocked): 24 real markets render per tab, including genuinely relevant
ones like live October 2026 Fed-meeting rate-decision odds with
$400K+ 24h volume. Not deployed yet.

**Community/subreddit page: researched, declined for now.** Checked
Reddit and Stocktwits (see API_RESEARCH.md's "Community/sentiment data"
section and the new BLOCKERS.md entry) — this isn't a "no free tier"
situation like congressional trading, it's "the platform is actively
closing." Reddit announced literally the day before this research that
it's shutting down RSS (Nov 13, 2026) and the entire public API (March
2027) in favor of paid AI-licensing deals. Stocktwits' official
developer program has new registrations closed indefinitely. Recommended
the same pattern already used for the Crypto page's "Regulation &
Adoption tracker" instead — periodic hand-curated research, not a live
feed — if community content is still wanted. Also flagged that
"summarize" specifically would need a real LLM API call (real cost), a
different category of spend than anything else in this app.

**13F/"notable figures" research note:** Jozsua clarified Burry was one
example, not the target — added a short note to API_RESEARCH.md stating
explicitly that the researched mechanism (SEC EDGAR + CUSIP mapping)
already covers any manager with a public CIK, so this should be scoped
as "a curated list of tracked managers" when built, not a Burry-specific
feature.

## Page-zoom reduced further; React migration Phase 3 scoped and started — 2026-10-02

Jozsua asked for the whole site scaled down "a little more." Reused the
existing `--page-zoom` variable (`msv-web/style.css`) — the same knob
behind the 2026-09-21 (+10%) and 2026-10-01 (-5%) adjustments — taken
from 1.045 to 0.95. Shipped directly (`msv-web` 1651f7f).

Also confirmed the Phase 3 React migration scope and pacing with
Jozsua: one page at a time — built, reviewed by Jozsua in the browser,
then committed and pushed — rather than attempting the whole migration
in one session (flagged up front as session-usage-limit risk, per
Jozsua's standing instruction). Order is the one already on record
above: Crypto first (port real functionality into `react-crypto/` and
retire `crypto.js`, rather than leaving it as a second parallel page to
the beta), then Sectors → ETFs → Screener → Market Data → Market
Intelligence → Learn → Home → ticker deep-dive → sidebar/router shell
last.

## React migration Phase 3, page 1: Crypto shipped — 2026-10-02

Same session as the page-zoom change above. Built and deployed the first
real page migration: `react-crypto/` (the React + TypeScript rebuild
that had been sitting live as a linked beta since 2026-09-30) became the
actual Crypto page, and `crypto.js` — the old vanilla implementation —
was deleted.

**What changed in `msv-web`:** `home.js`'s `ROUTES` table had three
entries pointed at crypto — `crypto`, `bitcoin-cycles`, `crypto-news`
(the latter two previously "Soon" placeholders, per the 2026-09-27 batch
entry above). All three now do a real `window.location.href` navigation
to `/react-crypto/` (`?tab=cycles` / `?tab=news` for the placeholders)
instead of calling the old `renderCryptoPage()` or `showPlaceholderPage()`
— a genuine browser navigation, not an in-app SPA route change, since
`react-crypto/` is its own separate static build with its own routing
and can't be slotted into the vanilla site's single-page-app view
switching. Confirmed via `grep` that nothing outside `crypto.js` called
any of its functions before deleting it; `node --check` run against
every remaining `.js` file afterward to catch any syntax fallout.
Sidebar: the "Soon" badges came off Crypto Cycles/News, and the
`#cryptoPageCard`/`#cryptoRoot` markup (dead once nothing rendered into
it) was removed from `index.html`.

**What changed in `react-poc`**, now that this is a real destination
people land on directly rather than a labelled experiment opened in a
new tab: the "React + TypeScript beta" banner became a "← Back to $MSV"
link (there's no shared sidebar between the vanilla site and this
separate React build yet, so without this link a visitor would have no
way back except browser-back); the page `<title>` dropped "(React
beta)"; and the theme toggle was switched from its own `poc-theme`
localStorage key to the main site's actual key
(`stockDashboardTheme`, matching `script.js`'s `THEME_KEY`) so a
visitor's light/dark choice carries over between the two instead of
flipping on every navigation — needed a small hand-written
read/write instead of the generic `useLocalStorage` hook, since that
hook JSON-stringifies values and the main site stores the theme as a
plain unquoted string.

**Verification, same discipline as the `.assetsignore` story**: `npm run
typecheck` and `npm run build` both passed; started a local static
server over the whole `msv-web` root (not just `react-poc`'s own dev
server) so the real end-to-end click path could be checked — sidebar →
real navigation → built `react-crypto/` bundle → back link — before
asking Jozsua to review it himself in a browser (no browser tool
available this session to check it directly). After Jozsua confirmed it
looked right, committed/pushed, then `npx wrangler deploy`, then
re-verified against the **live** production URLs: `/react-crypto/`'s
title, `/crypto.js` now falling through to the SPA shell instead of
serving real content, and the sidebar HTML having zero remaining
`crypto.js` references and no leftover "Soon" badges — response
*bodies* checked, not just status codes, since `not_found_handling:
"single-page-application"` makes everything return 200 regardless.

**Known gap, not fixed in this pass:** the React page still has no
sidebar/nav of its own, so leaving it for anywhere else on the site
means using the back link first. Fine with only one page migrated;
likely needs addressing once a few more pages exist and a shared app
shell is worth building — noted in `msv-web/CLAUDE.md`.

**Next up: Sectors**, per the already-recorded page order (ETFs,
Screener, Market Data, Market Intelligence, Learn, Home, ticker
deep-dive page, sidebar/router shell last).

## Homepage low-hanging-fruit pass: FX strip shipped, brainstorm list corrected — 2026-10-02

Jozsua asked to build out the cheapest/easiest items from the 2026-09-19
homepage brainstorm list (TODO.md), flagging he was running low on his
weekly token budget — a cue to move efficiently rather than re-explore
broadly.

**First finding, before writing any code:** checked each brainstormed
item against the actual live site rather than assuming the list was
still accurate, and 5 of the 6 cheap-tier items turned out to already be
built — the earnings calendar, economic calendar, a real watchlist, the
sector heatmap, and a "Did You Know" rotating fact (the last one, in
fact, better than originally envisioned: it pulls real Learn-hub lesson
content, deterministic by date, rather than raw indicator-tooltip
definitions). None of these had ever been checked off this particular
list, even though they shipped in earlier sessions — the brainstorm
section was simply stale. Corrected it in the same commit as this entry
so a future session doesn't spend tokens re-discovering the same thing.

**Only genuinely unbuilt item: the Currency/FX strip.** Built it:

- 6 major pairs (EUR/USD, GBP/USD, USD/JPY, USD/SGD, USD/AUD, USD/CHF),
  via Twelve Data's `/quote` endpoint — **confirmed live against the real
  production `msv-api` proxy before writing any front-end code** (not
  assumed from the 2026-09-24 single-pair test): one request with all 6
  symbols comma-separated returns one object keyed by symbol instead of
  a flat quote, so the whole strip costs exactly one API call, same
  zero-extra-cost pattern as every other homepage card. Finnhub's own
  forex coverage remains zero (unchanged).
- Reused the existing `.index-chip` styling (the same chips the World
  Markets strip already uses) rather than writing new CSS.
- **Real script-order bug caught before shipping:** `twelveDataUrl()`
  lives in `chart.js`, which loads *after* `home.js` in `index.html`'s
  script order (see `msv-web/CLAUDE.md`'s "script order matters" note —
  previously bit `crypto.js` the same way). Calling it synchronously
  inside `initHome()` would have thrown `ReferenceError` partway through
  that function and silently skipped everything after it, including the
  sidebar wiring (`initHomeLayout()`) — i.e. it would have broken
  homepage navigation entirely, not just the new card. Fixed with a 0ms
  `setTimeout`, which runs after the synchronous script-loading phase
  finishes.
- **Verification, no browser tool available this session:** syntax-
  checked every `.js` file, then validated the actual render/formatting
  logic in Node against a real captured API response (not fabricated
  test data) before shipping — confirmed sensible output for all 6
  pairs, including JPY correctly getting 2 decimal places instead of 4
  (price scale differs enormously from the other pairs). Checked the
  built markup was present in the served HTML, both locally and against
  the live production deploy afterward.
- Shipped directly to production (committed, pushed, `wrangler deploy`)
  given the stated time/token pressure, rather than pausing for a
  browser-review checkpoint the way the Crypto page migration did.

**Deliberately not attempted**, despite being on the same brainstorm
list: Trending/most-searched tickers (needs real shared-backend state,
the one item on the list that was never actually "cheap") and homepage
investor graphs (still not scoped — no homepage-native time-series data
exists yet to chart). Both left open in TODO.md.

## Bug check on today's changes: 3 real findings, all fixed — 2026-10-02

Jozsua asked for a comprehensive health check before moving on. Given
the stated token-budget pressure, scoped it down with him first rather
than attempting an exhaustive pass over the whole ~500KB `msv-web`
codebase: medium-depth, `msv-web` only (not `msv-api`, which had no
changes today). Found 3 real, high-confidence issues, all in the FX
strip just shipped:

1. **`renderForexStrip()` could render a literal "NaN%"** — if a pair's
   `close` price was valid but `percent_change` was missing/non-numeric,
   the old code still ran `pct.toFixed(2)` on `NaN` unconditionally.
2. **Stale, contradictory copy**: the Explore Products "Forex"
   placeholder still said "not built yet" even though the new homepage
   Currency strip had gone live in the same session — a user could see
   real FX quotes, then click through to a placeholder claiming they
   don't exist.
3. **Duplicated chip-formatting logic** between `loadMarketTickers()` and
   `renderForexStrip()` — the actual root cause of #1: the same
   positive/negative/muted-and-percentage logic existed in two places,
   so a correct version in one didn't help the other.

**Fix**: extracted a shared `indexChipValue(formattedValue, pct)` helper
(`home.js`) used by both chip-building call sites — degrades gracefully
now (shows just the price with no percentage if `pct` is bad, "···"
only if there's no price at all) instead of fabricating "NaN%". Updated
the stale Forex placeholder copy to accurately describe what now exists
(a homepage snapshot) vs. what doesn't (a full dedicated page, like
Sectors/ETFs have). Verified with `node --check` on every file plus a
standalone Node test of the new helper against normal/missing-percent/
missing-price/both-missing cases. Shipped directly (committed, pushed,
`wrangler deploy`).

## Codebase quality pass: Macro tab extracted, one finding deliberately skipped — 2026-10-02

Jozsua asked how the codebase looked "in the eyes of an SWE." Given the
standing budget concern, did a single-pass review (not the `/simplify`
skill's normal 4-parallel-agent process) across reuse, simplification,
efficiency, and altitude.

**Clean overall**: no leftover `console.log`/`debugger` statements, no
TODO/FIXME markers, and every feature except one follows the codebase's
own one-file-per-feature convention.

**Fixed — `home.js` was the one outlier.** At 2,019 lines (vs. 917 for
the next-largest, `chart.js`) it had accumulated routing/shell code
*and* the ~425-line Macro tab (World Bank/FRED country indicators)
together. Extracted the Macro tab into its own `macro.js` (pure move, no
logic changes) — `home.js` is now 1,582 lines. Placed `macro.js` before
`marketData.js` in `index.html`'s script order, since `marketData.js`
already calls `fetchWorldBankIndicator()`/reads
`WORLD_BANK_GOVERNANCE_INDICATORS` (both now in `macro.js`) from its
country-governance panel — safe either way since that's a call-time
reference, not load-time, but matched the dependency direction anyway.
Verified: `node --check` on every file, confirmed `macro.js` serves
correctly both locally and on the live production deploy, confirmed
`home.js` still calls `renderMacroTab()` correctly post-split. Shipped.

**Deliberately skipped — the throttled-fetch duplication.**
`dataUtils.js` has a `runThrottled()`/`loadQuotesThrottled()` helper
built specifically so "any page that loads many tickers must go through
this, not raw fetch" (its own doc comment) — but `home.js` hand-rolls
the same staggered-fetch pattern 3 separate times
(`ensureBrowseQuotes`, `ensureRankingLoaded`, `loadSectorHeatmap`, each
with a different stagger gap: 45ms/30ms/40ms) instead of using it.
Looked carefully at whether this was safely swappable before touching
anything: it isn't a clean drop-in. The shared helper caps true
concurrency at 3 requests with 180ms gaps between each worker's calls;
the hand-rolled versions just stagger *start* times with no concurrency
cap, a materially different request-timing/rate-limit profile against
Finnhub's 60/min budget. `ensureBrowseQuotes` additionally has its own
in-flight-promise dedup and a different cache shape than what the
shared helper assumes. Consolidating these for real would mean changing
actual loading behavior on 3 different pages, not just tidying code —
left alone rather than forced, flagged here for whoever next touches
this area to decide on purpose rather than by accident.

## 2026-10-03 — CI repaired; Sectors rebuilt in React

**CI was failing on `main` since Oct 2.** The Playwright smoke test caught
two real load-time bugs in the site, both introduced by the Oct 2 homepage
work: the World Bank indicator list in `macro.js` called a function that
lives in a file loaded later, and the new forex strip was deferred with a
zero-delay timer that could fire before `chart.js` had downloaded. Both
fixed (commit `4276a9b`), CI green on that commit. The test working as
intended is the point: it stopped a broken homepage from merging.

**Sectors page rebuilt in React** (`react-poc/`, served as `?page=sectors`
on the React build, not yet deployed or linked from the live sidebar).
Ports the vanilla Sectors page: all 11 sectors and 52 industries/themes as
colour-coded tiles, a click-to-open detail panel with price, performance
vs the S&P 500, what drives it, related ETFs with YTD / 1Y / beta / volume
columns, and representative companies. Live data goes through a queue that
stays under Finnhub's 60-calls-per-minute free limit. AUM is not shown:
fund size is paywalled on every free source the app has checked.

---

*Add new phases here as they happen, most recent last — this is meant to
stay current, not be a one-time snapshot.*

## Explore page matched to the old page; theme fixed on every React page — 2026-10-03, deployed (`msv-web` e8a5bd7)

Jozsua asked for every page to look like the old version, and asked why the React
pages looked so much worse. Found two causes on the Explore page:

- The light/dark choice was only applied by the Crypto page. Every other React page
  ignored the saved choice and showed the default theme, so they looked nothing like
  the old site. Now `main.tsx` applies the saved choice (the same key the old site uses)
  before anything renders, on every page.
- The Explore markup had its own header instead of the old page's card, heading and
  intro copy. It now matches the old structure. The sidebar also failed to highlight
  "Explore Products" (wrong key); fixed.

Compared against the old page at 1440 and 1920 px. Tile descriptions now run full
width. Deployed; live `/app/?page=explore` shows all 53 tiles with no page errors, and
the live homepage still loads.

Still different from the old page, on purpose or not yet decided: the React pages
have a top navigation strip the old page doesn't; the React sidebar is wider than the
old one (so labels aren't cut off); the old page's light/dark toggle isn't on the React
pages yet; and a few "Coming soon" badges still overlap long titles.

## Docs saved — 2026-10-03

`msv-web/CLAUDE.md` session status updated with the open bugs (Explore icon fix,
`index.html` guard, `home.js` dead route) and committed to `msv-web` (`b66822d`).

## CI repaired, live outage fixed, homepage rebuilt, site chrome shared — 2026-10-03 to 2026-10-04, deployed (`msv-web` 18a3ee6)

**CI emails.** Every push to `msv-web` main had been failing its Playwright smoke test
since the React app became the main site on 2026-10-03. The test still looked for the
retired vanilla homepage (`#homeView`). Also, the React build (`app/`) is gitignored, so
CI had no `/app/` to test. Fixed in `58a1d92`: the test checks the root redirect into
`/app/` and a rendered AAPL ticker page, and the smoke job builds `react-poc` first. CI
green on `58a1d92` and `18a3ee6`.

**Live site down.** `/app/` addresses were serving the redirect page, and the redirect
script threw before the page loaded. Two causes: the Cloudflare edge cache, and a guard
that touched `document.body` from `<head>`. Guard fixed; site redeployed and verified in
a headless browser.

**Accidental publish.** A deploy published `tests/smoke.spec.js` publicly because
`.assetsignore` didn't exclude `tests/`. Excluded and redeployed; the file no longer
serves. No secrets were in it.

**Homepage rebuilt in React** (`react-poc/src/components/HomePage.tsx` and new `Home*`
components):
- Markets today: intro with a live sentence (index and strongest/weakest sector moves,
  from the same quotes), and a sign-up card with Google and Apple placeholders.
- Search bar for tickers, placed above the intro. Topic search isn't built yet.
- Market news: 15 headlines in three compact columns, from Finnhub's general feed.
- Index strip: eight US index funds in one row, each checked live first.
- Global markets: country bubbles, the map (hover highlights and a popup at the cursor),
  the breadth bar, and a country list with a card showing hours, live index price, day
  range, and World Bank GDP growth, inflation and unemployment.
- Browse by category: 13 categories, with the sector-level ones checked live.
- Crypto: 14 coins with rank, 1-day, 7-day and 30-day changes.
- Sectors: tiles with move bars, and an Industries & themes view that loads on demand.
- Earnings: widened to companies with at least $1bn estimated revenue, up to 80 rows.
  Still a Finnhub estimate list, not a full week.
- Economic calendar: unchanged. No new dates were added; they need checking against the
  official schedules first.
- Currencies: 12 pairs, loaded in two batches (see the Twelve Data entry in BLOCKERS.md).
- Market intelligence card: the AI supply chain plus all 12 market categories with their
  layers and example companies (static data, not live).
- Learn: numbered cards with no emoji on the homepage.
- Explore $MSV: every live page grouped by category, with coloured icons and descriptions.
- Your lists, Did you know, How to use $MSV: redone as a two-column row, with a five-step
  how-to in numbered cards.
- Footer: the sidebar's logo lockup, a sitemap, socials and a legal row. All links are
  placeholders.

**Shared chrome across every React page.** A frozen ribbon at the top of every page: a
local-time pill and a market pill on the left (the market defaults to the US and can be
changed; the choice is remembered), and recommended links, the bell, the light/dark
toggle, Log in and, on phones, a menu button on the right. The search bar and footer are
also on every page. Panels are white cards with grey gaps between them.

**Responsive.** Checked at 1920, 1024 and 390 px with no horizontal overflow. On phones
the sidebar becomes a slide-out menu opened from the ribbon.

**Placeholder AI.** The sidebar's "Ask $MSV AI Anaiyst" item (working name; the spelling
is intentional) opens a placeholder page listing what it will include: plain-English answers
using live numbers, sector and market questions, sources for every answer, limits with no
buy or sell advice, and follow-up questions.

**Deployed.** `msv-web` commit `18a3ee6`, Cloudflare version `7f4cd850`. Live homepage
loads in a headless browser with no page errors. CI green.

## Current status — 2026-10-04 (recap)

**Built and live:** the React homepage described above, the frozen ribbon, search, footer
and card layout on every React page, responsive layout, the placeholder AI page, and the
Screener, ETFs, Sectors, Market Data (pending its own deploy, see below), IPO, News,
Prediction Markets, Crypto, Learn and ticker pages in React.

**Built, not yet deployed:** Market Data in React (`?page=market-data`). The sidebar still
points at the vanilla page for it.

**Still to port to React:** Market Intelligence, Learn, the ticker deep-dive page, and the
sidebar/router shell.

**Blocked:** AUM and expense ratio (paywalled everywhere checked). Currency pairs are
rate-limited on Twelve Data's free plan (see BLOCKERS.md).
