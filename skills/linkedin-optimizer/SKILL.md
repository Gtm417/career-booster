---
name: linkedin-optimizer
description: |
  Rewrites LinkedIn profile sections to maximize recruiter search visibility and profile attractiveness. Use when the user runs /optimize-linkedin or says "rewrite my LinkedIn", "improve my LinkedIn profile", "optimize my LinkedIn headline", "rewrite my about section", "optimize my LinkedIn for recruiters", or "help me update my LinkedIn". Uses the stored audit snapshot if available; otherwise audits first.
allowed-tools: [mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__get_page_text]
---

# LinkedIn optimizer

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.



## Purpose

Rewrite each LinkedIn profile section to maximize recruiter discovery (search ranking) and profile conversion (visitors → contact). Deliver sections separately for user review before applying.

## Pre-flight

If an audit snapshot is stored in the profile and is less than 30 days old, use it. Otherwise, run the linkedin-auditor logic first to establish baseline scores.

## Rewrite process

Rewrite each section in order. Deliver each one separately and ask "Does this look right?" before moving to the next.

### 1. Headline

Rules:
- Lead with the target job title or primary keyword
- Add a value proposition or specialization (e.g. "| Residential + Commercial" or "| 8 yrs in sustainable design")
- Include a secondary keyword if space allows
- Under 220 characters
- Do not use just the current job title

Deliver: new headline + explanation of keyword choices made

### 2. About section

Rules:
- First 2 lines (before "see more"): hook the reader — state who you are, what you do, and one compelling signal
- Body: expand on specialization, key achievements, working style, and what you're looking for
- Weave in keywords naturally — do not stuff
- Close with a call to action ("Open to [role type] opportunities in [market]. Feel free to connect or reach out directly.")
- First person throughout
- Under 2,000 characters
- Human-readable — remove all AI-sounding phrasing

Deliver: full About section draft + character count

### 3. Experience section (top 2–3 most recent roles)

For each role:
- Rewrite the description with action verbs and quantified achievements
- Ensure relevant keywords appear naturally
- Lead with the most impactful bullet point
- 3–5 bullets per role

Deliver: each role description separately

### 4. Skills section

Rules:
- Recommend the top 15–20 skills to list based on the user's profile and target role's demand
- Order top 3 pins for maximum impact (most in-demand, most endorsed)

Deliver: ordered skills list with top 3 pinned skills highlighted

### 5. Featured section (if applicable)

Suggest 1–3 items to add to the Featured section:
- Portfolio link, article, case study, or notable project
- Whatever best demonstrates the user's expertise for the target role

## Localization

Apply market-specific norms:
- For markets where English + local language profiles are common (e.g. Poland), flag whether a bilingual profile is recommended
- For single-language markets, keep the profile in one language

## Output format

Deliver each section as a clean, paste-ready block with no markdown formatting (LinkedIn doesn't render markdown). The user should be able to copy and paste directly.

## Storage

After all sections are confirmed, update the LinkedIn audit snapshot in the profile with the optimized versions and date.

## Next step

After all sections are confirmed:

"Your LinkedIn sections are ready to paste in. Once updated, recommended next steps:
- Run `/find-connections` to identify recruiters and hiring managers to connect with
- Run `/find-jobs` to search for roles — your stronger LinkedIn profile will improve visibility in recruiter searches"

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path.
