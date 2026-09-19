# CLAUDE.md

## What this repo is

The special `.github` repo for the **Maximising-Shareholder-Value**
GitHub org — not a code repo, just org-wide defaults:

- `profile/README.md` — renders on the org's GitHub page
  (github.com/Maximising-Shareholder-Value).
- `.github/ISSUE_TEMPLATE/` and `.github/PULL_REQUEST_TEMPLATE.md` —
  fall-back templates for any repo in the org that doesn't define its
  own (neither `msv-web` nor `msv-api` do yet, so both currently use
  these).

The actual project repos are [msv-web](https://github.com/Maximising-Shareholder-Value/msv-web)
(frontend) and [msv-api](https://github.com/Maximising-Shareholder-Value/msv-api)
(backend) — created 2026-09 when the original combined
`JozsuaHeng/Maximising-shareholder-value` repo was split and the project
moved under this org. That old repo is now archived; the old combined
Cloudflare Worker deployment was deleted outright once the split was
verified working.

## Maintenance

Keep the "Live" link in `profile/README.md` in sync if the deployed URL
ever changes again. Nothing else here needs regular upkeep.
