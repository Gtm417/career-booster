---
name: connection-dashboard
description: |
  Opens the persistent Connection Review Board — a live dashboard of all discovered LinkedIn connections, where the user reviews, copies notes, opens profiles, and marks status. Use when the user runs /connection-dashboard or says "open my connections board", "show my connection queue", "review my connections", "open the connection dashboard", "where are my daily connections", or "review board". Reads and writes connections-queue.json.
allowed-tools: [mcp__cowork__create_artifact, mcp__cowork__list_artifacts, mcp__cowork__update_artifact]
---

# Connection dashboard

## Purpose

Create (or refresh) a persistent Cowork artifact that renders `connections-queue.json` as a review board. Each card shows a contact's name, title, company, why-relevant, the drafted connection note (with copy button and character count), an "Open profile" link, and status buttons (Mark reviewed / Mark sent / Skip). The board filters by status and company, sorts, and writes status changes back to the queue file directly.

The dashboard HTML is packaged with this skill at `references/dashboard.html`. This skill ships in the plugin; the **artifact instance** it creates is runtime and lives in the user's Cowork sidebar.

## File access architecture

The dashboard is a sandboxed page: it cannot use built-in file tools, only MCP tools via `callMcpTool`. Cowork's directory access does NOT expose callable file tools to artifacts, so the dashboard uses the **Desktop Commander** MCP connector (verified working), which exposes callable file tools:

- read: `mcp__Desktop_Commander__read_file({ path })` → returns `{ fileName, content }` where `content` is the file text **prefixed by a `[Reading N lines ...]` header that must be stripped** (the template's `extractQueue()` handles this — it slices from the first `{`).
- write: `mcp__Desktop_Commander__write_file({ path, content, mode: "rewrite" })` → overwrites the file.

(The `linkedin-outreach` skill and the scheduled task do NOT use Desktop Commander — they read/write the queue through the agent's native file access. Desktop Commander exists solely to give the sandboxed artifact file access. Both touch the same `connections-queue.json`.)

## Preconditions

1. **Desktop Commander connector enabled.** If its tools aren't present, the dashboard can't read/write — report and stop, or render the queue in chat and update statuses via conversation as a fallback.
2. **Write line limit.** Desktop Commander's `fileWriteLineLimit` defaults to 50. A full-file rewrite of the queue exceeds this once there are ~4+ connections. Before relying on write-back, raise it via `set_config_value({ key: "fileWriteLineLimit", value: 2000 })`, or the status writes may be truncated.
3. **Profile + queue exist.** The queue path comes from the profile's `connectionQueuePath`. If there's no profile, route the user to `/setup`; if the queue file is missing, `/find-connections` or `/setup-daily-connections` creates it.

## Step 1: Probe and verify (REQUIRED)

1. Call `mcp__Desktop_Commander__read_file({ path: "<queue path>" })` once in chat. Confirm it resolves and returns `{ fileName, content }` with the file text inside `content`.
2. Confirm `extractQueue()` parses it (it strips the `[Reading ...]` header and JSON-parses). If Desktop Commander's response shape has changed, adjust `extractQueue()`.
3. Confirm `write_file` resolves and `fileWriteLineLimit` is raised (precondition 2).

Do not skip this — a wrong tool name or unhandled response shape silently breaks the board.

## Step 2: Build the artifact HTML

1. Load the profile and read `connectionQueuePath` (set by `profile-setup`, e.g. `<USER_HOME>/.career-booster/connections-queue.json`). If no profile exists, route the user to `/setup` first.
2. Read `references/dashboard.html` from this skill.
3. Substitute `__QUEUE_PATH__` with the profile's `connectionQueuePath` value (never a hardcoded path).
4. Write the substituted document to a scratch file.

## Step 3: Create or update the artifact

- Call `list_artifacts`. If an artifact with id `connection-review-board` exists, use `update_artifact`; otherwise `create_artifact`.
- `id`: `connection-review-board`
- `html_path`: the scratch file from Step 2
- `description`: `Live review board for daily-discovered LinkedIn connections, backed by connections-queue.json.`
- `mcp_tools`: `["mcp__Desktop_Commander__read_file", "mcp__Desktop_Commander__write_file"]` — the only tools the HTML calls.

## Output

Confirm: the board is open in the sidebar, how many connections it currently shows (by status), and the queue path it is bound to.

## Notes on behavior

- The dashboard is the **sole writer** of the queue file. It uses single-record read-modify-write on each status change (re-reads fresh, mutates one record, writes back) to avoid clobbering appends from the daily task.
- It does **not** send anything. "Open profile" opens LinkedIn in the browser; the user pastes the copied note and sends manually (Phase 4 will add assisted send).
- The artifact has a built-in Reload button in its header — the HTML must not add its own.
- View preferences (filter, company, sort) persist in `localStorage`.

## Next step

After opening:

"Your Connection Review Board is open. Each morning's discovered connections appear here automatically (if you've run `/setup-daily-connections`). For each one: review the note, click **Open profile**, **Copy note**, send the request on LinkedIn, then **Mark sent** here. Run `/tracker` to see committed outreach logged to your profile."

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md`. Specific cases:

- **Desktop Commander tools not found:** the connector isn't enabled. Report it and fall back to rendering the queue in chat and updating statuses via conversation until it's available.
- **Status writes silently truncate / file corrupted after a write:** `fileWriteLineLimit` is too low — raise it via `set_config_value({ key: "fileWriteLineLimit", value: 2000 })` and re-save.
- **`write_file` access denied / path outside allowed directory:** Desktop Commander's `allowedDirectories` excludes the queue path. Report the path that must be allowed.
- **Queue file missing:** route the user to `/find-connections` or `/setup-daily-connections`.
- **create_artifact unavailable (non-Cowork environment):** report that the live dashboard requires Cowork, and offer a chat-rendered table of the queue instead.

Never abort silently — report what failed and the alternative path.
