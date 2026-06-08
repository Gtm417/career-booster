---
name: setup-daily-connections
description: |
  Provisions a recurring daily scheduled task that automatically discovers new LinkedIn connections and adds them to the review queue. Use when the user runs /setup-daily-connections or says "automate my connection search", "find connections every day", "schedule daily LinkedIn outreach", "set up automatic networking", "run find-connections daily", or "turn on the daily connection job". Creates or updates the schedule; does not send anything.
allowed-tools: [mcp__scheduled-tasks__create_scheduled_task, mcp__scheduled-tasks__list_scheduled_tasks, mcp__scheduled-tasks__update_scheduled_task]
---

# Setup daily connections

## Profile schema

Load the user profile from `career_booster_profile.json` in the workspace `career-booster/` folder (see `references/profile-schema.md`). This skill reads `connectionQueuePath`, `targets.roles`, `targets.locations`, and `targets.industries` to seed the daily search. Never invent fields. Always update `_lastUpdated` on any write.

## Purpose

Create (or update) a recurring scheduled task that runs the `linkedin-outreach` skill once per day in unattended mode, discovers up to `dailyTarget` new LinkedIn contacts, and appends them to `connections-queue.json`. This is the automation trigger for the connection pipeline. It never sends connection requests — discovery and persistence only.

This skill provisions a **runtime** scheduled task; the task itself is not part of the plugin. It is stored separately under the user's Scheduled tasks folder and runs on the user's machine.

## Preconditions

Before provisioning, confirm:

1. **Browser connected** (Claude in Chrome) for live LinkedIn search, OR accept the WebSearch fallback. The **workspace folder must stay connected** so the scheduled run can read/write the queue with native file tools (no filesystem connector needed). A scheduled run can't prompt for connectors, so confirm these now.
2. **Profile populated:** `targets.roles` and `targets.locations` must be non-empty, and `connectionQueuePath` must be set. If empty, instruct the user to run `/setup` first and stop.
3. **Tools must be pre-approved.** A background run cannot answer permission prompts — it stalls on the first unapproved tool. Verified true for this plugin's flow. After creating the task, the user must run it once manually ("Run now") to approve the browser and artifact tools so future daily runs proceed unattended (Step 3.5).
4. **No Drive in the background flow.** The Google Drive connector is unavailable in scheduled runs (requires interactive re-auth — verified). The daily task therefore does discovery + queue write + board refresh only; it never calls Drive. The board's Drive upload stays an interactive, user-pressed action.

## Inputs (optional arguments)

Parse from the command argument or ask if ambiguous:

- **count** — connections to find per day. Default **5**. Clamp to 1–15 (keep volume low to reduce LinkedIn block risk; warn if the user requests more than 10).
- **time** — local time of day to run. Default **08:00**. Convert to a cron expression `M H * * *` in the user's local timezone.

## Process

### Step 1: Write config to the queue file

Load the profile and read `connectionQueuePath` (set by `profile-setup`). If no profile exists, route the user to `/setup` first and stop. Read the queue at that path; if it does not exist, create it from the skeleton in `references/connection-queue-schema.md` (empty `connections`, `config` block). Set:

- `config.dailyTarget` = count
- `config.targetRoles` = profile `targets.roles` (if not already set)
- `config.targetCompanies` = companies of interest if known, else leave as-is

Use read-modify-write; update `_lastUpdated`. This makes the daily target configurable without editing the task prompt.

### Step 2: Check for an existing task

Call `list_scheduled_tasks`. If a task with `taskId` `career-booster-daily-connections` already exists:

- Use `update_scheduled_task` to change its `cronExpression` and/or prompt rather than creating a duplicate.
- Tell the user you updated the existing schedule (show old vs new time/count).

Otherwise proceed to create.

### Step 3: Create the scheduled task

Call `create_scheduled_task` with:

- `taskId`: `career-booster-daily-connections`
- `cronExpression`: derived from **time** (e.g. `0 8 * * *` for 08:00)
- `description`: `Career Booster — daily LinkedIn connection discovery`
- `prompt`: the **self-contained** prompt below (each run starts with no memory of this conversation, so it must name the skill and all context explicitly):

```
Run the Career Booster `linkedin-outreach` skill in UNATTENDED scheduled mode.

1. Load the user profile `career_booster_profile.json` from the connected workspace folder's `career-booster/` subfolder and read `connectionQueuePath`. Read the queue at that path: config.dailyTarget (fallback 5), config.targetRoles, config.targetCompanies. If the profile or queue is missing or corrupt, stop and report — do not reset it.
2. Use the profile for target roles, locations, and industries.
3. Search for up to config.dailyTarget NEW contacts (hiring managers, recruiters, peers, alumni) matching the targets, EXCLUDING anyone whose normalized linkedinUrl is already in the queue (any status). Prefer Claude in Chrome (tag source "chrome"); if the browser isn't connected, fall back to WebSearch `site:linkedin.com/in` (tag source "websearch").
4. For each new contact, draft a personalized connection note under 300 characters and capture name, title, company, profile URL, why-relevant, type, source, and titleVerified. Do not assert a title you didn't read off the profile — set titleVerified false and flag it in why-relevant if unverified.
5. Append each as a new record (status "new") to the queue at connectionQueuePath, using read-modify-write. Generate ids as li-<today>-NNN. Update _lastUpdated.
6. DO NOT send any connection requests or messages. Discovery and persistence only.
6.5. Best-effort, non-fatal: refresh the connection review board (linkedin-outreach Step 5.5). Derive the per-workspace artifact id (connection-review-board-<workspace-slug> from connectionQueuePath), build the HTML from skills/connection-dashboard/references/dashboard.html, and create-or-update that artifact so the new contacts are visible next time the board is opened. Do NOT call Google Drive (unavailable in background). If artifact tools fail, report and continue — never fail the run on the board.
7. Emit a one-line summary: "Added N new connections (M skipped as duplicates). Queue total: T. Board refreshed/skipped: <which>." If notifications are on, this is the run notification.

If Claude in Chrome is unavailable or the queue can't be written, persist whatever was gathered, then report the failure clearly. Never abort silently.
```

- `notifyOnCompletion`: true (so the user sees the daily summary)

### Step 3.5: Pre-approve the tools (one-time, required for unattended runs)

Background runs cannot answer permission prompts, so the first run must be done with the user present to grant approvals (these are then stored on the task and auto-applied to future runs). After creating/updating the task, tell the user:

> "Click **Run now** on the `career-booster-daily-connections` task once and approve the browser and artifact prompts. That pre-approves the tools so the daily run never stalls. Make sure Chrome is open and logged into LinkedIn before you do."

If the user runs it now, confirm the trial run's summary and that approvals were granted.

### Step 4: Confirm and explain

After creation/update, confirm to the user:

- The schedule (e.g. "Every day at 08:00, find up to 5 new connections").
- That it runs **only while the Claude app is open** AND **Chrome is open and logged into LinkedIn**; if either is missing at fire time, discovery can't run. If the app is closed at the scheduled time, it runs on next launch.
- That it must be **run once manually first** to pre-approve tools (Step 3.5), or daily runs will stall on permission prompts.
- That it **never sends** — it only fills the review queue and refreshes the board.
- That **Google Drive is not used** by the daily run (unavailable in the background); Drive upload is a manual action on the board.

## Output

Report exactly: what was created or updated, the cron schedule in plain English, the daily target, and the queue file path the task writes to.

## Next step

After provisioning:

"Daily connection discovery is scheduled. Recommended next step:
- Run `/connection-dashboard` to open the review board where each morning's connections appear for review, copy, and outreach.
- To change the count or time later, run `/setup-daily-connections` again.
- To stop it, ask me to disable the `career-booster-daily-connections` task."

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md`. Specific cases:

- **Scheduled-tasks tool unavailable:** report that scheduling isn't available in this environment and offer to run `/find-connections` manually instead.
- **Queue file unwritable:** still create the task (it can create the file on first run), but warn that config defaults will be used until the file is writable.
- **Profile targets empty:** do not create the task; route the user to `/setup`.

Never abort silently — always report what succeeded, what failed, and the alternative path.
