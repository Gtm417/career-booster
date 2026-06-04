---
name: application-packager
description: |
  Generates a complete application package for a specific job: tailored CV, cover letter, and outreach email. Use when the user runs /apply, says "apply to this job", "prepare my application", "create an application package", "help me apply for this role", or references a specific job they want to apply to. Requires a job posting and an existing CV in the profile.
allowed-tools: [Read, Write, WebFetch, mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__get_page_text, mcp__355a8161-8c22-49e2-b2b3-4a7c21344898__create_draft]
---

# Application packager

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.




## Scope

This skill produces the email that **accompanies a complete application package** (Document 3). It is not a general email writer.

For standalone outreach emails — cold contact, recruiter messages, follow-ups not tied to an active application package — use `email-writer` instead.

## Purpose

Produce a complete, job-specific application package — three documents, all language-matched to the posting, all specific to this company and role.

## Pre-flight checks

1. Load the user's optimized CV from the profile. If none exists, run cv-optimizer first.
2. Confirm the job posting is available (URL or text).
3. Run job-analyzer internally to get the suitability score. If the score is below 50, stop and warn the user before continuing.

## Document 1: Tailored CV

Run the cv-tailor logic for this specific posting:
- Extract keywords and requirements from the posting
- Rewrite the professional summary for this role
- Reorder and rewrite bullet points to surface the most relevant experience
- Insert relevant keywords where they apply truthfully
- Match language to the posting language
- Apply target-market localization

## Document 2: Cover letter

Write a cover letter in three paragraphs maximum:

**Paragraph 1 — Why this role:**
- Reference the specific company, not a generic employer
- Name one specific thing about the company, team, or role that makes it relevant to the user
- Connect it to the user's experience or direction

**Paragraph 2 — Why this candidate:**
- Pick the 2–3 strongest matches between the user's profile and the job requirements
- Lead with an achievement, not a duty
- Use specific numbers or outcomes where possible

**Paragraph 3 — Call to action:**
- Express clear interest
- Invite next step
- One sentence maximum

Match the language of the job posting. Keep total length to one page. No generic phrases ("I am writing to express my interest..."). No AI-sounding language.

## Document 3: Outreach email

Write an email to accompany or precede the application:
- Subject line: specific, not generic ("Application: Senior Architect – [Company] – [Date]" or a hook if appropriate)
- Body: 3–4 sentences. Who the user is, what they're applying for, one specific reason they fit, call to action
- Professional tone matched to the company culture signals in the posting
- Match the language of the posting

## Output

Present all three documents in sequence:
1. Tailored CV
2. Cover letter
3. Outreach email

After each document, note the key tailoring decisions made.

Provide the direct application link.

Ask the user: "Would you like to apply manually via the link above, or should I submit the application for you?"

## After confirmation

Once the user confirms they're applying:
- Log the application to the tracker: job title, company, date, status "Applied", language used, link
- Confirm: "Application logged. Good luck!"

## Sending

Never send an email or submit an application without explicit user confirmation. If the user says "send it", confirm once more with the recipient and content before acting.

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path.
