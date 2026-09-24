# Maximising-Shareholder-Value — Project Hub

This repo is the **project management home base** for $MSV — everything
about *why* the project exists, *where it's headed*, and *what's already
happened*, in one place, separate from either code repo. If you're
looking for the actual app, see the links below; if you're trying to
understand the project itself, you're in the right place.

*(This repo is also GitHub's special `.github` repo for the org — its
name is fixed by that GitHub convention, not a project-naming choice. It
serves two purposes at once: org-wide defaults — the profile page below
and fallback issue/PR templates — and, as of 2026-09-21, this governance
hub. See `CLAUDE.md` for the mechanical detail.)*

## What $MSV is

A free stock/ETF/crypto/commodity research dashboard — search a ticker,
get valuation, financial health, and a plain-English read on where it
stands, plus a live view of markets worldwide and the economic backdrop
driving them. ("Maximising Shareholder Value" is a joke name; the
numbers are genuinely real.)

## The repos

| Repo | What it is |
|---|---|
| **[msv-web](https://github.com/Maximising-Shareholder-Value/msv-web)** | The frontend — plain HTML/CSS/JS, no build step. |
| **[msv-api](https://github.com/Maximising-Shareholder-Value/msv-api)** | The backend — a Cloudflare Worker proxying Finnhub/Twelve Data/FRED/CoinGecko/Alpaca/World Bank so real API keys never reach the browser. |
| **Live site** | [msv-web.jozsua-heng.workers.dev](https://msv-web.jozsua-heng.workers.dev/) |
| **Internal roadmap view** | [msv-web.jozsua-heng.workers.dev/roadmap.html](https://msv-web.jozsua-heng.workers.dev/roadmap.html) — a visual status board of the six long-term pillars, not linked from the site's own nav. |

## Governance docs (this repo)

| Question | File |
|---|---|
| **"Where is this project going, long-term?"** | [ROADMAP.md](ROADMAP.md) — vision, architecture diagram, the six pillars broken down, build order and why. |
| **"What should I actually work on next?"** | [TODO.md](TODO.md) — the concrete, checkable task list. |
| **"What's already happened?"** | [HISTORY.md](HISTORY.md) — a dated, chronological story of the project from its first commit onward. |
| **"What's blocked right now, and why?"** | [BLOCKERS.md](BLOCKERS.md) — things that genuinely can't be built yet (no free data source, a missing permission), so it never has to be re-explained. |
| **"What free data sources exist for feature X?"** | [API_RESEARCH.md](API_RESEARCH.md) — researched APIs, free tiers, rate limits, confirmed live rather than guessed. |
| **"How do we get more free data/detail into the app?"** | [DATA_EXPANSION_RECOMMENDATIONS.md](DATA_EXPANSION_RECOMMENDATIONS.md) — concrete, sourced ideas for using more of what's already free, not a paid-upgrade wishlist. |

Each repo's own `CLAUDE.md`/`README.md` stays the *technical* reference
(how the code actually works, data-source quirks, config/secrets setup)
— this repo is the *planning* reference. Keep both current: a real
change touches the code repo's docs *and*, if it affects the roadmap or
history, this repo's.

## Team

Currently a solo project — [Jozsua](https://github.com/JozsuaHeng)
(product + direction) working with Claude (build + research). Structured
with this hub, `CONTRIBUTING.md` in each code repo, and issue/PR
templates below specifically so a second contributor could ramp up here
without needing anything explained in person.

## Org-wide GitHub defaults (why this repo is named `.github`)

- [`profile/README.md`](profile/README.md) — renders on the org's public
  GitHub page.
- [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE/) and
  [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md) —
  fallback templates for any repo in the org that doesn't define its own.
