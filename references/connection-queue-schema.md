# Career Booster — Connection Queue Schema

`connections-queue.json` is the **operational staging queue** for LinkedIn connection discovery and the review dashboard. It is distinct from the profile's `outreach[]` array (the canonical system of record). Discovery writes here; only when a connection is actually sent is it mapped into `profile.outreach[]`.

It is a plain file in the canonical career-booster folder. The agent-side skills (find-connections, the scheduled task, the dashboard build) read/write it directly. The dashboard *artifact* cannot touch local files (sandbox limit), so it renders an embedded snapshot and exports status changes for the user to paste back — see the connection-dashboard skill.

---

## Storage

**Location:** the `career-booster/` subfolder of the connected workspace folder, alongside the profile —
`<WORKSPACE>/career-booster/connections-queue.json`.

Never hardcode this path. Read it from the profile's `connectionQueuePath` field (set by `profile-setup`). If the profile or that field is missing, the user hasn't run `/setup` yet — route them there. The folder is created by `profile-setup`; the queue file is created there on first use. Because it lives inside a connected folder, it is reachable by native Read/Write — no filesystem connector required.

Always read-modify-write the whole object; never overwrite blindly. Update `_lastUpdated` on every write.

---

## File skeleton (create on first run if absent)

```json
{
  "_version": "1.0",
  "_lastUpdated": "ISO8601",
  "config": {
    "dailyTarget": 5,
    "targetRoles": [],
    "targetCompanies": []
  },
  "connections": []
}
```

## Connection record

```json
{
  "id": "li-YYYY-MM-DD-NNN",
  "contactName": "string",
  "contactTitle": "string",
  "company": "string",
  "linkedinUrl": "string (normalized)",
  "type": "hiring_manager | recruiter | peer | alumni | other",
  "whyRelevant": "string",
  "message": "string (<300 chars)",
  "messageChars": "integer",
  "status": "new | reviewed | approved | sent | skipped",
  "dateFound": "YYYY-MM-DD",
  "dateSent": "YYYY-MM-DD | null",
  "source": "chrome | websearch",
  "titleVerified": "boolean (true only if contactTitle was read off the actual profile page)"
}
```

---

## Rules

- **id**: `li-` + `dateFound` + `-` + zero-padded sequence (`001`, `002`…), unique within that date. Continue from the highest existing index for today.
- **type**: reuse the profile `outreach[]` enum exactly, so records map 1:1 into `profile.outreach[]` later.
- **status**: starts at `new`. Lifecycle: `new → reviewed → approved → sent` or `→ skipped`. The dashboard advances status; discovery only ever writes `new`.
- **Dedup key**: normalized `linkedinUrl` — lowercase, strip trailing slash, strip query string and fragment. A contact already present in ANY status is never re-added.
- **Never overwrite** existing records on append. Discovery appends only.
- **source**: which channel found the contact — `chrome` (Claude in Chrome, live profile) or `websearch` (Google/Bing index, may be stale).
- **titleVerified**: `true` only if `contactTitle` was read directly off the profile page. If a title came from a search snippet and wasn't profile-confirmed, set `false` and note it in `whyRelevant`. Never assert an unverified title as fact in the outreach `message`.

---

## Mapping to `profile.outreach[]` (on send)

When a connection reaches `sent`, create/update the corresponding `profile.outreach[]` entry:

| queue field | profile.outreach field |
|---|---|
| contactName | contactName |
| contactTitle | contactTitle |
| company | company |
| linkedinUrl | linkedinUrl |
| type | type |
| (on send) | connectionRequestSent = true |
| dateSent | connectionRequestDate |

The profile `outreach[]` id uses its own format (`YYYYMMDD-{name-slug}`); generate it at map time.

---

## Dashboard artifact id (per workspace)

Connection boards are **per workspace** — each connected project folder gets its own review board so separate searches (e.g. for different people) never overwrite each other. Both `connection-dashboard` (which builds the board) and `linkedin-outreach` (which refreshes it after discovery) must derive the **same** id from the same input, deterministically:

- **Input:** the workspace folder — the parent directory of the `career-booster/` folder that holds `connectionQueuePath`. (For `<WS>/career-booster/connections-queue.json`, the workspace folder is `<WS>`.)
- **Slug:** take the workspace folder's base name, lowercase it, replace every run of non-alphanumeric characters with a single hyphen, and trim leading/trailing hyphens.
- **Artifact id:** `connection-review-board-<slug>`.

Example: workspace `AI career booster assistant` → slug `ai-career-booster-assistant` → id `connection-review-board-ai-career-booster-assistant`.

If the slug is empty (unnameable path), fall back to the bare id `connection-review-board`. Compute this id the same way everywhere; never hardcode a specific board id.

---

## Drive export tool (build-time resolution)

The dashboard's optional "Upload to Drive (CSV)" button calls a Google Drive connector from inside the artifact. The connector's tool name is **per user/install**, so it must never be hardcoded in the shipped HTML. The `connection-dashboard` skill resolves it at build time and substitutes the `__DRIVE_TOOL__` placeholder:

- If the user has a Google Drive connector, substitute the fully-qualified `create_file` tool name (e.g. `mcp__<drive-server-uuid>__create_file`) and include that exact name in the artifact's `mcp_tools`.
- If no Drive connector is available, substitute an empty string. The button stays hidden and `mcp_tools` remains `[]`.

The button uploads the CSV as `text/csv`, which Drive converts to a Google Sheet. It is interactive-only (a present user triggers it); the daily scheduled run never calls Drive — background Drive access is unavailable.
