---
name: market-analyzer
description: |
  Researches and reports on the job market for the user's specialization and target location. Use when the user runs /analyze-market or says "what's the job market like", "analyze the market", "how is demand for my role", "what are companies hiring for", "what's the salary range", "is my field in demand", or "what skills should I add". Uses the user's stored target role and location by default.
allowed-tools: [WebSearch, WebFetch]
---

# Market analyzer

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.



## Purpose

Give the user an accurate, specific picture of the job market they are targeting so they can make informed decisions about skills, positioning, and strategy.

## Process

### Step 1: Confirm scope

If the user's profile has a clear target role and location, proceed with those. If ambiguous, ask:
- Which role/specialization to analyze
- Which location or market (city, country, or remote)

### Step 2: Research

Search for current information on:
- **Demand level** — volume of active job postings for this role in this market; trend (growing, stable, declining)
- **Top employers actively hiring** — companies posting for this role right now
- **In-demand skills and tools** — which skills appear most frequently in postings; which are emerging vs. mature
- **Salary range** — current market rate for this role at the user's seniority level in this location; cite source and note approximate vs. authoritative
- **Work model distribution** — percentage of postings that are remote / hybrid / on-site
- **Key job boards** — which platforms have the most relevant listings for this role and market

### Step 3: Gap analysis

Compare the market's in-demand skills against the user's stored profile:
- Skills the user has that are in high demand — strengths to emphasize
- In-demand skills the user lacks — gaps to address
- Skills the user has that are rare or declining — flag but do not hide

### Step 4: Output

Deliver the full market report:
1. Demand level with trend
2. Top employers actively hiring (with names, not just categories)
3. In-demand skills ranked by frequency
4. Salary range with source caveat
5. Work model distribution
6. Gap analysis vs. user profile
7. Recommended next steps (e.g. "Add X certification", "Target Y companies", "Update profile to emphasize Z")

Cite sources for all data points. Note where figures are approximate or based on limited data.

## Edge cases

- If the target market has limited data (e.g. niche role in small country), acknowledge this and provide the best available picture
- If no target role or location is stored, prompt the user before proceeding

## Next step

After delivering the market report, tell the user:

"Market analysis complete. Recommended next steps:
- Run `/find-jobs` to see live postings in this market now
- If the gap analysis flagged missing skills, consider updating your profile or CV before applying
- Run `/optimize-cv` if the market keywords differ significantly from your current CV"

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path.
