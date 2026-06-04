---
name: cv-optimizer
description: |
  Improves an existing CV for ATS compatibility and recruiter attractiveness without changing factual content. Use when the user runs /optimize-cv or says "improve my CV", "fix my CV", "optimize my resume", "make my CV better", "ATS check my CV", or "my CV needs work". Requires an existing CV in the profile or uploaded in the conversation.
allowed-tools: [Read, Write]
---

# CV optimizer

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.



## Purpose

Improve the user's CV on two axes:
1. **ATS compatibility** — ensure the CV passes automated screening systems
2. **Recruiter attractiveness** — ensure it performs well in a 10-second human scan

Never change factual content. Only improve structure, language, and presentation.


## Lifecycle rule

This skill reads from `cvs.base` and writes to `cvs.optimized[language]`. It does not write to `cvs.tailored`.

**Use this skill only when a base CV exists in the profile.** If no base CV exists, direct the user to `/build-cv` or to upload their CV first.

After writing to `cvs.optimized`, note which language version was updated and the date.

## References

Apply the ATS checklist from `references/ats-checklist.md`. Apply localization rules from `references/localization-rules.md` for the target market. For Polish CVs, use the RODO clause from `references/rodo-clauses.md`. Do not restate these rules inline.

## ATS compatibility checks

- Section labels match standard ATS-recognized headings (Work Experience, Education, Skills, etc.) in the target language
- No tables, text boxes, columns, headers/footers, or graphics that break parsing
- Contact details in plain text, not image or table
- File is in a parseable format (flag if PDF is image-based)
- Keywords from the target role or market are present in appropriate sections
- Dates are in consistent, parseable formats
- No unexplained gaps that would flag the profile

## Recruiter attractiveness improvements

- Rewrite bullet points to lead with strong action verbs
- Add quantification to achievements where data can be inferred or is present (flag where data is missing so user can fill in)
- Remove filler phrases ("responsible for", "helped with", "assisted in")
- Tighten language — remove redundancy, passive voice, and vague claims
- Ensure the most impactful information appears in the top third of the first page
- Add or improve a professional summary if missing or weak
- Check logical flow and consistency across sections

## Localization

Apply target-market conventions before output:
- **Poland** — add RODO clause if missing; check if Polish language version is needed
- **Israel** — aim for one page; check if Hebrew/English split is needed
- **Germany** — flag if a photo section is expected; check formal tone
- **US/UK** — remove date of birth, photo, marital status if present
- Apply equivalent rules for any other target market in the user's profile

## Output

1. Deliver the full improved CV
2. Provide a summary of every change made with brief reasoning for each
3. Flag any places where user input is needed (e.g. "Please add the actual number of users you managed")
4. Ask if the user wants to save this as the new base CV

## Storage

On user confirmation, store the optimized version as the latest CV in the profile, tagged with language and date.

## Next step

After delivering the optimized CV, tell the user:

"Your optimized CV is saved. Recommended next steps:
- Run `/find-jobs` to search for roles matching your profile
- Run `/audit-linkedin` to check your LinkedIn is as strong as your CV
- Run `/tailor-cv` when you have a specific job to apply for"
