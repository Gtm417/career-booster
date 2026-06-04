# LinkedIn Authoring Specs & Optimization Protocol (2026)

Authoritative reference for character limits, 2026 algorithm facts, section authoring rules, coherence/cross-validation protocol, and completeness checklist. All Career Booster skills that write or evaluate LinkedIn content reference this file. Numbers here take precedence over any conflicting figures in other references, including `linkedin-scorecard-rubric.md`.

Sources: independently verified across multiple 2026 sources (AuthoredUp analysis of 372K posts, Pollen, Derrick, Jobsprout, RecruitBPM, Jobright, Postipy, CareerBldr, W3Era).

---

## Character limits

| Section | Hard limit | Visible without action |
|---|---|---|
| Headline | 220 (240 on mobile) | ~60 chars in search results and connection requests; ~40–50 in mobile notifications |
| About | 2,600 | ~200–300 chars before "See more" |
| Experience description | 2,000 per entry | — |
| Position title | 100 | Highest search-weight field within Experience |
| Skills | 50 platform max | Only pinned top 3 visible in collapsed profile view |
| Featured section | — | 3–6 items recommended; mobile shows only 1 before swipe |
| Connection request | 200 (free) / 300 (Premium) | — |

---

## Search ranking (2026 consensus)

### Key weights

| Factor | Effect |
|---|---|
| Headline | Highest-weighted field — indexed at approximately 5× other fields |
| Profile completeness (All-Star) | ~40× more recruiter searches than incomplete profiles |
| Experience titles + descriptions | Second tier — quantification required for AI parsing |
| Skills (pinned, endorsed) | Third tier; each skill needs ≥1 endorsement to count |
| Cross-section keyword coherence | Compounds across all sections — determines the profile's "trust score" |

### The 2026 shift — semantic entity mapping

LinkedIn now scores profiles as a whole entity via its Knowledge Graph and an LLM-based retrieval engine. The algorithm maps relationships between skills, titles, companies, and industries across the entire profile. A headline claiming expertise that the Experience and Skills sections do not substantiate receives a lower trust score regardless of connection count or posting frequency.

**Coherence across sections is the defining ranking lever in 2026 — not keyword density.**

Practical implications:
- Keyword stuffing is penalised
- Every claim in the headline and About must be evidenced elsewhere in the profile
- The AI Hiring Assistant reads profiles semantically and generates value propositions for recruiters — every word in Experience, About, and Skills is directly discoverable
- Recruiters increasingly use natural-language prompts, not Boolean strings — depth and context outperform repetition
- Strategic profile updates every 2–4 weeks trigger a recency signal and a temporary ranking boost

---

## Section authoring rules

### Headline (220 chars)

- **Front-load:** primary role + strongest keyword within the first ~60 characters — this is what appears in search results, connection requests, and comment threads
- **Use all 220 characters** — headlines over 100 characters significantly outperform shorter ones in search impressions
- **Format:** `[Primary Role] | [Value proposition] | [Credibility marker] | [Secondary keyword]`
- **Separators:** pipes ( | ) throughout; avoid emojis except sparingly in industries where they are culturally appropriate
- **Acronyms:** for non-obvious terms, list both: "Machine Learning (ML)" — semantic search clusters related terms, but non-obvious abbreviations still miss some Boolean searches
- **Never:** use only a job title — it wastes up to 190 characters of the highest-weighted field

### About section (2,600 chars)

Only ~200–300 characters are visible before "See more." Apply the 4-part framework in order:

**1 — Hook (~200–300 chars)**
Who you are, what you do, and one compelling signal. Must include primary keywords. This is the only part most visitors read without clicking — if it doesn't compel, the rest doesn't matter.

**2 — Proof (body)**
3–6 specific achievements with numbers: revenue, teams, projects, delivery timelines, savings. Short scannable lines, 2–3 sentences per paragraph. Weave in secondary keywords naturally — this section is Google-indexed.

**3 — Repository (transition)**
Tell visitors what they will find on the profile and manage expectations about posting frequency. Example: "I'm not active in the daily feed — my Featured section contains [case studies / project write-ups]. Scroll down to explore [specific section]." This reduces bounce from visitors expecting a content-heavy feed.

**4 — CTA (final lines)**
Direct and specific. **Include an email address.** Fewer than 15% of profiles include contact information; those that do receive approximately 200% more outreach. Example: "Open to [role type] opportunities in [market]. Reach me directly at [email]."

**Voice and formatting:**
- First person throughout — third person reads like a press release
- No markdown formatting (LinkedIn does not render it)
- No Unicode bold/italic text (breaks screen readers, undermines accessibility)
- Short paragraphs with generous line breaks
- URLs can be included but are not clickable inside About

### Experience descriptions (2,000 chars per entry)

**Structure:** narrative paragraph establishing scope and context → bulleted achievements with metrics.

**Every bullet must follow:** Action verb + context + **quantified result**

- Correct: "Reduced construction delivery timeline by 18% across 12 residential projects by introducing phased BIM coordination"
- Incorrect: "Improved project delivery" — unquantified results are treated as filler by the AI ranking system

**Position titles (100-char limit):**
- Make specific and searchable: "Architect" → "Senior Architect — Residential & Commercial | Sustainable Design"
- Semantic search surfaces candidates without exact title matches, so cross-section coherence matters more than any single title string — but specific titles still improve discovery for traditional keyword searches
- Lead with the highest-impact bullet; 3–5 bullets per role

### Skills section

- **Recommended count: 15–20 highly relevant skills** — quality over quantity; generic or weak skills (e.g. "Microsoft Word", "Teamwork") dilute the list and signal poor prioritisation
- **Pin top 3:** pinned skills receive approximately 10× more algorithmic visibility; only pinned skills display in the collapsed profile view
- **Order:** hard/technical skills first; soft skills last — soft skills carry minimal algorithmic weight without recommendation backing
- **Tagging:** associate each skill in the LinkedIn UI with the relevant Experience entry, Education, or Certification — this cross-validation strengthens the algorithm's confidence that the skill is genuine
- **Endorsements:** a skill requires at least 1 endorsement to factor into search rankings; 50+ endorsements significantly outperform. Reciprocity loop: endorse 10–15 genuine connections quarterly, and LinkedIn prompts reciprocal endorsements automatically
- **Acronyms:** list both forms for non-obvious abbreviations

### Featured section

- **Count: 3–6 items** — more creates clutter, fewer leaves prime real estate empty
- **Lead with the strongest:** mobile shows only the first item without swiping; desktop shows approximately 2 large cards before scrolling
- **Priority order (highest conversion impact first):**
  1. Flagship case study, portfolio link, or product demo
  2. LinkedIn Article (Google-indexed, evergreen)
  3. PDF carousel — high Save rates create a long-lived distribution flywheel
  4. Media appearance, speaking deck, or demo video
  5. Booking or consultation link
- **Reorder manually** — LinkedIn defaults to newest first, which is rarely the strongest
- Update quarterly to stay aligned with current positioning

---

## Coherence / cross-validation protocol

Run after all sections are drafted, before delivering to the user. Mandatory — not optional polish. This is the primary defence against the 2026 algorithm's trust-score penalty for incoherent profiles.

### Step 1 — Extract claims
List every role, specialization, skill, industry, and achievement claim made in the **approved headline** and **About section**.

### Step 2 — Run two tests per claim

| Test | Pass condition |
|---|---|
| Skill test | A corresponding skill exists in the Skills section |
| Evidence test | At least one Experience bullet substantiates the claim with a quantified result |

A claim passes only if it passes BOTH tests.

### Step 3 — Run the reverse test (orphan skills)
For each skill in the Skills section, check:
- Does it appear (or is it clearly implied) in the About?
- Does at least one Experience bullet demonstrate it in action?

A skill with no About mention and no Experience evidence is an **orphan skill** — a low-trust signal to the algorithm.

### Step 4 — Produce the coherence report

Deliver before any storage write:

> **Coherence check:**
> - ✅ Validated claims: [list each claim and where it is evidenced]
> - ⚠️ Orphan claims (in headline/About but no matching skill or evidence bullet):
>   — "[Claim]": missing [skill / experience bullet] → suggested fix: [...]
> - ⚠️ Orphan skills (listed but no About mention or Experience evidence):
>   — "[Skill]": not evidenced → suggested fix: [add to X role bullet / remove from list]

### Step 5 — Resolve and confirm
Ask the user to resolve each flag or explicitly accept it. Unresolved orphan claims and orphan skills lower the profile's trust score — state this plainly. Do not write to the profile until the user responds.

---

## Completeness checklist

Run after the coherence check. For every missing item, write the field path to `gaps.profileFields` in the user profile per `references/profile-schema.md`.

| Item | Impact | Gap field path to write |
|---|---|---|
| Custom URL (linkedin.com/in/name) | ~40% more views; Google indexes the headline from the URL title | `personal.linkedinUrl` (flag if non-canonical or absent) |
| Email in Contact Info | Profiles with contact info receive ~200% more outreach | `personal.email` |
| Profile photo | 21× more views, 36× more messages vs no photo | `linkedin.photo` |
| Banner image | First visual impression; mobile (57% of traffic) crops left/right | `linkedin.banner` |
| Projects section (1+ entries) | Project name field is highly search-weighted by the algorithm | `linkedin.projects` |
| 3+ Recommendations | Profiles with recommendations receive ~14× more views | `linkedin.recommendations` |
| Identity verification (CLEAR / Persona) | Verified profiles receive ~30% more messages; Trust Score boost in search | `linkedin.verification` |
| Public visibility toggled ON | Required for Google and Bing indexing | `linkedin.settings.public` |

**All-Star status** requires: photo, 2+ experience entries, 5+ skills, education, About, 50+ connections. Reaching All-Star is the single largest low-effort ranking lever — approximately 40× more recruiter searches.

---

## Sync note for skill authors

This file is the single numeric source of truth for all LinkedIn-related Career Booster skills. `references/linkedin-scorecard-rubric.md` (used by `linkedin-auditor`) must use the same section weights and limits as defined here. If a figure conflicts between the two files, this file takes precedence — update the rubric to match, do not update this file to match the rubric.
