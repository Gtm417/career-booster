# Connection message rules

Authoritative content for drafting LinkedIn connection notes. The `linkedin-outreach`
skill loads this file before drafting (Step 2). The short, non-negotiable guardrail block
lives inline in `SKILL.md`; this file holds the detailed per-role templates and decision logic.

---

## 1. Prime directive

The note has exactly **one job: get accepted and open the channel.** It is never the
place for an ask. Authenticity is preserved by using only **honest material** — verifiable
facts about the person plus the role's neutral, generally-true expression of interest.

**Honest generic always beats fake specific.** Never fabricate a specific reaction,
opinion, or claim of having read/seen something (no "I loved your post on X", no "your
talk changed how I think"). If you did not verifiably observe it, you may not say it.

---

## 2. Hard constraints (every note)

1. **One true detail max.** State only verifiable facts about the person, plus the role's
   honest-generic interest. Never attribute a specific reaction/opinion to the user.
2. **No ask in the note.** No "are you hiring?", no "can you refer me?", no "I'm looking
   for a job." The note's only job is to get accepted and open the channel.
3. **Generic fallback.** If a person has no usable factual hook, use the role's template
   as-is. Never invent specificity to fill the gap.
4. **300-character cap** (~2–4 sentences). Record the exact count.
5. **Skip LinkedIn default phrasing** ("I'd love to add you to my network").
6. **Preserve the user's voice** — do not over-polish into generic LinkedIn-speak.
7. **Never auto-send.** Surface drafts for review (interactive runs).

### What counts as a verifiable hook (allowed)

- Shared school / graduation year / former employer (alumni — read off their profile)
- Their current title, team, company, location (title must pass Step 1.7 verification)
- A mutual connection visible in Chrome
- A public, factual attribute of their company's domain ("works in [field] at [Company]")

### What is NOT allowed (fabrication)

- Any claim of having read, watched, or reacted to their content
- Any invented opinion, compliment, or emotional reaction
- Any unverified title stated as fact (see `SKILL.md` Step 1.7)
- Filling a missing hook with a guess

---

## 3. Per-role templates

Each role differs by **incentive**, which sets the framing. All four are fully
automatable because their substance is either factual or honest-generic. Substitute
bracketed fields from verified data only; if a field cannot be verified, fall back to the
role's no-hook variant rather than guessing.

### 3.1 Alumni — substance is factual (warmest, highest accept rate)

Hook is "we share [school/employer]" — a fact, nothing to fabricate.

> Hi [Name] — fellow [University] grad (Class of [year])! Saw we share the alma mater and
> that you're at [Company]. I'm in [field] and always glad to connect with fellow alumni.
> Hope you're well.

**No-hook variant** (shared employer instead of school):
> Hi [Name] — looks like we both spent time at [Former Employer]. I'm in [field] now and
> always glad to connect with former colleagues from there. Hope you're doing well.

### 3.2 Recruiters — substance is factual (most receptive: a relevant candidate is useful)

Hand them a concise "search string"; never dump a CV.

> Hi [Name] — I'm a [role/specialization] targeting [location], exploring opportunities in
> [field]. Connecting with recruiters who place in this space. Happy to share more if useful.

**No-hook variant** (recruiter, company unknown/unverified):
> Hi [Name] — I'm a [role/specialization] targeting [location]. I like to stay connected
> with recruiters in [field]. Happy to share more if it's ever useful.

### 3.3 Peers — honest generic (step one toward a future referral)

Craft-to-craft framing. **Zero mention of jobs or referrals.** No specific reaction needed.

**Voice rules specific to this template (do not skip):**
- **Self-describe, don't assert shared role.** Never open with "fellow [role] here" — that
  states the *contact's* role as fact, which is rarely verified (Step 1.7 verifies the
  contact's title, not that it matches the sender's field). Say "I'm a/an [role] working in
  [specialization]" instead — a claim only about the sender, always true.
- **Comma greeting, not em dash.** "Hi [Name]," — not "Hi [Name] —". The dash reads as
  templated.
- **No closing tagline.** Stop after the location/interest line. Do not append a flourish
  like "always up for trading notes," "excited to connect," or similar — these are the
  generic LinkedIn-speak guardrail #6 (`SKILL.md`) already prohibits; the old template text
  itself violated it.

> Hi [Name], I'm a/an [role] working in [specialization]. I'm expanding my network in
> [location] and would love to connect with [specialists/people] in [city].

**No-hook variant** (contact's field/location unverified):
> Hi [Name], I'm a/an [role] working in [specialization]. I'm expanding my network in
> [location] and would be glad to connect.

### 3.4 Hiring managers — honest generic (a job-ask reads as noise, so no ask)

A light, true expression of interest in their team's work — never escalate to a fabricated
specific compliment.

> Hi [Name] — I follow the work [Company]'s [team/department] is doing in [field] and find
> it interesting. I work in [my domain] and like staying connected with leaders in this
> space. Glad to connect.

**No-hook variant** (manager, team/department unverified):
> Hi [Name] — I work in [my domain] and like staying connected with leaders in [field].
> Glad to connect.

---

## 4. Account type and note quota (read before drafting)

- **Free accounts:** LinkedIn limits personalized connection notes (a monthly quota, plus
  many 3rd-degree profiles allow only note-free "Connect"). Prioritize personalized notes
  for the highest-value tiers (alumni, recruiters) and flag in output when a contact may
  only support a note-free request.
- **Premium accounts:** higher note allowance — personalize all tiers.

When account type is unknown, assume Free, personalize the top tiers, and note the
assumption in the summary so the user can adjust.

---

## 5. Scope

This skill drafts the **connection note only** — the message that gets the request
accepted and opens the channel. It does not draft post-acceptance follow-ups. Any later
messaging is handled separately (e.g. `/write-email`).
