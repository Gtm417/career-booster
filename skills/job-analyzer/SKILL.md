---
name: job-analyzer
description: |
  Analyzes a specific job posting against the user's profile and returns a detailed suitability score with recommendation. Use when the user provides a job URL or posting text and says "analyze this job", "score this role", "is this a good fit for me", "what are my chances", "should I apply for this", or pastes a job description directly. Also runs automatically before application packaging.
allowed-tools: [WebFetch, mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__get_page_text]
---

# Job analyzer

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.



## Purpose

Give the user a clear, honest assessment of fit for a specific job — including what they're strong on, where they fall short, and whether to apply.


## Scoring

Apply the scoring model from `references/scoring-model.md` exactly. Do not redefine the weights or thresholds inline. The scoring model also defines threshold enforcement rules (sub-50 warning and confirmation requirement).

## Process

### Step 1: Get the job posting

If the user provided a URL, fetch and read the full posting. If they pasted text, use that directly. If neither, ask for one.

### Step 2: Extract requirements

Parse the posting into:
- **Must-haves** — explicitly required qualifications, skills, experience, language proficiency
- **Nice-to-haves** — preferred but not required
- **Red flags** — requirements the user cannot meet at all
- **Hidden signals** — seniority markers, culture signals, urgency indicators

Distinguish must-haves from nice-to-haves carefully — many postings conflate them.

### Step 3: Score suitability (0–100)

Use the same scoring model as job-finder:
- Skills match: 40%
- Experience match: 25%
- Location/work model match: 15%
- Industry match: 10%
- Role title match: 10%

### Step 4: Output

Deliver the analysis:

**Score:** [0–100]
**Recommendation:** Apply / Consider / Skip

**Top matching factors (3):**
- [Specific match with reasoning]

**Key gaps (3):**
- [Specific gap with impact level: critical / moderate / minor]

**Red flags:**
- [Any requirements that are hard blockers]

**Hidden signals:**
- [Seniority mismatch, culture clues, urgency, etc. if detected]

**Language of posting:** [detected language]

**Assessment:** 2–3 sentences of honest overall judgment.

### Step 5: Threshold enforcement

If the score is below 50:
- Explicitly state: "This role scores below 50. Applying is likely to result in rejection. Here's why: [top reason]."
- Ask whether the user wants to proceed anyway
- Do not proceed to application packaging without explicit user confirmation

### Step 6: Suggest next step

If score ≥ 50: "Run /apply to generate your tailored application package for this role."
If score < 50 and user wants to proceed: warn once more, then proceed if confirmed.

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path.
