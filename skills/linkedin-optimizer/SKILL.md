---
name: linkedin-optimizer
description: |
  Rewrites LinkedIn profile sections to maximize recruiter search visibility and profile attractiveness. Use when the user runs /optimize-linkedin or says "rewrite my LinkedIn", "improve my LinkedIn profile", "optimize my LinkedIn headline", "rewrite my about section", "optimize my LinkedIn for recruiters", or "help me update my LinkedIn". Uses the stored audit snapshot if available; otherwise audits first.
allowed-tools: [mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__get_page_text]
---

# LinkedIn optimizer

## Reference files

- `references/linkedin-specs.md` — character limits, 2026 algorithm facts, section authoring rules, coherence/cross-validation protocol, completeness checklist
- `references/localization-rules.md` — market-specific conventions (see Localization section below if this file covers CV norms only)
- `references/profile-schema.md` — user profile structure, memory key, and field definitions
- `references/connector-failure-protocols.md` — failure handling

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.

## Purpose

Rewrite each LinkedIn profile section to maximize recruiter discovery (search ranking) and profile conversion (visitors → contact). Deliver sections separately for user review. After all sections are approved, run a mandatory coherence/cross-validation pass, then a completeness check. Do not skip either closing step.

## Pre-flight

If an audit snapshot is stored in the profile (`linkedin.lastAudit`) and is less than 30 days old, use it as the baseline. Otherwise, run the linkedin-auditor logic first to establish baseline scores before rewriting anything.

## Rewrite process

Rewrite each section in order. Deliver each one separately and ask "Does this look right?" before moving to the next. All character limits and rules are in `references/linkedin-specs.md` — apply them exactly.

---

### 1. Headline

Rules:
- **220 characters total** (240 on mobile) — use them all; headlines over 100 characters significantly outperform shorter ones in recruiter search impressions
- **Front-load:** primary role + strongest keyword within the first **~60 characters** — this is what appears in search results, connection requests, and comment threads
- **Format:** `[Primary Role] | [Value proposition] | [Credibility marker] | [Secondary keyword]`
- Use pipe ( | ) separators throughout
- For non-obvious acronyms, list both forms: "Building Information Modelling (BIM)"
- Never use only a job title — it wastes up to 190 characters of the highest-weighted field on the platform

Deliver: new headline + explanation of why each keyword was placed where it was

---

### 2. About section

Rules:
- Limit: **2,600 characters**; only ~200–300 are visible before "See more"
- Apply the 4-part framework from `references/linkedin-specs.md` in strict order:

  **Hook (~200–300 chars):** who you are, what you do, one compelling signal. Primary keywords must appear here. This is the only part most visitors read without clicking.

  **Proof (body):** 3–6 specific achievements with numbers. Short scannable lines, 2–3 sentences per paragraph. Weave in secondary keywords — this section is Google-indexed.

  **Repository (transition):** tell visitors what they'll find on the profile and manage expectations about posting frequency. Example: "I'm not active in the daily feed — my Featured section contains [case studies / project write-ups]. Scroll down to explore [specific section]."

  **CTA (final lines):** direct and specific. **Include an email address.** Fewer than 15% of profiles do this; those that do receive approximately 200% more outreach. Example: "Open to [role type] opportunities in [market]. Reach me directly at [email]."

- First person throughout
- No markdown formatting (LinkedIn does not render it)
- No Unicode bold/italic text (breaks screen readers)
- Short paragraphs with generous line breaks

Deliver: full About section draft + character count

---

### 3. Experience section (top 2–3 most recent roles)

For each role, work in this order:

**Step A — Position title check**
Is the current title specific and searchable? If it is generic (e.g. "Architect", "Manager"), propose a richer version within the 100-character limit. Example: "Architect" → "Senior Architect — Residential & Commercial | Sustainable Design". Confirm the proposed title with the user before writing the description.

**Step B — Description rewrite**
Structure: narrative paragraph establishing scope and context → bulleted achievements with metrics.

Every bullet must follow: **Action verb + context + quantified result**
- Correct: "Reduced construction delivery timeline by 18% across 12 residential projects by introducing phased BIM coordination"
- Incorrect: "Improved project delivery" — unquantified results are treated as filler by the AI ranking system

- Lead with the highest-impact bullet
- 3–5 bullets per role
- Weave relevant keywords in naturally

Deliver: title proposal then description for each role, separately, with confirmation between them

---

### 4. Skills section

Rules:
- Recommend **15–20 highly relevant skills** — quality over quantity; generic or weak skills (e.g. "Teamwork", "Microsoft Word") dilute the list and signal poor prioritisation to the algorithm
- Hard/technical skills first; soft skills last — soft skills carry minimal algorithmic weight without recommendation backing
- **Identify the top 3 to pin** — pinned skills receive approximately 10× more algorithmic visibility and are the only skills visible in the collapsed profile view
- For non-obvious acronyms, list both forms
- After delivering the list, give the user these two instructions:
    1. **Tag each skill** in the LinkedIn UI to the relevant Experience entry or Certification — this cross-validation strengthens the algorithm's confidence that the skill is genuine
    2. **Endorsement loop** — endorse 10–15 genuine connections for skills they actually have; LinkedIn prompts reciprocal endorsements automatically. A skill with zero endorsements does not factor into search rankings.

Deliver: ordered skills list with top 3 pinned skills clearly marked + tagging and endorsement instructions

---

### 5. Featured section

Rules:
- Recommend **3–6 items** — fewer leaves prime real estate empty; more creates clutter
- **Lead with the strongest, most conversion-ready piece** — mobile shows only the first item without swiping; desktop shows approximately 2 large cards initially
- Suitable items (in priority order): flagship case study or portfolio link, LinkedIn Article (Google-indexed), PDF carousel, media appearance or speaking deck, booking or consultation link
- Tell the user to **reorder manually** after adding — LinkedIn defaults to newest first, which is rarely the strongest piece

Deliver: ordered list of recommended Featured items with rationale for the placement order

---

## Coherence / cross-validation pass

Run this after **all five sections are confirmed** by the user. Do not skip or merge with section delivery.

Apply the full protocol from `references/linkedin-specs.md` — Coherence/cross-validation protocol section. Steps in brief:

1. Extract every role, specialization, skill, industry, and achievement claim from the confirmed headline and About section
2. For each claim, test: (a) a corresponding skill exists in the Skills list, and (b) at least one Experience bullet substantiates it with a quantified result — a claim must pass both tests
3. For each skill in the Skills list, test: does the About mention it and does an Experience bullet demonstrate it in action? Skills failing both are orphan skills.
4. Deliver the coherence report before any storage write:

> **Coherence check:**
> - ✅ Validated claims: [list each claim and where it is evidenced]
> - ⚠️ Orphan claims (in headline/About but no matching skill or evidence bullet):
    >   — "[Claim]": missing [skill / evidence bullet] → suggested fix: [...]
> - ⚠️ Orphan skills (listed but no About mention or Experience evidence):
    >   — "[Skill]": not evidenced → suggested fix: [add bullet to X role / remove from list]

5. Ask the user to resolve each flag or explicitly accept it. State plainly: unresolved orphan claims and orphan skills lower the profile's trust score in the 2026 algorithm. Do not write to the profile until the user responds.

---

## Completeness checklist

Run immediately after the coherence check is resolved. Check each item and report status to the user. For every missing item, write the field path to `gaps.profileFields` in the user profile.

Full checklist and gap field paths are in `references/linkedin-specs.md` — Completeness checklist section.

Items to check: custom URL, email in Contact Info, profile photo, banner image, Projects section (1+ entries), 3+ Recommendations, identity verification (CLEAR/Persona), public visibility toggled ON.

Report format:
> **Completeness check:**
> - ✅ [Item]: present
> - ⚠️ [Item]: missing — [one-sentence impact statement]

After reporting, remind the user: All-Star status (photo, 2+ experience entries, 5+ skills, education, About, 50+ connections) unlocks approximately 40× more recruiter searches — the single largest low-effort ranking lever available.

---

## Localization

Refer to `references/localization-rules.md` for market-specific conventions.

If that file covers CV norms only and does not address LinkedIn profiles, apply these LinkedIn-specific defaults:

- **Poland:** English primary profile is standard for international and corporate roles. Recommend a localized Polish headline and About variant if the user is targeting domestic Polish companies. Ensure target-role keywords exist in both languages for recruiter search coverage.
- **Israel:** English primary is standard. Hebrew variant is optional for domestic roles only.
- **Single-language markets:** one language throughout.

For users targeting multiple markets, flag whether a bilingual setup is recommended and identify which specific sections to duplicate.

---

## Output format

Deliver each section as a clean, paste-ready block with no markdown formatting — LinkedIn does not render it. The user should be able to copy and paste each block directly into the corresponding LinkedIn field without modification.

---

## Storage

After the coherence check is resolved and completeness check is complete, write to the profile:

1. `linkedin.lastAudit.optimizedSections.headline` — confirmed headline
2. `linkedin.lastAudit.optimizedSections.about` — confirmed About
3. `linkedin.lastAudit.optimizedSections.experience[]` — confirmed role rewrites
4. `linkedin.lastAudit.optimizedSections.skills[]` — confirmed skills list
5. `linkedin.lastAudit.optimizedAt` — current ISO8601 timestamp
6. `gaps.profileFields` — any completeness items flagged as missing
7. `_lastUpdated` — current ISO8601 timestamp

---

## Next step

After all sections, the coherence check, and the completeness check are complete, deliver:

> "Your LinkedIn sections are ready to paste in. A few things before you go:
> - Turn off **Activity Broadcast** in Settings before making changes to avoid notifying your network for every edit, then turn it back on when done
> - After adding Featured items, **reorder them manually** — LinkedIn defaults to newest first
> - If any completeness gaps were flagged, the custom URL and public visibility toggle are the highest-leverage quick wins
>
> Recommended next steps:
> - Run `/find-connections` to identify recruiters and hiring managers in your target markets to connect with
> - Run `/find-jobs` — your stronger profile will improve visibility in recruiter searches immediately"

---

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path.