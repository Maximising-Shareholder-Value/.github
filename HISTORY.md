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

## Homepage overhaul — scoped, not yet started (2026-09-22)

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

**Not yet started.** This is a genuinely large, structural piece of work
on top of everything shipped today — flagged to Jozsua as needing its
own dedicated pass rather than being bundled into the same session as
the Learn content. Planned build order once picked up: (1) the sidebar
shell + Learn banner as one focused PR first, since that's the riskiest/
most structural part and everything else layers on top of it, (2) the
Watchlist feature, (3) the "Did you know" tip (cheapest — just surfaces
existing Learn content), (4) the sector heatmap, (5) the economic
calendar strip.

---

*Add new phases here as they happen, most recent last — this is meant to
stay current, not be a one-time snapshot.*
