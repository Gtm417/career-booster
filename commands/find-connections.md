---
description: Find relevant LinkedIn contacts and draft personalized connection requests.
argument-hint: "[count] [company, role, or industry]  e.g. 2 architects Warsaw"
---

The user has run /find-connections. Execute the `linkedin-outreach` skill.

Argument parsing:
- If the argument starts with a number, that is the COUNT of connections to find this run — it overrides `config.dailyTarget` for this invocation (clamp 1–15). The rest of the argument is the target context (company, role, or industry).
- If no number is given, use `config.dailyTarget`.
- If no target context is given, fall back to the profile's target role/location/companies; if the profile has none, ask for context.

Prefer Claude in Chrome for live LinkedIn search; fall back to WebSearch (`site:linkedin.com/in`) if the browser isn't connected. Verify titles per the skill before persisting. Never send connection requests — discovery and persistence only.
