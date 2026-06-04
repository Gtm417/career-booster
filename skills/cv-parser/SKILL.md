---
name: cv-parser
description: |
  Parses and extracts structured information from an uploaded CV or pasted CV content. Use when the user uploads a CV file, pastes their CV text, or says "parse my CV", "read my CV", "extract my CV", "I have a CV to upload", or "here is my resume". Maps all extracted data to the user profile.
allowed-tools: [Read, Write]
---

# CV parser

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.

## Trigger conditions

- User uploads a file (PDF, DOCX, or similar)
- User pastes raw CV text into the chat
- User says they have a CV to share

**Do not fire if `profile-setup` is currently running.** When a user uploads a CV during `/setup`, `profile-setup` invokes this skill internally and controls the merge flow. Firing independently at the same time would create a conflict.

## Extraction process

Read the full CV content. Extract and map the following to the user profile:

- **Personal details** — name, email, phone, location, LinkedIn URL
- **Target role** — infer from job title history, summary, or objective statement if present
- **Work experience** — each role: job title, company, location, dates, responsibilities, achievements
- **Education** — institution, degree, field, dates, GPA if present
- **Skills** — technical skills, tools, technologies, soft skills
- **Languages** — each language with proficiency level stated
- **Certifications and courses** — name, issuer, date
- **Other sections** — publications, volunteer work, awards, references, RODO clause, etc.

## After extraction

1. Display a clear summary of everything extracted, organized by section
2. Highlight any gaps — sections missing or fields that could not be determined
3. Show any conflicts with existing stored profile data and ask which version to keep
4. Ask the user to confirm before merging into the profile
5. Once confirmed, update the profile and confirm: "Profile updated with your CV data."

## Storage

Store the parsed CV as `cvs.base` in the profile schema (see `references/profile-schema.md`), tagged with the detected language and current date. Set `source: "parsed"`. Merge all extracted fields using the conflict resolution rules defined in the schema. Apply the CV lifecycle rule: this skill writes to `cvs.base` only — never to `cvs.optimized` or `cvs.tailored`. Apply connector failure protocols from `references/connector-failure-protocols.md` if file reading fails.

## Edge cases

- If the file cannot be parsed, ask the user to paste the text directly
- If the CV is in a non-English language, detect the language, extract in that language, and flag it for localization review
- If multiple CVs are uploaded, parse each one and ask which to use as the base version

## Next step

After confirming the profile update, tell the user: "Your CV has been parsed and saved as your base version."

Then suggest the most logical next step:
- If no optimized CV exists: "Run `/optimize-cv` to improve it for ATS compatibility and recruiter attractiveness."
- If an optimized CV already exists: "Your profile already has an optimized CV. Run `/tailor-cv` to adapt it for a specific role, or `/find-jobs` to search for matches."
