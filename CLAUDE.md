# CLAUDE.md

## What this repo is

Two things at once, both legitimate:

1. **The special `.github` repo** for the **Maximising-Shareholder-Value**
   GitHub org — its name is fixed by GitHub's own convention, not
   negotiable. Provides org-wide defaults:
   - `profile/README.md` — renders on the org's GitHub page
     (github.com/Maximising-Shareholder-Value).
   - `.github/ISSUE_TEMPLATE/` and `.github/PULL_REQUEST_TEMPLATE.md` —
     fall-back templates for any repo in the org that doesn't define its
     own (neither `msv-web` nor `msv-api` do yet, so both currently use
     these).
2. **The project's governance/PMO hub** (added 2026-09-21) — `README.md`
   (root), `ROADMAP.md`, `TODO.md`, `HISTORY.md`, `BLOCKERS.md`, and
   `API_RESEARCH.md` at this repo's root. This is where project-wide
   planning lives — vision, task list, chronological history, known
   blockers, data-source research — since it spans both code repos and
   shouldn't be siloed inside just one of them.

The actual project repos are [msv-web](https://github.com/Maximising-Shareholder-Value/msv-web)
(frontend) and [msv-api](https://github.com/Maximising-Shareholder-Value/msv-api)
(backend) — created 2026-09 when the original combined
`JozsuaHeng/Maximising-shareholder-value` repo was split and the project
moved under this org. That old repo is now archived; the old combined
Cloudflare Worker deployment was deleted outright once the split was
verified working.

## History: governance docs moved here (2026-09-21)

The five governance docs above originally lived in `msv-web/.github/` —
a folder that happened to share the `.github` name with this repo, which
caused real confusion (Jozsua expected *this* repo, the org's actual
`.github` repo, to be the project's planning hub, and was instead
finding roadmap/history content buried inside one of the two code repos
that this repo is supposed to sit above). Moved here to fix that mismatch
— this repo is now genuinely the single "project structure, governance,
roadmap" home Jozsua originally wanted. `msv-web/.github/` now holds only
its CI workflow (`workflows/ci.yml`), which is correctly repo-specific
and stays there.

All five files' cross-references to each other were relative filenames
(`[TODO.md](TODO.md)` etc.), not `../` paths — so the move didn't break
any of their mutual links, only the handful of links that pointed out at
the two code repos (fixed to full GitHub URLs instead of `this repo`).

## Maintenance

- Keep the "Live" link in `profile/README.md` in sync if the deployed
  URL ever changes again.
- **Keep the governance docs current going forward** — same discipline
  as before the move, just in this repo now: any real change to $MSV
  (feature shipped, decision made, blocker found) should update the
  relevant file here (`HISTORY.md`/`TODO.md`/`BLOCKERS.md`) in the same
  work session, not as an afterthought. `msv-web/changelog.js` is a
  separate, user-facing changelog — update that too for anything
  user-visible, but it isn't a substitute for these files.
- If msv-web or msv-api ever need to link back here, use the full
  `https://github.com/Maximising-Shareholder-Value/.github` URL (or a
  specific file within it) — there's no clean relative path between
  sibling repos.
