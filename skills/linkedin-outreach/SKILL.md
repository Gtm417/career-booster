---
name: linkedin-outreach
description: |
  Finds relevant LinkedIn contacts and drafts personalized connection requests. Use when the user runs /find-connections or says "find recruiters on LinkedIn", "who should I connect with", "find hiring managers", "find people at this company", "draft a LinkedIn connection message", "find contacts for this role", or "help me with LinkedIn outreach". Requires a target company, role, or industry.
allowed-tools: [mcp__Claude_in_Chrome__list_connected_browsers, mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__get_page_text, mcp__Claude_in_Chrome__find, mcp__Claude_in_Chrome__form_input, WebSearch, mcp__cowork__list_artifacts, mcp__cowork__create_artifact, mcp__cowork__update_artifact]
---

# LinkedIn outreach

> **File access:** This skill reads and writes the connection queue using your built-in file access. The queue path is **not** hardcoded — read it from the profile's `connectionQueuePath` field (set by `profile-setup` to `<WORKSPACE>/career-booster/connections-queue.json`, inside the connected workspace folder). No filesystem connector is needed. If no profile exists, route the user to `/setup` first.

## Profile schema

Load the user profile from `career_booster_profile.json` in the workspace `career-booster/` folder (see `references/profile-schema.md` for the path, field definitions, CV versioning rules, job ID format, status transition rules, conflict resolution, and gap tracking rules). Never invent fields outside the schema. Always merge changes rather than overwriting; always update `_lastUpdated` on every write.

The connection queue (`connections-queue.json`) is a separate operational store defined in `references/connection-queue-schema.md`. It is the staging surface for discovered connections and the review dashboard; the profile's `outreach[]` array remains the canonical record for sent connections.



## Purpose

Identify the right people to connect with on LinkedIn and write personalized, human-sounding messages for each. Never send without explicit user confirmation.

## Authenticity guardrails (HARD CONSTRAINTS — always apply, never skip)

These are non-negotiable and must hold for every note. Detailed templates and fallback logic live in `references/connection-message-rules.md` (loaded in Step 2); the constraints below stay inline so they are always in context.

1. **The note has one job: get accepted and open the channel.** It is never the place for an ask — no "are you hiring?", no "can you refer me?", no "I'm looking for a job."
2. **Honest material only.** State only verifiable facts about the person plus the role's neutral, generally-true expression of interest. **Never fabricate a specific reaction, opinion, or claim of having read/seen something** (e.g. "I loved your post on X"). Honest generic always beats fake specific.
3. **Generic fallback.** If a person has no usable factual hook, use the role's template as-is — never invent specificity to fill the gap.
4. **Preserve the user's voice;** do not over-polish into generic LinkedIn-speak.
5. **Never auto-send.** Surface finished drafts for review before any send (interactive runs).

## Process

### Step 1.5: Load queue config, count, and dedup set (do this first)

Load the profile to get `connectionQueuePath`, then read the connection queue (see `references/connection-queue-schema.md`).

- If the queue does not exist, create it from the skeleton with `config.dailyTarget` = 5.
- **How many to find this run (`N`):** if the user passed a count in the command (e.g. `/find-connections 2 …`), use it. Otherwise use `config.dailyTarget`. Clamp to 1–15.
- Read `config.targetRoles` / `config.targetCompanies` if set; otherwise fall back to the profile's `targets.roles`, `targets.locations`, and industries/companies of interest.
- **Account type (note quota):** read the account type (Free vs Premium) from the profile if present; otherwise assume **Free** and note the assumption. This governs the personalized-note quota — see `references/connection-message-rules.md` §5. On Free accounts, prioritize personalized notes for the highest-value tiers (alumni, recruiters) and flag contacts that may only support a note-free request.
- **Per-role volume:** if the user specified how many to prepare per role, honor it; otherwise distribute `N` across the priority tiers. Also confirm the alumni inputs are available (university + grad year, former employers) since alumni notes depend on them.
- Build the dedup set: **normalized** `linkedinUrl` values already in the queue (lowercase, strip trailing slash, query, fragment). Never surface anyone already present, in ANY status.

### Step 1.6: Establish the discovery channel (do this BEFORE searching — do not skip)

Claude in Chrome is the **preferred** channel; WebSearch is a last resort. You MUST actively verify Chrome before falling back — never assume it is unavailable just because the tools aren't in front of you.

1. **Make sure the Claude-in-Chrome tools are loaded.** In this runtime they may be *deferred* (present but not yet in your toolbox). If you do not currently see `mcp__Claude_in_Chrome__list_connected_browsers` and the other Chrome tools, **load them first** (e.g. via tool search for "Claude in Chrome" / "browser") before drawing any conclusion. Not seeing the tools is NOT evidence the browser is unavailable — it usually just means they haven't been loaded yet.
2. **Probe for a connected browser.** Call `mcp__Claude_in_Chrome__list_connected_browsers`.
   - If it returns one or more browsers → Chrome is available. Use it.
   - If it returns none (or the tool genuinely cannot be loaded in this environment) → only then consider WebSearch.
3. **Confirm LinkedIn login (Chrome path).** Navigate to `https://www.linkedin.com/feed/` (or the people-search URL) and check for a login wall. If logged in → proceed with Chrome. If a login wall appears: interactive run — tell the user "Chrome is connected but not logged into LinkedIn; log in and re-run, or I'll use the shallower WebSearch fallback" and wait; scheduled/unattended run — note it and fall back to WebSearch.
4. **State the decision and the reason**, e.g. "Channel: Claude in Chrome (browser connected, LinkedIn logged in)" or "Channel: WebSearch fallback — no connected browser found." Never fall back silently.

### Step 1: Identify targets (multi-channel)

Run the search on the channel established in Step 1.6:

- **Claude in Chrome** (chosen when a browser is connected and LinkedIn is logged in): run a LinkedIn people search (`/search/results/people/?keywords=…`) using role + location + target companies. Chrome sees degree (1st/2nd/3rd), mutual connections, and live profile detail. Tag these contacts `source: "chrome"`.
- **WebSearch** (only when Step 1.6 ruled out Chrome): query `site:linkedin.com/in <role> <location> <company>`. This is broad and login-free but shallow — it returns only Google-indexed snippets, no degree/mutuals, and titles may be stale. Tag these `source: "websearch"`.

State which channel you used (carry the decision and reason from Step 1.6). Domain matters: a generic role keyword (e.g. "architect") may surface the wrong field (software vs building) — refine with domain terms or target known firms by name.

Priority order for who to surface:
1. **Hiring managers** — "Head of", "Director of", "VP of", "Lead" in the relevant department
2. **Recruiters** — internal recruiters / talent acquisition / HR at target companies
3. **Peers** — same role at target companies (potential referrers)
4. **Alumni** — shared university or previous employer

For each target capture: name, title, company, profile URL, mutual connections (Chrome only), why relevant, `source`. Carry at most `N` new (non-duplicate) contacts into Step 2.

### Step 1.7: Verify titles (correctness gate)

Do NOT assert a `contactTitle` you have not read off the actual profile:
- **Chrome:** if the title isn't clearly visible in the search card, open the profile (`navigate` + `get_page_text`) and read it. Set `titleVerified: true`.
- **WebSearch:** snippet titles are unverified. Either open the profile in Chrome to confirm (`titleVerified: true`), or keep the snippet title with `titleVerified: false` and add "(title unverified — from search snippet)" to `whyRelevant`. Never state an unverified title as fact inside the outreach `message`.

### Step 2: Draft connection requests

**Load `references/connection-message-rules.md` now** — it holds the four per-role templates, their no-hook variants, the note-quota rules, and the worked examples. The inline Authenticity guardrails above are the non-negotiable summary; the reference file is the source of the actual wording.

For each identified contact, select the template matching its role bucket (alumni / recruiter / peer / hiring manager) and fill bracketed fields **from verified data only**:

- Use the role template that matches the tier the contact came from in Step 1.
- Personalize with **one** verifiable hook (shared school/employer, verified title/team/company, mutual connection). If no usable hook exists, use the template's **no-hook variant as-is** — never invent specificity.
- Never state an unverified title as fact (see Step 1.7).
- Keep under 300 characters; record the exact count.
- No ask of any kind in the note (per guardrail 1).
- Do not over-polish into generic LinkedIn-speak; do not use LinkedIn's default phrasing.

Because all four role templates are honest-by-construction (factual hook or honest-generic), drafting is fully automated end-to-end — no mid-process input from the user is required.

### Step 3: Output

**Group output by role bucket** (Hiring managers → Recruiters → Peers → Alumni). For each contact, return a block with:

- **Name + role bucket** (1–4) and profile URL
- **Factual hook + its source** — the exact verifiable detail used and where it came from (e.g. "Profile headline read via Chrome" / "shared university on profile"), so the user can verify. If the no-hook variant was used, state "no specific hook — generic template (honest)".
- **Title verification flag** (`titleVerified` true/false, per Step 1.7)
- **The drafted connection note** — send-ready, with character count
- **Note-quota flag** where relevant (e.g. "Free account — may only support a note-free Connect")
- **Status: awaiting your review**

Present all blocks for user review before any action. (Scheduled runs skip this presentation per the execution-context note below.)

### Step 4: Persist to the connection queue

For each contact identified and drafted this run that is NOT in the dedup set:

1. Build a connection record per `references/connection-queue-schema.md`. Generate `id` as `li-<dateFound>-NNN` (sequence continuing from the highest existing index for today). Set `status: "new"`, `dateFound` = today, `dateSent: null`, `messageChars` = the exact character count of `message`, plus `source` (`chrome`/`websearch`) and `titleVerified` (from Step 1.7).
2. Map `type` to the profile outreach enum (`hiring_manager | recruiter | peer | alumni | other`) based on the priority tier the contact came from in Step 1.
3. Append all new records to `connections`. Never overwrite existing records.
4. Update `_lastUpdated`. Use read-modify-write: read the full file, append, write back the whole object to `connectionQueuePath` using your file access.
5. Report: "Added N new connections to the queue (M skipped as duplicates). Queue total: T."

Persistence is non-destructive — it never sends. Do not gate this step on confirmation.

### Step 4.5: Refresh the review board (best-effort, non-fatal)

After the queue is written, refresh the connection dashboard so newly found contacts appear without the user manually re-running `/connection-dashboard`. This step must **never** fail the run — the queue write in Step 4 is the durable result; a board refresh failure is only reported, not raised.

1. Derive the per-workspace artifact id per `references/connection-queue-schema.md` → "Dashboard artifact id" (`connection-review-board-<workspace-slug>`, from `connectionQueuePath`).
2. Read the dashboard template at `skills/connection-dashboard/references/dashboard.html`, substitute `__QUEUE_PATH__`, `__CONNECTIONS_JSON__`, and `__DRIVE_TOOL__` exactly as the `connection-dashboard` skill does (Step 2 there), and write to a scratch file.
3. Call `list_artifacts`. If the derived id exists, `update_artifact`; otherwise `create_artifact` (create-or-update — so even an unattended daily run leaves a visible, current board). Set `mcp_tools` to the resolved Drive tool name if any, else `[]`.
4. If any part of this fails (artifact tools unavailable, non-Cowork environment, etc.), report it in the summary and continue — do not abort. Append to the Step 4 summary: "Board refreshed." or "Board refresh skipped: <reason>."

### Execution context: interactive vs scheduled

This skill runs in two contexts:

- **Interactive** (user typed `/find-connections`): present the drafted contacts for review (Step 3), then persist to the queue (Step 4).
- **Scheduled** (daily task via `setup-daily-connections`): no user is present. Skip the Step 3 review presentation, persist directly with status `new`, emit the one-line summary, and never attempt to send.

## Sending

Never send a connection request or message without explicit user confirmation. For each contact, ask: "Send this connection request to [Name] at [Company]?" and wait for yes/no.

After confirmed sends, log the outreach in the application tracker with status "Outreach sent".

## Next step

After outreach messages are drafted and confirmed:

"Outreach logged. Recommended next steps:
- Check back in 7–10 days — if connections haven't responded, run `/write-email` or `/find-connections` again for a follow-up pass
- Run `/apply` if any of these contacts are associated with a specific role you want to package"

## Failure handling

Apply the appropriate protocol from `references/connector-failure-protocols.md` if any connector or file read fails during this skill. Never abort silently — always report what failed, what succeeded, and offer an alternative path. Specific cases for the queue persistence step:

- **Queue file unreadable / corrupt JSON:** do not overwrite. Report the corruption, write findings to a sidecar `connections-queue.recovered.json`, and stop. Never silently reset the queue.
- **No profile / `connectionQueuePath` missing:** the user hasn't run `/setup`. Route them to `/setup` (it creates the folder and queue), then retry. Do not invent a path.
- **Claude in Chrome / LinkedIn failure mid-run:** persist whatever was successfully gathered before the failure; report partial success (X found, Y persisted) and what failed.
- **Dedup ambiguity (same person, different URL):** treat distinct normalized URLs as distinct people; note possible duplicates in the summary rather than auto-merging.
