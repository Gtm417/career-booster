---
name: application-tracker
description: |
  Maintains and displays the user's full job application history. Use when the user runs /tracker, says "show my applications", "what have I applied to", "application status", "track my applications", "update application status", or "how many applications have I sent". Also triggers automatically after any completed application to log it.
allowed-tools: [Read, Write]
---

# Application tracker

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.



## Purpose

Maintain a complete, always-accurate log of every job application and outreach action. Log automatically after every completed application. Display on demand.

## Data model

Each application entry contains:
- **Job title**
- **Company name**
- **Date applied / contacted**
- **Status** — one of: Outreach sent / Applied / Application viewed / Interview scheduled / Interview completed / Offer received / Rejected / Withdrawn
- **Language** — language used in the application
- **Documents sent** — CV version, cover letter (yes/no), outreach email (yes/no)
- **Source** — which job board or channel
- **Direct link** — URL to the original posting
- **Notes** — any free-form notes (e.g. recruiter name, interview date, feedback received)

## Auto-logging

After every application-packager or linkedin-outreach action completes and the user confirms, log the entry automatically. The user should never need to manually log a standard application.

## Display format (/tracker command)

When the user runs /tracker or asks to see their applications, output a clear summary:

**Application tracker — [total count] applications**

Group by status:

```
Active applications ([count]):
• [Job Title] @ [Company] | Applied [date] | [language] | [link]
  Status: [current status]

Interviews ([count]):
• ...

Offers ([count]):
• ...

Closed ([count]):
• [Job Title] @ [Company] | Applied [date] | Status: Rejected / Withdrawn
```

Show oldest-first within each group. Include a summary line at the top: "X applied, Y in progress, Z interviews, W offers."

## Status updates

When the user provides a status update (e.g. "I got an interview at Deloitte" or "I was rejected by Company X"), update the relevant entry immediately and confirm the change.

## Manual log

If the user applied somewhere outside of Career Booster, they can say "log an application" and provide the details. Collect: job title, company, date, status, link. Store it.

## Statistics (on request)

If the user asks for stats, provide:
- Total applications sent
- Response rate (applications with at least one response / total applied)
- Interview rate
- Average days from application to response
- Most active job boards used
- Most common rejection stage

## Edge cases

- If the same job appears twice (e.g. applied again after rejection), create a new entry and note it as a reapplication
- If a link is dead or unavailable, note "link unavailable" rather than leaving it blank

## Next step

After displaying the tracker:

- If the user has applications with no response after 10+ days: "Some applications haven't had a response in a while. Run `/write-email` to draft a follow-up for any of them."
- If the user has no applications yet: "No applications logged yet. Run `/find-jobs` to start your search."
- If the user has interviews scheduled: "You have interviews coming up. Ask me to help you prepare — company research, likely questions, or a summary of the role."
