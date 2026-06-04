---
name: cv-tailor
description: |
  Tailors the user's optimized CV for a specific job posting. Use when the user runs /tailor-cv, pastes a job posting and asks to match it, or says "tailor my CV for this job", "customize my CV for this role", "adjust my CV to match this posting", or "adapt my CV for this position". Requires an existing optimized CV and a job posting (URL or text).
allowed-tools: [Read, Write, WebFetch]
---

# CV tailor

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.




## Lifecycle rule

This skill reads from `cvs.optimized[language]` and writes to `cvs.tailored[jobId]`. It never reads directly from `cvs.base`.

If no `cvs.optimized` version exists for the target language, stop and tell the user: "You need an optimized CV in [language] before I can tailor it. Run `/optimize-cv` first." Do not proceed.

Generate a `jobId` using the format defined in `references/profile-schema.md` (YYYYMMDD-{company-slug}-{title-slug}). If a tailored CV for the same jobId already exists, append `-v2`, `-v3`, etc. — never overwrite.

Apply localization rules from `references/localization-rules.md` for the target market.

## Purpose

Produce a job-specific version of the user's CV that maximizes keyword match and relevance for one target posting. Never fabricate experience. Only reframe, reorder, and emphasize existing content.

## Process

### Step 1: Analyze the job posting

Extract from the posting:
- Required skills and tools (must-haves)
- Preferred skills and tools (nice-to-haves)
- Key responsibilities
- Seniority signals
- Industry and company context
- Keywords that will likely be used by ATS screening
- Language of the posting

### Step 2: Gap analysis

Compare the extracted requirements against the user's existing optimized CV:
- Identify strong matches — skills and experience that directly satisfy requirements
- Identify partial matches — experience that relates but isn't explicitly stated
- Identify gaps — requirements the user genuinely cannot claim

Report gaps to the user if they are significant. Do not paper over hard gaps with vague language.

### Step 3: Tailor the CV

- Rewrite the professional summary to mirror the target role's language and priorities
- Reorder skills to lead with the most relevant ones
- Rewrite or reorder bullet points to surface the most relevant experience at the top of each role
- Insert missing keywords from the job posting where they can be applied truthfully
- Adjust job title in summary if a close synonym is more relevant (e.g. "Software Engineer" → "Backend Engineer") — flag this change to the user
- Match language of the CV to the language of the job posting

Do not:
- Add skills or tools the user has not used
- Invent responsibilities or achievements
- Change dates, company names, or titles

### Step 4: Output

1. Deliver the tailored CV
2. List every change made with a one-line reason for each
3. List any requirements from the posting that could not be addressed — honest gap report
4. Provide the direct application link if available

## Storage

Store the tailored version linked to the specific job (by job title + company + date). Do not overwrite the base CV.

## Next step

After delivering the tailored CV, tell the user:

"Your tailored CV for [Job Title] at [Company] is ready. Recommended next step:
- Run `/apply` to generate the full application package (cover letter + email) for this role
- Or run `/tailor-cv` again with another job posting to keep building your pipeline"
