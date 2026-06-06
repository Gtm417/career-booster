---
name: profile-setup
description: |
  Builds the user's Career Booster profile from scratch through guided conversation. Use when the user runs /setup, starts a first session with no stored profile, or says "set up my profile", "create my profile", "I'm new", or "get started". Also triggers when Career Booster detects no profile exists at session start.
allowed-tools: [Read, Write]
---

# Profile setup

## Profile schema

Load the user profile from `career_booster_profile.json` in the canonical storage folder (see "Storage location" below and `references/profile-schema.md`). Refer to the schema for the complete field definitions, versioning rules, conflict resolution, and gap tracking rules. Write all collected data to the schema structure defined there. Never invent fields outside it.

Run this skill on first use or when the user explicitly requests a profile rebuild.

## Storage location (resolve first — the user chooses nothing)

Before collecting data, establish the canonical per-user storage folder via the Desktop Commander connector:

1. Resolve `<USER_HOME>`: on Windows `start_process("echo $env:USERPROFILE")`; on macOS/Linux `echo $HOME`. Take the trimmed output.
2. Build `storageDir = <USER_HOME>/.career-booster` (i.e. `%USERPROFILE%\.career-booster` on Windows, `~/.career-booster` on Unix).
3. Create it (idempotent): `create_directory(storageDir)`.
4. The profile file is `storageDir/career_booster_profile.json`; the connection queue is `storageDir/connections-queue.json`.

Record both absolute paths in the profile: set `_storageDir` and `connectionQueuePath`. Every other skill and the dashboard read these values rather than resolving paths themselves. Never hardcode a path; never ask the user where to put files.

## Behavior

Ask questions one topic at a time. Never present a numbered list or a form. Never ask two questions in one message. Wait for the answer, confirm it, then move to the next topic. If the user gives a partial answer, ask a targeted follow-up before advancing.

**Pacing rule:** One question per message. One answer per question. Repeat if the user is vague. Advance only when the current item is clear.

## Information to collect (in order)

Ask each item below in sequence. Do not reveal the sequence to the user — it should feel like a conversation, not intake.

1. **Personal details** — ask for full name and email. Then phone. Then current city and country.
2. **Target location(s)** — ask where they want to work. Then ask if they're open to relocation.
3. **Target role(s)** — ask what job titles they're targeting. Then ask for seniority level.
4. **Specialization** — ask what field or industry. Then ask for total years of relevant experience.
5. **Skills and tools** — ask for their main technical skills. Then ask which tools and software they use regularly and at what level.
6. **Languages** — ask which languages they speak. For each, ask for the proficiency level (native / CEFR A1–C2 or equivalent).
7. **Work model preference** — ask: remote, hybrid, or on-site?
8. **Target industries and company types** — ask which industries they're targeting. Then ask for preferred company size.
9. **Target job boards** — ask which job sites they plan to use. If targeting specific markets, suggest known local boards for that market (e.g. Pracuj.pl for Poland, AllJobs / LinkedIn for Israel, Indeed for global). Record each board under `targets.jobBoards[market]`.
10. **LinkedIn profile** — ask if they have a LinkedIn profile. If yes, record the URL in `personal.linkedinUrl`. If no, note it as null.
11. **Portfolio or additional links** — ask if they have a portfolio, GitHub, Behance, or similar link. If yes, record in `additionalSections`. If no, move on.

## If a CV is uploaded during setup

If the user uploads a CV at any point during this conversation, pause the intake and invoke `cv-parser` internally. Let `cv-parser` extract all fields it can. Then resume the intake, skipping any questions that were answered by the CV. Fill in the gaps that `cv-parser` could not determine.

## Gap analysis

After all items are collected, compare the stored profile against the full schema in `references/profile-schema.md`. Write the path of every missing required field to `gaps.profileFields`. Mark `gaps.lastGapCheckDate` with the current ISO8601 timestamp.

Required fields for a complete profile:
- `personal.fullName`, `personal.email`, `personal.currentCity`, `personal.currentCountry`
- `targets.roles` (at least one), `targets.locations` (at least one), `targets.workModel`
- `experience` (at least one entry with jobTitle, company, startDate)
- `languages` (at least one entry)

If any required field is missing after the full intake, surface the gaps clearly: "Your profile is missing a few things: [list]. You can fill these in now or come back later."

## Storage

After confirming all collected data with the user, write the profile to `career_booster_profile.json` in the resolved `storageDir` (see "Storage location"). Structure it exactly as defined in `references/profile-schema.md`. Set `_version: "1.0"`, `_createdAt`, `_lastUpdated` (current ISO8601), and the storage fields `_storageDir` and `connectionQueuePath` (= `storageDir/connections-queue.json`).

Also create an empty connection queue at `connectionQueuePath` if it does not exist, using the skeleton in `references/connection-queue-schema.md`, so downstream skills have a target.

If a profile already exists (this is a rebuild), read the existing file from `storageDir` first. Merge the new data using the conflict resolution rules in `references/profile-schema.md`. Never overwrite the full file — patch and write back.

## Confirmation and next step

After saving, confirm: "Your Career Booster profile is set up and saved."

Then surface the most logical next step based on what is present:
- If no CV is stored: "Next, share your CV so I can parse it — upload the file or paste the text. Run `/parse-cv` or just drop the file here."
- If a CV is stored but not optimized: "Your CV is saved. Run `/optimize-cv` to improve it for ATS and recruiter attractiveness."
- If everything is present: "You're ready. Run `/find-jobs` to search for matching roles, or `/analyze-market` to see market conditions for your target role."
