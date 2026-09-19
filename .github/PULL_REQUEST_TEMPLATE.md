## What does this change?


## Why?


## Checklist

- [ ] Tested against real, live data (not just assumed it works) — several
      past bugs in this project only showed up against real multi-day
      data, not a mocked single day
- [ ] If this touches a new API endpoint/field, confirmed it's actually
      available on the free tier being used (see the repo's `CLAUDE.md`
      for known free-tier gaps)
- [ ] If this only applies to stocks (not ETFs/crypto), it degrades
      sensibly for the other two rather than showing broken/blank content
- [ ] No API keys or secrets included anywhere in the diff
