---
name: linkedin-outreach
description: |
  Finds relevant LinkedIn contacts and drafts personalized connection requests and follow-up messages. Use when the user runs /find-connections or says "find recruiters on LinkedIn", "who should I connect with", "find hiring managers", "find people at this company", "draft a LinkedIn connection message", "find contacts for this role", or "help me with LinkedIn outreach". Requires a target company, role, or industry.
allowed-tools: [mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__get_page_text, mcp__Claude_in_Chrome__find, mcp__Claude_in_Chrome__form_input]
---

# LinkedIn outreach

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.



## Purpose

Identify the right people to connect with on LinkedIn and write personalized, human-sounding messages for each. Never send without explicit user confirmation.

## Process

### Step 1: Identify targets

Based on the user's target role, industry, and any companies of interest, identify relevant LinkedIn contacts to approach:

Priority order:
1. **Hiring managers** — people with titles like "Head of", "Director of", "VP of", or "Lead" in the relevant department
2. **Recruiters** — internal recruiters or HR at target companies (title: "Recruiter", "Talent Acquisition", "HR Manager")
3. **Peers** — people in the same role at target companies (potential referrers)
4. **Alumni** — people who share a university or previous employer with the user

For each target, note:
- Name and title
- Company
- Mutual connections (if any)
- Why they're relevant

### Step 2: Draft connection requests

For each identified contact, write a personalized connection request:

Rules:
- Under 300 characters (LinkedIn limit for connection note)
- Reference something specific — their role, the company, a mutual connection, or shared background
- State who you are in one clause
- State why you're connecting in one clause
- No "I am looking for a job" in the first message
- No generic "I'd love to connect" without a reason

Example structure: "Hi [Name], I'm an architect with 8 years in residential projects, currently exploring opportunities in Warsaw. Noticed you work in [area] at [Company] — happy to connect and learn more about the team."

### Step 3: Draft follow-up messages

For connections that have accepted but not responded, provide a follow-up message (7–14 days after acceptance):
- Brief, specific, low-friction ask
- One question or request (e.g. "Would you have 15 minutes for a quick call about the team?")
- Reference the original reason for connecting

### Step 4: Output

For each contact:
- Name, title, company
- Why they're a relevant target
- Connection request message (character count noted)
- Optional follow-up message

Present all messages for user review before any action.

## Sending

Never send a connection request or message without explicit user confirmation. For each contact, ask: "Send this connection request to [Name] at [Company]?" and wait for yes/no.

After confirmed sends, log the outreach in the application tracker with status "Outreach sent".

## Next step

After outreach messages are drafted and confirmed:

"Outreach logged. Recommended next steps:
- Check back in 7–10 days — if connections haven't responded, run `/write-email` or `/find-connections` again for a follow-up pass
- Run `/apply` if any of these contacts are associated with a specific role you want to package"

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path.
