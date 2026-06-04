---
name: job-finder
description: |
  Searches for relevant job listings across the user's target job boards and returns a ranked list with suitability scores. Use when the user runs /find-jobs or says "find me jobs", "search for jobs", "show me job listings", "what jobs are available", "search LinkedIn for jobs", or "find jobs in [location/field]". Uses stored profile for role, location, and preferred job boards.
allowed-tools: [WebSearch, WebFetch, mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__get_page_text]
---

# Job finder

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.



## Purpose

Search relevant job boards and return a ranked, scored list of matching jobs so the user can prioritize their applications effectively.

## Process

### Step 1: Confirm search parameters

Load from user profile:
- Target role(s) and keywords
- Target location(s) and work model preference
- Preferred job boards per market
- Any filters (salary, company size, industry)

If the user provides overrides in the request (e.g. "only remote", "senior roles only"), apply them.

### Step 2: Search

Search across the user's specified job boards. Where browser access is available, search live. For each board:
- Use the target role as the primary query
- Apply location and work model filters
- Retrieve listings from the past 30 days by default (surface older ones only if results are thin)

## Scoring

Apply the scoring model from `references/scoring-model.md` exactly. Do not redefine the weights or thresholds inline. The scoring model also defines threshold enforcement rules (sub-50 warning and confirmation requirement).

### Step 4### Step 4: Output

Return a ranked list ordered by suitability score (highest first), then by posting date (most recent first):

For each job:
- Job title and company
- Location and work model
- Date posted
- Suitability score (0–100)
- Top 3 matching factors
- Top 3 gaps
- Recommendation: **Apply** (≥75) / **Consider** (50–74) / **Skip** (<50)
- Direct link to the posting

Flag all jobs below 50 with a clear warning. Do not remove them — let the user decide.

### Step 5: After output

Suggest the next step: "/apply [job title] at [company] to start your application package" or "/analyze [job URL] to get a deeper suitability analysis."

## Edge cases

- If a job board cannot be accessed, report the failure and which boards were successfully searched
- If fewer than 5 results are found, widen the search and note that the criteria were broadened
- If the user has no job boards configured, default to LinkedIn and Indeed for the target market

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path.
