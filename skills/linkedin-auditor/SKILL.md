---
name: linkedin-auditor
description: |
  Audits a LinkedIn profile against recruiter search behavior and LinkedIn's ranking algorithm. Use when the user runs /audit-linkedin or says "audit my LinkedIn", "review my LinkedIn profile", "score my LinkedIn", "check my LinkedIn", "how good is my LinkedIn", "check my LinkedIn for recruiters", "is my LinkedIn profile strong", or "audit someone's LinkedIn profile". Works for own account or a third-party profile. Requires a LinkedIn profile URL.
allowed-tools: [mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__get_page_text, mcp__Claude_in_Chrome__javascript_tool, mcp__Claude_in_Chrome__find, mcp__Claude_in_Chrome__computer, mcp__Claude_in_Chrome__browser_batch, mcp__Claude_in_Chrome__tabs_context_mcp]
---

# LinkedIn auditor

## Reference files

- `references/linkedin-scorecard-rubric.md` — section weights and 1–5 scoring anchors
- `references/ssi-guide.md` — SSI pillar definitions, interpretation thresholds, and per-pillar fixes
- `references/profile-schema.md` — user profile structure, memory key, and field definitions

## Profile schema

Load the user profile from memory key `career_booster_profile`. Refer to `references/profile-schema.md` for the complete field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules. Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.

---

## STRICT execution order

Execute steps in order. Do not skip, merge, or reorder any step. Do not proceed to the next step until the current one is complete.

---

## Step 1: Pre-flight — two questions, no exceptions

Send exactly this message and wait for both answers before doing anything else:

> Before I start the audit, two quick questions:
>
> **1. Whose account is this?**
> - My own LinkedIn account
> - Someone else's LinkedIn account
>
> **2. Can I use your browser to read the profile automatically?**
> - Yes — use my browser (Chrome)
> - No — I'll paste the content myself

Do not read the profile. Do not ask for any other input. Do not proceed until both answers are received.

Record: `account_mode` = own / third-party. `session_mode` = browser / manual.

---

## Step 2: Collect profile URL, CV, and target role

Ask for URL and CV in one message. Extract the target role from the CV after upload — do not ask for it upfront.

**Profile URL:**
- `account_mode = own`: load from `personal.linkedinUrl` in the stored profile. If present, confirm it with the user rather than asking.
- `account_mode = third-party`: ask for the URL.

**CV / resume:**
- `account_mode = own`: load from `cvs.base` in the stored profile. If present, state that you have it. If absent, ask for an upload.
- `account_mode = third-party`: always ask for an upload.

**Target role:**
- `account_mode = own`: load from `targets.roles[0]` in the stored profile. If present, state it and ask if it's still the target. If absent, ask.
- `account_mode = third-party`: read the uploaded CV and extract the most recent job title or stated target role. Present it for confirmation:
  > Based on the CV, I'm treating the target role as **[extracted title]**. Does that match, or would you like to change it?
  If no role can be extracted from the CV, ask directly.

**SSI — inform, do not collect yet:**
Include this line in the same message so the user can prepare:
> I'll also need an SSI screenshot later. You can get it by opening `linkedin.com/sales/ssi` while logged into the LinkedIn account being audited and screenshotting the full page. Have it ready for the next step.

Do not proceed until URL, CV, and target role are confirmed.

---

## Step 3: Collect SSI data

**If `session_mode = browser` AND `account_mode = own`:** Skip this step entirely. You will navigate to SSI automatically in Step 5.

**All other cases:** Send a dedicated message asking only for the SSI screenshot:

> Please upload the SSI screenshot now. Open `linkedin.com/sales/ssi` while logged into the LinkedIn account being audited, screenshot the full page, and attach it here.
>
> If SSI is not available, just say so and I'll skip that section.

Wait for the screenshot upload or explicit "skip" before proceeding.

---

## Step 4: Read the LinkedIn profile

### Browser mode

Use `tabs_context_mcp` to get a valid tab ID, then navigate to the profile URL.

**LinkedIn public profiles do not require login.** Navigate directly to the URL. Do not ask the user to log in before attempting.

**Scroll to load all content** — LinkedIn lazy-loads sections below the fold. Use `browser_batch` to navigate and scroll in a single batched sequence with waits between each scroll position:

```
browser_batch actions:
1. navigate to profile URL
2. wait 2000ms
3. javascript: window.scrollTo(0, document.body.scrollHeight * 0.25)
4. wait 1500ms
5. javascript: window.scrollTo(0, document.body.scrollHeight * 0.5)
6. wait 1500ms
7. javascript: window.scrollTo(0, document.body.scrollHeight * 0.75)
8. wait 1500ms
9. javascript: window.scrollTo(0, document.body.scrollHeight)
10. wait 2000ms
```

After scrolling, call `get_page_text` to extract the full text content.

**Take a screenshot for photo and banner assessment.** After `get_page_text`, scroll back to the top and call `computer` (screenshot) to capture the profile hero area (name, photo, banner, headline). Describe what is visible: photo quality, framing, banner image or default gray, Open to Work badge.

Capture every visible section from the text:
- Name, location, headline (full text)
- About section (full text, or confirm empty)
- All Experience entries — title, company, dates, full description and bullets
- Education — degree, institution, dates, GPA if shown
- Skills — all listed, especially top 3 pinned
- Featured section — contents, or confirm empty
- Recommendations — count and any visible text
- Certifications and Licenses
- Contact info — email, website links, custom URL
- Open to Work indicator

**Incomplete page load check:** If the page text contains only the header section and Activity (no About, Experience, or Skills sections), the page did not fully load. Attempt one retry: scroll to the top, wait 3 seconds, repeat the scroll sequence. If still incomplete after retry, switch to manual mode.

**If LinkedIn blocks the page or shows a login wall:**
Do not ask the user to log in. Switch to manual mode immediately:
> LinkedIn is blocking direct access to this profile. Please paste the following sections from the profile directly into the chat: headline, About section, each Experience entry (title, company, dates, description), skills list, and any other sections you want assessed. You can also attach a screenshot of the profile if that's easier.

### Manual mode

Ask the user to paste or screenshot the profile content. Accept text, screenshots, or both. Work with whatever is provided. Mark any missing sections as "not assessed" in the scorecard.

Note: Photo and Banner cannot be assessed from text alone. If the user shares a screenshot or describes them, use that. Otherwise mark as "not assessed."

---

## Step 5: Read SSI

### Browser + own account
Navigate to `linkedin.com/sales/ssi`. This page requires the user to be logged into their own LinkedIn account in the browser. Use the same scroll sequence from Step 4 to ensure full page load. Take a screenshot if needed to read scores.

If a login wall appears:
> Please log into LinkedIn in your browser, then let me know and I'll read the SSI page automatically.

Extract:
- Overall SSI score (0–100)
- Four pillar scores (each 0–25): Professional Brand, Find the Right People, Engage with Insights, Build Relationships
- Industry rank % and network rank %

### Screenshot (all other cases)
Read the same values from the uploaded image(s). If the image is unclear, ask the user to type the numbers directly.

### SSI not available
Note in output: *SSI not available — pillar analysis skipped.* Continue with profile scoring.

---

## Step 6: Score the profile

Use `references/linkedin-scorecard-rubric.md` for section weights and 1–5 anchors. Score each section against the target role. Calculate weighted overall score (1–5, one decimal).

**SSI pillar mapping:**
- Headline, About, Experience, Skills, Education & Certs, Featured → **Professional Brand**
- Recommendations (and connection count if visible) → **Build Relationships**
- Find the Right People and Engage with Insights → behavioral signals only. Scores come from SSI data, not profile content.

**Priority flag rules:**
- 🔴 Score ≤ 2 on any section, or score ≤ 3 on a section with weight ≥ 10%
- 🟡 Score ≤ 3, or score ≤ 4 on a high-weight section
- 🟢 Score ≥ 4

---

## Step 7: Output

**LinkedIn Profile Audit — [Name]**
Target role: [role | seniority] · Target market: [geography / remote] · Account: [Own / Third-party] · Date: [YYYY-MM-DD]

---

### Profile scorecard

| Section | Weight | Score | SSI Pillar | Priority | Top fix |
|---|---|---|---|---|---|
| Headline | 20% | X/5 | Brand | 🔴/🟡/🟢 | {one-line fix} |
| About | 15% | X/5 | Brand | 🔴/🟡/🟢 | {one-line fix} |
| Experience — current | 15% | X/5 | Brand | 🔴/🟡/🟢 | {one-line fix} |
| Experience — prior | 10% | X/5 | Brand | 🔴/🟡/🟢 | {one-line fix} |
| Skills | 10% | X/5 | Brand | 🔴/🟡/🟢 | {one-line fix} |
| Recommendations | 10% | X/5 | Relationships | 🔴/🟡/🟢 | {one-line fix} |
| Featured | 5% | X/5 | Brand | 🔴/🟡/🟢 | {one-line fix} |
| Education & Certs | 5% | X/5 | Brand | 🔴/🟡/🟢 | {one-line fix} |
| URL & Contact | 5% | X/5 | — | 🔴/🟡/🟢 | {one-line fix} |
| Photo & Banner | 3% | X/5 | Brand | 🔴/🟡/🟢 | {one-line fix} |
| Open to Work | 2% | X/5 | — | 🔴/🟡/🟢 | {one-line fix} |
| **Profile overall** | **100%** | **X.X/5** | | | |

---

### SSI

**If SSI data is available:**

SSI — [score]/100 · Industry top [X]% · Network top [X]%

| Pillar | Score | Status | Fastest fix |
|---|---|---|---|
| Professional Brand | X/25 | 🔴/🟡/🟢 | {one action} |
| Find the Right People | X/25 | 🔴/🟡/🟢 | {one action} |
| Engage with Insights | X/25 | 🔴/🟡/🟢 | {one action} |
| Build Relationships | X/25 | 🔴/🟡/🟢 | {one action} |

Use `references/ssi-guide.md` for score thresholds and per-pillar fix guidance.

**If SSI data is not available:**
*SSI not available — pillar analysis skipped.*

---

### Verdict
2 sentences max: current profile state + single highest-leverage improvement.

### Top 3 improvements
Three changes with the highest expected impact on recruiter visibility. Each as one sentence: what to change and why it matters.

**Threshold warning** — output only if overall score is below 2.5/5:
> Profile score is [X.X]/5. This significantly reduces recruiter visibility. Run `/optimize-linkedin` before submitting applications.

---

## Step 8: Storage and next step

**Own account only.** Write to `linkedin.lastAudit` in `career_booster_profile`:

| Field | Value |
|---|---|
| `linkedin.lastAudit.date` | ISO8601 timestamp |
| `linkedin.lastAudit.targetRole` | target role used for this audit |
| `linkedin.lastAudit.overallScore` | weighted overall score (1.0–5.0) |
| `linkedin.lastAudit.sectionScores.headline` | section score |
| `linkedin.lastAudit.sectionScores.about` | section score |
| `linkedin.lastAudit.sectionScores.experienceCurrent` | section score |
| `linkedin.lastAudit.sectionScores.experiencePrior` | section score |
| `linkedin.lastAudit.sectionScores.skills` | section score |
| `linkedin.lastAudit.sectionScores.featured` | section score |
| `linkedin.lastAudit.sectionScores.recommendations` | section score |
| `linkedin.lastAudit.sectionScores.educationAndCerts` | section score |
| `linkedin.lastAudit.sectionScores.urlAndContact` | section score |
| `linkedin.lastAudit.sectionScores.openToWork` | section score |
| `linkedin.lastAudit.sectionScores.photoAndBanner` | section score |
| `linkedin.lastAudit.ssi.overallScore` | SSI total (if available) |
| `linkedin.lastAudit.ssi.professionalBrand` | pillar score (if available) |
| `linkedin.lastAudit.ssi.findTheRightPeople` | pillar score (if available) |
| `linkedin.lastAudit.ssi.engageWithInsights` | pillar score (if available) |
| `linkedin.lastAudit.ssi.buildRelationships` | pillar score (if available) |
| `linkedin.lastAudit.ssi.industryRankPct` | industry rank % (if available) |
| `linkedin.lastAudit.ssi.networkRankPct` | network rank % (if available) |
| `linkedin.lastAudit.topIssues` | top issues list from audit |

**Third-party account:** Do not write to the stored profile.

**Next step — own account:**
> Audit complete. Recommended next steps:
> - Run `/optimize-linkedin` to get rewritten sections ready to paste into your profile
> - Run `/find-jobs` to search for roles — a stronger LinkedIn profile improves your visibility in recruiter searches
> - Run `/find-connections` to identify recruiters and hiring managers to reach out to

**Next step — third-party account:**
> Audit complete. Share this report with the profile owner. They can run `/optimize-linkedin` in their own Career Booster session for rewritten sections.

---

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path.