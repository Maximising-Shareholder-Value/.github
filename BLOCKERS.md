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
The FMP key Jozsua provided was tested live and **not stored anywhere**
(no current free-tier use for it, so no reason to hold onto a credential
with nothing to do) — if a paid FMP plan is ever chosen, or another use
for the key comes up, ask for it again.

### ETF/fund top holdings, sector weightings, portfolio composition
**Same blocker as above** — same FMP endpoints, same missing key.

### Full official fund name + issuer, for any arbitrary searched ETF
**Partially blocked.** Finnhub's `profile2` returns empty `{}` for
ETFs (confirmed — this is literally how the app detects "this is an
ETF" today), so there's no live source for "Vanguard Total World Stock
ETF" as a proper name, or "Vanguard"/"iShares"/etc. as an issuer, for
an arbitrary ticker typed into search. **Workaround shipped
2026-09-21:** a small hand-curated lookup table
(`ETF_FUND_INFO` in `script.js`) covering the ~60 tickers already
featured in the home page's browse categories, with real names/issuers
looked up and verified. Anything outside that curated list still shows
generic info only. This is a real, permanent gap for the long tail —
fixing it properly needs the same FMP confirmation as the item above.

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
