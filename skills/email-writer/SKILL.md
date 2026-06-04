---
name: email-writer
description: |
  Writes a personalized outreach email to an employer or recruiter. Use when the user runs /write-email or says "write an email to a recruiter", "draft an outreach email", "email this company", "write a cold email for this job", "contact the hiring manager", or "draft a follow-up email". Requires a target contact or company and a specific job or context.
allowed-tools: [mcp__355a8161-8c22-49e2-b2b3-4a7c21344898__create_draft]
---

# Email writer

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.




## Scope

This skill writes **standalone outreach emails**: cold outreach, recruiter pings, post-interview follow-ups, and networking messages. It is not triggered as part of `/apply`.

**Do not use for the outreach email inside an application package.** That email is produced by `application-packager` as Document 3. The distinction:

| `email-writer` | `application-packager` Document 3 |
|---|---|
| Standalone outreach | Accompanies a specific application package |
| No CV attached | CV and cover letter already exist |
| Goal: start a conversation | Goal: introduce the full application |

## Purpose

Write a short, specific, professional outreach email that gets a response. Not a template. Not generic. Every email is written for one recipient and one context.

## Inputs to gather

If not provided in the request:
- Who is the email going to? (recruiter, hiring manager, contact, cold outreach)
- Is this related to a specific job posting? If yes, which one?
- What is the goal? (Express interest, follow up, request a conversation, ask about opportunities)
- Does the user know the recipient's name?

## Email structure

**Subject line:**
- Specific and relevant — never "Job Application" or "Reaching Out"
- Options: reference the role ("Re: Senior Architect role – [Company]"), a shared connection, or a relevant hook
- Maximum 8 words

**Body (3–4 sentences):**

1. Who you are — one sentence, specific to this contact (e.g. "I'm an architect with 8 years in residential projects, currently targeting roles in Warsaw.")
2. Why you're reaching out — specific to this company or role (not "I admire your company")
3. Why you're a fit — one achievement or specific skill match
4. Call to action — clear and low-friction ("Happy to share my portfolio if useful." or "Would a 15-minute call work for you this week?")

**Tone:**
- Professional but human — not stiff
- Confident but not presumptuous
- Match to signals from the company/posting (startup = casual-professional; corporate = formal)
- Match the language of the target market / recipient

## Quality checks

Before outputting:
- Does it sound like a person wrote it? Remove any AI-signaling phrases ("I hope this message finds you well", "I am writing to express my interest")
- Is the subject line specific enough to get opened?
- Is the call to action clear and low-friction?
- Is it short enough to read in 30 seconds?

## Output

Deliver:
1. Subject line
2. Email body
3. A note on any personalization gaps ("I couldn't find the hiring manager's name — replace [Name] if you know it")

Ask: "Shall I send this, or would you like to adjust it first?"

## Sending

Never send without explicit user confirmation. On confirmation, show the final recipient and subject line and ask for one final "yes" before sending.

## Next step

After the user confirms the email (sent or saved):

- If this was a cold outreach or recruiter ping: "Email ready. I've noted this outreach. If you don't hear back in 7–10 days, run `/write-email` again for a follow-up."
- If this was part of a job application: "Email ready. Run `/tracker` to log this application, or `/apply` to generate the full package if you haven't yet."

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path.
