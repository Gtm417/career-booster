# Connector failure protocols

When a connector or external tool fails during a skill, apply the appropriate protocol from this file. Do not leave the user without a path forward.

---

## Browser / web access failure

**Affects:** `job-finder`, `job-analyzer`, `market-analyzer`, `linkedin-auditor`, `linkedin-optimizer`, `linkedin-outreach`

**Symptoms:** Tool returns an error, times out, or returns an empty/unusable response.

**Protocol:**
1. Report which site or tool failed: "I couldn't access [site/tool] right now."
2. State what was successfully retrieved before the failure (if anything).
3. Offer alternatives:
   - For job boards: ask the user to paste job listings directly from the site
   - For LinkedIn: ask the user to paste their profile text or specific sections
   - For market research: note that the analysis will be based on training knowledge (which may be less current) and flag this clearly in the output
4. Continue with available information. Do not abort the whole skill.

---

## Gmail / email connector failure

**Affects:** `email-writer`, `application-packager`

**Symptoms:** Cannot send, draft, or access Gmail.

**Protocol:**
1. Report the failure: "I couldn't connect to Gmail right now."
2. Still produce the email draft as text.
3. Offer: "Copy the email below and send it manually, or I can try again when the connection is restored."
4. Do not attempt to send without confirming the connector is working.

---

## File read failure (CV upload)

**Affects:** `cv-parser`

**Symptoms:** Uploaded file cannot be read (binary, corrupted, image-based PDF, unsupported format).

**Protocol:**
1. Report the specific failure: "I couldn't read the uploaded file. It may be image-based, corrupted, or in an unsupported format."
2. Ask the user to paste the CV text directly into the chat.
3. If they paste it, proceed with the text as if it were the file.
4. If the file is an image-based PDF, specifically say: "Your PDF appears to be a scanned image rather than a text document. ATS systems will also fail to read it — I recommend exporting a text-based PDF from your original document before applying."

---

## Memory / profile read failure

**Affects:** All skills

**Symptoms:** Profile cannot be loaded from `career_booster_profile` memory key.

**Protocol:**
1. Report: "I couldn't load your profile. It may not have been set up yet."
2. If this is a first session, offer to run `/setup` immediately.
3. If the profile should exist, ask the user if they'd like to rebuild it or troubleshoot.
4. Do not proceed with any personalized action without a profile — generic output is failure.

---

## LinkedIn connector failure

**Affects:** `linkedin-outreach`

**Symptoms:** Cannot send LinkedIn connection requests or messages.

**Protocol:**
1. Report the failure.
2. Still draft all messages as text.
3. Instruct the user to send them manually: "Copy the connection request below and send it directly from your LinkedIn profile to [Name] at [URL]."
4. Log the outreach attempt in the profile with status `pending` and a note that it was drafted but not sent automatically.

---

## Partial failure (some sources succeed, some fail)

When a multi-source skill (e.g. `job-finder` searching multiple boards) has partial failures:
1. Complete the search using available sources.
2. Report which sources failed and which succeeded at the top of the output.
3. Note that results may be incomplete.
4. Suggest the user check the failed boards manually and provide the direct search URL if possible.
