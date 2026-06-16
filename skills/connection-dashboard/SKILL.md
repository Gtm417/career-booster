---
name: connection-dashboard
description: |
  Opens the persistent Connection Review Board — a dashboard of all discovered LinkedIn connections, where the user reviews, copies notes, opens profiles, and marks status. Use when the user runs /connection-dashboard or says "open my connections board", "show my connection queue", "review my connections", "open the connection dashboard", "where are my daily connections", or "review board". Reads the queue and applies status changes the user pastes back.
allowed-tools: [mcp__cowork__create_artifact, mcp__cowork__list_artifacts, mcp__cowork__update_artifact]
---

# Connection dashboard

## Purpose

Create (or refresh) a persistent Cowork artifact that renders the connection queue as a review board. Each card shows a contact's name, title, company, why-relevant, the drafted connection note (with copy button and character count), an "Open profile" link, and status buttons (Reviewed / Sent / Skip). The board filters by status and company, sorts, and shows live counts.

The dashboard HTML is packaged with this skill at `references/dashboard.html`. This skill ships in the plugin; the **artifact instance** it creates is runtime and lives in the user's Cowork sidebar.

## File access architecture (read first — this is non-obvious)

A Cowork artifact runs in a sandbox with hard limits, established empirically:

1. `window.cowork.callMcpTool` works **only for hosted/cloud connectors**. Local connectors (Desktop Commander, any local filesystem MCP) return **400** — the sandbox refuses them. So the board **cannot read or write a local file**.
2. The artifact API is exactly `callMcpTool`, `askClaude` (inference only), and `runScheduledTask` (no args). There is **no** function to send a message to chat or otherwise write state back.

Therefore the board uses an **embed + paste** model:

- **READ** — this skill reads `connections-queue.json` agent-side with built-in file access (it's inside the connected workspace folder) and **bakes the `connections` array into the HTML at build time**. The board renders from embedded data; it makes no file calls.
- **WRITE** — the board cannot persist. Status changes are staged in `localStorage` and exported as a one-line command (`Apply status changes to my connection queue: <id>=<status>, …`) that the user copies and **pastes into chat**. This skill (see "Applying pasted status changes") then writes them to the queue.

Do NOT reintroduce `callMcpTool` file reads/writes against a local connector — it returns 400. This was tested and confirmed.

## Preconditions

1. **Profile + queue exist.** The queue path is the profile's `connectionQueuePath` (set by `profile-setup`, e.g. `<WORKSPACE>/career-booster/connections-queue.json`). No profile → route to `/setup`. Missing queue file → `/find-connections` or `/setup-daily-connections` creates it.

## Step 1: Read the queue agent-side

Load the profile, read `connectionQueuePath`, and read the queue file with your built-in file access (it's in the connected workspace folder). Parse the JSON; keep the `connections` array and the absolute path.

## Step 2: Build the artifact HTML

1. Read `references/dashboard.html` from this skill.
2. Substitute `__QUEUE_PATH__` with `connectionQueuePath`. **Escape backslashes** for the JS string literal (e.g. `C:\\Users\\me\\Documents\\…\\career-booster\\connections-queue.json`).
3. Substitute `__CONNECTIONS_JSON__` with `JSON.stringify(connections)` (the array only). If empty, use `[]`.
4. Drive export button: the template now ships with this user's Drive `create_file` tool and
   target folder id baked into `QUEUE_DATA` defaults (tool
   `mcp__205b77f2-291a-4d3d-b633-ed56405ca47b__create_file`, folder
   `1XKFAeqckIj7dBkLu4npDa4ACk0wZfUg4`). No `__DRIVE_TOOL__` substitution is required, and the
   button persists across foreground and background rebuilds. Uploads create a new
   timestamp-suffixed Google Sheet in that folder (the connector has no delete/overwrite tool).
5. Write the substituted document to a scratch file.

## Step 3: Create or update the artifact

- **Derive the per-workspace artifact id** per `references/connection-queue-schema.md` → "Dashboard artifact id": `connection-review-board-<workspace-slug>`, computed from `connectionQueuePath`. Never hardcode the id.
- Call `list_artifacts`. If an artifact with that derived id exists, use `update_artifact`; otherwise `create_artifact`.
- `id`: the derived `connection-review-board-<workspace-slug>`
- `html_path`: the scratch file from Step 2
- `description`: `Review board for discovered LinkedIn connections (embedded snapshot of connections-queue.json).`
- `mcp_tools`: `["mcp__205b77f2-291a-4d3d-b633-ed56405ca47b__create_file"]` (the Drive tool the upload button calls). The board calls no other MCP tools.

## Applying pasted status changes

When the user pastes a command like `Apply status changes to my connection queue: li-2026-06-06-001=sent, li-2026-06-06-003=skipped`:

1. Read the queue at `connectionQueuePath`.
2. For each `id=status` pair, set that record's `status`; if `sent`, also set `dateSent` to today. Validate ids exist and statuses are one of `new | reviewed | sent | skipped`.
3. Update `_lastUpdated` and write back (read-modify-write; never overwrite blindly).
4. Rebuild the artifact (Step 2–3) so it shows the saved state with no pending changes.
5. For any record moved to `sent`, map it into the profile's `outreach[]` per `references/connection-queue-schema.md`.

## Output

Confirm: the board is open in the sidebar, how many connections it shows (by status), and the queue path it reflects. Remind the user that status changes are saved by pasting the board's "Copy update for Claude" command into chat.

## Notes on behavior

- The board does **not** send anything. "Open profile" opens LinkedIn; the user pastes the copied note and sends manually (Phase 4 will add assisted send).
- The board is a snapshot. After discovery runs or after you apply pasted changes, re-run `/connection-dashboard` to refresh the embedded data.
- The artifact has a built-in Reload button — the HTML must not add its own.
- View preferences and pending (unsaved) status changes persist in `localStorage`, so unsaved changes survive a reload until pasted.

## Next step

After opening:

"Your Connection Review Board is open. For each connection: review the note, click **Open profile**, **Copy note**, send the request on LinkedIn, then mark **Sent** here. When you've marked some, click **Copy update for Claude** and paste it to me — I'll save the statuses. Run `/tracker` to see sent outreach logged to your profile."

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md`. Specific cases:

- **No profile / `connectionQueuePath` missing:** route the user to `/setup`. Do not invent a path.
- **Queue file missing or empty:** build the board with `[]` (it shows "No connections"), and tell the user to run `/find-connections` or `/setup-daily-connections`.
- **create_artifact unavailable (non-Cowork environment):** report that the board requires Cowork, and offer a chat-rendered table of the queue instead.
- **Pasted command references unknown ids:** apply the valid ones, report the unknown ids rather than failing the whole batch.

Never abort silently — report what failed and the alternative path.
