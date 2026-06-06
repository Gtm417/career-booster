---
name: setup-daily-connections
description: |
  Provisions a recurring daily scheduled task that automatically discovers new LinkedIn connections and adds them to the review queue. Use when the user runs /setup-daily-connections or says "automate my connection search", "find connections every day", "schedule daily LinkedIn outreach", "set up automatic networking", "run find-connections daily", or "turn on the daily connection job". Creates or updates the schedule; does not send anything.
allowed-tools: [mcp__scheduled-tasks__create_scheduled_task, mcp__scheduled-tasks__list_scheduled_tasks, mcp__scheduled-tasks__update_scheduled_task]
---

# Setup daily connections

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for field definitions. This skill reads `targets.roles`, `targets.locations`, and `targets.industries` to seed the daily search. Never invent fields. Always update `_lastUpdated` on any write.

## Purpose

Create (or update) a recurring scheduled task that runs the `linkedin-outreach` skill once per day in unattended mode, discovers up to `dailyTarget` new LinkedIn contacts, and appends them to `connections-queue.json`. This is the automation trigger for the connection pipeline. It never sends connection requests — discovery and persistence only.

This skill provisions a **runtime** scheduled task; the task itself is not part of the plugin. It is stored separately under the user's Scheduled tasks folder and runs on the user's machine.

## Preconditions

Before provisioning, confirm:

1. **Connectors connected:** Claude in Chrome (Browser) and Local file system. If either is missing, tell the user which to connect and stop — a scheduled run cannot prompt for connectors.
2. **Profile populated:** `targets.roles` and `targets.locations` must be non-empty. If empty, instruct the user to run `/setup` first and stop.

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

1. Load the user profile `career_booster_profile.json` from <USER_HOME>/.career-booster/ and read `connectionQueuePath`. Read the queue at that path: config.dailyTarget (fallback 5), config.targetRoles, config.targetCompanies. If the profile or queue is missing or corrupt, stop and report — do not reset it.
2. Use the profile for target roles, locations, and industries.
3. Using Claude in Chrome, search LinkedIn for up to config.dailyTarget NEW contacts (hiring managers, recruiters, peers, alumni) matching the targets, EXCLUDING anyone whose normalized linkedinUrl is already in the queue (any status).
4. For each new contact, draft a personalized connection note under 300 characters and capture name, title, company, profile URL, why-relevant, and type.
5. Append each as a new record (status "new") to the queue at connectionQueuePath, using read-modify-write. Generate ids as li-<today>-NNN. Update _lastUpdated.
6. DO NOT send any connection requests or messages. Discovery and persistence only.
7. Emit a one-line summary: "Added N new connections (M skipped as duplicates). Queue total: T." If notifications are on, this is the run notification.

If Claude in Chrome is unavailable or the queue can't be written, persist whatever was gathered, then report the failure clearly. Never abort silently.
```

- `notifyOnCompletion`: true (so the user sees the daily summary)

### Step 4: Confirm and explain

After creation/update, confirm to the user:

- The schedule (e.g. "Every day at 08:00, find up to 5 new connections").
- That it runs **only while the Claude app is open**; if closed at the scheduled time, it runs on next launch.
- That it **never sends** — it only fills the review queue.
- That the Browser connector must stay authenticated to LinkedIn for it to work.

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
