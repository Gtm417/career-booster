# Career Booster — Profile Schema

This is the single source of truth for the user profile. Every skill reads from and writes to this schema. Never invent fields. Never store data outside this structure.

---

## Storage mechanism

All Career Booster data lives in one **canonical per-user folder**, resolved at runtime — never hardcoded and never chosen by the user:

```
<USER_HOME>/.career-booster/
   ├─ career_booster_profile.json     # the profile (this schema)
   └─ connections-queue.json          # the connection queue (see connection-queue-schema.md)
```

`<USER_HOME>` is resolved at setup via the Desktop Commander connector:
- Windows: `$env:USERPROFILE` (e.g. `start_process("echo $env:USERPROFILE")`) → `%USERPROFILE%\.career-booster\`
- macOS/Linux: `$HOME` → `~/.career-booster/`

The hidden `.career-booster` folder is created automatically with Desktop Commander `create_directory` (idempotent). The user never creates or selects a folder — every user's home directory always exists, so this path is universal.

The profile file is `career_booster_profile.json` in that folder. When updating, always merge changed fields into the existing profile — read first, patch, then write back. Never overwrite the whole profile in a single write. Tag every write with `_lastUpdated: ISO8601 timestamp`.

The profile records its own storage location so every other skill and the dashboard read one value instead of resolving paths themselves:
- `_storageDir` — absolute path to the canonical folder
- `connectionQueuePath` — absolute path to `connections-queue.json` inside it

---

## Full schema

```json
{
  "_version": "1.0",
  "_createdAt": "ISO8601",
  "_lastUpdated": "ISO8601",
  "_storageDir": "string (absolute path to the canonical career-booster folder)",
  "connectionQueuePath": "string (absolute path to connections-queue.json)",

  "personal": {
    "fullName": "string",
    "email": "string",
    "phone": "string",
    "currentCity": "string",
    "currentCountry": "string",
    "linkedinUrl": "string | null"
  },

  // Note: personal.linkedinUrl is the canonical field for the user's LinkedIn URL.
  // linkedin.url mirrors it and is kept for read convenience within the linkedin namespace.
  // Always write the URL to personal.linkedinUrl. Skills reading the URL should prefer personal.linkedinUrl.

  "targets": {
    "roles": ["string"],
    "seniorityLevel": "entry | mid | senior | lead | executive",
    "locations": ["string"],
    "openToRelocation": "boolean",
    "workModel": "remote | hybrid | onsite | flexible",
    "industries": ["string"],
    "companySizes": ["startup | sme | enterprise"],
    "markets": ["string"],
    "jobBoards": {
      "<market>": ["string"]
    }
  },

  "experience": [
    {
      "jobTitle": "string",
      "company": "string",
      "location": "string",
      "startDate": "YYYY-MM",
      "endDate": "YYYY-MM | null",
      "isCurrent": "boolean",
      "responsibilities": ["string"],
      "achievements": ["string"]
    }
  ],

  "education": [
    {
      "institution": "string",
      "degree": "string",
      "field": "string",
      "startDate": "YYYY-MM | null",
      "endDate": "YYYY-MM | null",
      "gpa": "string | null",
      "notes": "string | null"
    }
  ],

  "skills": {
    "technical": ["string"],
    "tools": [
      {
        "name": "string",
        "proficiency": "beginner | intermediate | advanced | expert"
      }
    ],
    "soft": ["string"],
    "other": ["string"]
  },

  "languages": [
    {
      "language": "string",
      "proficiency": "native | A1 | A2 | B1 | B2 | C1 | C2",
      "notes": "string | null"
    }
  ],

  "certifications": [
    {
      "name": "string",
      "issuer": "string",
      "date": "YYYY-MM | null",
      "url": "string | null"
    }
  ],

  "additionalSections": {
    "publications": ["string"],
    "volunteerWork": ["string"],
    "awards": ["string"],
    "portfolio": "string | null",
    "other": ["string"]
  },

  "cvs": {
    "base": {
      "language": "string (ISO 639-1 code, e.g. 'en', 'pl')",
      "market": "string (e.g. 'Poland', 'Israel', 'Global')",
      "content": "string (full CV text)",
      "source": "uploaded | built | parsed",
      "createdAt": "ISO8601",
      "notes": "string | null"
    },
    "optimized": {
      "<language>": {
        "language": "string",
        "market": "string",
        "content": "string",
        "basedOn": "base | base.<language>",
        "optimizedAt": "ISO8601",
        "changesApplied": ["string"]
      }
    },
    "tailored": {
      "<jobId>": {
        "language": "string",
        "market": "string",
        "content": "string",
        "basedOn": "optimized.<language>",
        "tailoredAt": "ISO8601",
        "jobTitle": "string",
        "company": "string",
        "jobUrl": "string | null",
        "changesApplied": ["string"],
        "gapReport": ["string"]
      }
    }
  },

  "linkedin": {
    "url": "string | null",
    "lastAudit": {
      "date": "ISO8601 | null",
      "targetRole": "string | null",
      "overallScore": "number 1.0–5.0 (one decimal) | null",
      "sectionScores": {
        "headline": "number 1–5 | null",
        "about": "number 1–5 | null",
        "experienceCurrent": "number 1–5 | null",
        "experiencePrior": "number 1–5 | null",
        "skills": "number 1–5 | null",
        "featured": "number 1–5 | null",
        "recommendations": "number 1–5 | null",
        "educationAndCerts": "number 1–5 | null",
        "urlAndContact": "number 1–5 | null",
        "openToWork": "number 1–5 | null",
        "photoAndBanner": "number 1–5 | null"
      },
      "ssi": {
        "overallScore": "integer 0–100 | null",
        "professionalBrand": "integer 0–25 | null",
        "findTheRightPeople": "integer 0–25 | null",
        "engageWithInsights": "integer 0–25 | null",
        "buildRelationships": "integer 0–25 | null",
        "industryRankPct": "number | null",
        "networkRankPct": "number | null"
      },
      "topIssues": ["string"],
      "optimizedSections": {
        "headline": "string | null",
        "about": "string | null",
        "experience": [
          {
            "role": "string",
            "content": "string"
          }
        ],
        "skills": ["string"]
      },
      "optimizedAt": "ISO8601 | null"
    }
  },

  "applications": [
    {
      "id": "string (format: YYYYMMDD-{company-slug}-{title-slug})",
      "jobTitle": "string",
      "company": "string",
      "dateApplied": "YYYY-MM-DD",
      "status": "outreach_sent | applied | viewed | interview_scheduled | interview_completed | offer_received | rejected | withdrawn",
      "language": "string",
      "source": "string (job board name)",
      "jobUrl": "string | null",
      "cvVersion": "tailored.<jobId> | optimized.<language> | base",
      "coverLetterSent": "boolean",
      "outreachEmailSent": "boolean",
      "linkedinOutreachSent": "boolean",
      "notes": "string | null",
      "statusHistory": [
        {
          "status": "string",
          "date": "ISO8601"
        }
      ]
    }
  ],

  "outreach": [
    {
      "id": "string (format: YYYYMMDD-{name-slug})",
      "contactName": "string",
      "contactTitle": "string",
      "company": "string",
      "linkedinUrl": "string | null",
      "type": "hiring_manager | recruiter | peer | alumni | other",
      "connectionRequestSent": "boolean",
      "connectionRequestDate": "YYYY-MM-DD | null",
      "followUpSent": "boolean",
      "followUpDate": "YYYY-MM-DD | null",
      "status": "pending | connected | no_response | replied | meeting_booked",
      "relatedJobId": "string | null",
      "notes": "string | null"
    }
  ],

  "gaps": {
    "profileFields": ["string (field paths with missing data, e.g. 'personal.phone')"],
    "cvSections": ["string"],
    "marketSkills": ["string (skills flagged as in-demand but missing from profile)"],
    "lastGapCheckDate": "ISO8601 | null"
  }
}
```

---

## CV versioning rules

The CV lifecycle is strictly ordered. Skills must not skip levels.

```
raw upload / profile data
        ↓
    base CV          ← set by cv-parser or cv-builder. One per language.
        ↓
  optimized CV       ← set by cv-optimizer. Keyed by language. Based on base.
        ↓
  tailored CV        ← set by cv-tailor or application-packager. Keyed by jobId.
                       Always based on optimized, never directly on base.
```

**Rules:**

1. `cv-builder` and `cv-parser` write to `cvs.base`. They do not write to `optimized` or `tailored`.
2. `cv-optimizer` reads from `cvs.base` and writes to `cvs.optimized[language]`. It does not write to `tailored`.
3. `cv-tailor` and `application-packager` read from `cvs.optimized[language]` and write to `cvs.tailored[jobId]`. If no optimized version exists for the target language, prompt the user to run `/optimize-cv` first.
4. Tailored CVs are never overwritten — each job gets a unique `jobId`. If the same job is re-tailored, append `_v2`, `_v3` etc.
5. Storing a new base CV does not automatically invalidate existing optimized or tailored versions. Flag the user: "You've updated your base CV. Existing optimized versions may be outdated — run `/optimize-cv` to refresh."

---

## Job ID format

`jobId` is used to key tailored CVs and link applications. Format:

```
YYYYMMDD-{company-slug}-{title-slug}
```

Example: `20240315-deloitte-senior-architect`

Slugs: lowercase, hyphens only, max 20 characters each. Truncate if longer.

If the same company/title combination is applied to again on the same date, append `-2`, `-3`, etc.

---

## Status transition rules

Application status must only move forward in this sequence. No skipping, no reversal:

```
outreach_sent → applied → viewed → interview_scheduled →
interview_completed → offer_received
```

Dead-end statuses (can be set from any active state):
- `rejected` — set on explicit rejection signal
- `withdrawn` — set when user decides to withdraw

Every status change appends to `statusHistory` with the date. Never overwrite history.

---

## Conflict resolution

When merging data from a new source (e.g. a newly uploaded CV conflicts with existing profile data), apply these rules:

| Conflict type | Rule |
|---|---|
| New CV parsed, existing work experience differs | Show both versions. Ask user which to keep field by field. Never auto-merge silently. |
| CV in a different language parsed | Store as additional `base` entry for that language. Do not overwrite the existing base. |
| Profile-setup answer contradicts stored data | Newer answer wins. Flag the change and confirm with user. |
| Optimized CV exists, base is updated | Flag outdated optimized versions. Do not delete them. Let user decide whether to re-optimize. |
| Multiple tailored CVs for the same job | Never merge. Always create a new versioned entry. |

---

## Gap tracking rules

When any skill detects a missing required field, write the field path to `gaps.profileFields`. When the gap is filled, remove it from the array.

Skills that run gap analysis:
- `profile-setup` — on setup completion
- `cv-parser` — after extract