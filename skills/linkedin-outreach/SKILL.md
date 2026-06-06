---
name: linkedin-outreach
description: |
  Finds relevant LinkedIn contacts and drafts personalized connection requests and follow-up messages. Use when the user runs /find-connections or says "find recruiters on LinkedIn", "who should I connect with", "find hiring managers", "find people at this company", "draft a LinkedIn connection message", "find contacts for this role", or "help me with LinkedIn outreach". Requires a target company, role, or industry.
allowed-tools: [mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__get_page_text, mcp__Claude_in_Chrome__find, mcp__Claude_in_Chrome__form_input]
---

# LinkedIn outreach

> **File access:** This skill reads and writes the connection queue using your built-in file access. The queue path is **not** hardcoded — read it from the profile's `connectionQueuePath` field (set by `profile-setup` to `<USER_HOME>/.career-booster/connections-queue.json`). If no profile exists, route the user to `/setup` first.

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.

The connection queue (`connections-queue.json`) is a separate operational store defined in `references/connection-queue-schema.md`. It is the staging surface for discovered connections and the review dashboard; the profile's `outreach[]` array remains the canonical record for sent connections.



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

### Step 1.5: Load queue config and dedup set

Read the connection queue at the profile's `connectionQueuePath` (see `references/connection-queue-schema.md` for shape). Load the profile first to get that path.

- If it does not exist, create it from the skeleton with `config.dailyTarget` = 5.
- Read `config.dailyTarget` — this is the number of NEW contacts to surface this run. Read `config.targetRoles` / `config.targetCompanies` if set; otherwise fall back to the profile's `targets.roles` and industries/companies of interest.
- Load the existing `connections` array and build a set of **normalized** `linkedinUrl` values already present (lowercase, strip trailing slash, query, and fragment). This is the dedup set: never surface anyone already in the queue, in ANY status.

Limit the targets carried into Step 2 to at most `config.dailyTarget` new (non-duplicate) contacts.

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

### Step 5: Persist to the connection queue

For each contact identified and drafted this run that is NOT in the dedup set:

1. Build a connection record per `references/connection-queue-schema.md`. Generate `id` as `li-<dateFound>-NNN` (sequence continuing from the highest existing index for today). Set `status: "new"`, `dateFound` = today, `dateSent: null`, and `messageChars` = the exact character count of `message`.
2. Map `type` to the profile outreach enum (`hiring_manager | recruiter | peer | alumni | other`) based on the priority tier the contact came from in Step 1.
3. Append all new records to `connections`. Never overwrite existing records.
4. Update `_lastUpdated`. Use read-modify-write: read the full file, append, write back the whole object to `connectionQueuePath` using your file access.
5. Report: "Added N new connections to the queue (M skipped as duplicates). Queue total: T."

Persistence is non-destructive — it never sends. Do not gate this step on confirmation.

### Execution context: interactive vs scheduled

This skill runs in two contexts:

- **Interactive** (user typed `/find-connections`): present the drafted contacts for review (Step 4), then persist to the queue (Step 5).
- **Scheduled** (daily task via `setup-daily-connections`): no user is present. Skip the Step 4 review presentation, persist directly with status `new`, emit the one-line summary, and never attempt to send.

## Sending

Never send a connection request or message without explicit user confirmation. For each contact, ask: "Send this connection request to [Name] at [Company]?" and wait for yes/no.

After confirmed sends, log the outreach in the application tracker with status "Outreach sent".

## Next step

After outreach messages are drafted and confirmed:

"Outreach logged. Recommended next steps:
- Check back in 7–10 days — if connections haven't responded, run `/write-email` or `/find-connections` again for a follow-up pass
- Run `/apply` if any of these contacts are associated with a specific role you want to package"

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path. Specific cases for the queue persistence step:

- **Queue file unreadable / corrupt JSON:** do not overwrite. Report the corruption, write findings to a sidecar `connections-queue.recovered.json`, and stop. Never silently reset the queue.
- **No profile / `connectionQueuePath` missing:** the user hasn't run `/setup`. Route them to `/setup` (it creates the folder and queue), then retry. Do not invent a path.
- **Claude in Chrome / LinkedIn failure mid-run:** persist whatever was successfully gathered before the failure; report partial success (X found, Y persisted) and what failed.
- **Dedup ambiguity (same person, different URL):** treat distinct normalized URLs as distinct people; note possible duplicates in the summary rather than auto-merging.
