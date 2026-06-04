---
name: cv-builder
description: |
  Builds a complete ATS-optimized CV from scratch using the user's stored profile. Use when the user runs /build-cv or says "build me a CV", "create my CV", "write my CV from scratch", "I don't have a CV yet", or "generate a CV for me". Requires a populated user profile.
allowed-tools: [Read, Write]
---

# CV builder

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.



## Purpose

Generate a complete, ATS-optimized, recruiter-ready CV using all information stored in the user's profile. Apply local format conventions for the target market.


## Lifecycle rule

This skill writes to `cvs.base` in the profile schema only. It does not write to `cvs.optimized` or `cvs.tailored`.

**Use this skill only when no CV exists in the profile.** If a base CV already exists, direct the user to `/optimize-cv` instead. If the user explicitly requests a full rebuild, confirm before proceeding.

After writing to `cvs.base`, flag any existing `cvs.optimized` entries as potentially outdated and prompt the user to re-run `/optimize-cv`.

## Pre-flight checks

- Confirm the user profile has sufficient information to build a CV. If key fields are missing (work experience, skills, contact details), identify exactly what's missing and ask the user to provide it before proceeding.
- Confirm the target market and language for this version of the CV.

## References

Apply the ATS checklist from `references/ats-checklist.md` to every CV produced. Apply localization rules from `references/localization-rules.md` for the target market. For Polish-market CVs, add the RODO clause from `references/rodo-clauses.md`. Do not restate these rules inline.

## Structure (adapt per target market)

Build sections in this order, adjusting for local conventions:

1. **Header** — name, contact details (email, phone, location, LinkedIn URL). Remove date of birth and photo for US/UK; flag photo expectation for Germany; include RODO clause for Poland.
2. **Professional summary** — 3–4 lines, role-specific, keyword-rich, written for a recruiter's 10-second scan
3. **Work experience** — reverse chronological; each role with title, company, location, dates, 3–5 bullet points using action verbs + quantified achievements
4. **Education** — reverse chronological; institution, degree, field, dates
5. **Skills** — grouped by category (e.g. Technical, Tools, Soft Skills, Languages)
6. **Certifications** — if present in profile
7. **Additional sections** — publications, volunteer work, awards, etc. if present and relevant
8. **RODO clause** — for Polish market only: "Wyrażam zgodę na przetwarzanie moich danych osobowych dla potrzeb niezbędnych do realizacji procesu rekrutacji..."

## Quality standards

- All bullet points lead with a strong action verb
- At least 60% of experience bullets include a quantified achievement or outcome
- No filler phrases ("responsible for", "helped with")
- Language matches the target market — translate or write in the appropriate language
- ATS-safe formatting: no tables, columns, text boxes, or images
- Professional summary contains the target job title and 2–3 key skills

## Length guidelines

- Entry level (0–3 years): 1 page
- Mid-level (3–10 years): 1–2 pages
- Senior (10+ years): 2 pages maximum (Israel: 1 page regardless)

## Output

1. Deliver the complete CV
2. Note which target market conventions were applied
3. Flag any sections that are thin due to missing profile data
4. Ask if the user wants to save this as their base CV

## Storage

On user confirmation, store as the latest CV version tagged with language, target market, and date.

## Next step

After delivering the built CV, tell the user:

"Your CV is built and saved as your base version. Recommended next steps:
- Run `/optimize-cv` to refine it for ATS compatibility and recruiter impact
- Run `/audit-linkedin` to align your LinkedIn with the new CV
- Or upload it and run `/find-jobs` when you're ready to start applying"
