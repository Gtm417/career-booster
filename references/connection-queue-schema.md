# Career Booster — Connection Queue Schema

`connections-queue.json` is the **operational staging queue** for LinkedIn connection discovery and the review dashboard. It is distinct from the profile's `outreach[]` array (the canonical system of record). Discovery writes here; only when a connection is actually sent is it mapped into `profile.outreach[]`.

It is a plain file in the canonical career-booster folder so the dashboard artifact can read and write it directly through the Desktop Commander connector.

---

## Storage

**Location:** the canonical per-user folder, alongside the profile —
`<USER_HOME>/.career-booster/connections-queue.json` (`%USERPROFILE%\.career-booster\` on Windows).

Never hardcode this path. Read it from the profile's `connectionQueuePath` field (set by `profile-setup`). If the profile or that field is missing, the user hasn't run `/setup` yet — route them there. The folder is created by `profile-setup`; the queue file is created there on first use.

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
  "dateSent": "YYYY-MM-DD | null"
}
```

---

## Rules

- **id**: `li-` + `dateFound` + `-` + zero-padded sequence (`001`, `002`…), unique within that date. Continue from the highest existing index for today.
- **type**: reuse the profile `outreach[]` enum exactly, so records map 1:1 into `profile.outreach[]` later.
- **status**: starts at `new`. Lifecycle: `new → reviewed → approved → sent` or `→ skipped`. The dashboard advances status; discovery only ever writes `new`.
- **Dedup key**: normalized `linkedinUrl` — lowercase, strip trailing slash, strip query string and fragment. A contact already present in ANY status is never re-added.
- **Never overwrite** existing records on append. Discovery appends only.

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
