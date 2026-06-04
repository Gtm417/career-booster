# ATS compatibility checklist

Apply this checklist in `cv-optimizer` and `cv-builder`. Reference this file; do not restate these rules in individual skill files.

ATS (Applicant Tracking System) parsing fails silently — the CV looks fine to a human but the system extracts nothing, or extracts garbage. These checks prevent that.

---

## Formatting (parser-breaking issues)

- [ ] **No tables** — ATS parsers read tables as a stream of unformatted text. Replace any table layout with flat sections.
- [ ] **No text boxes** — Content inside text boxes is invisible to most ATS systems. Move all content to the main body.
- [ ] **No columns** — Two-column layouts cause parsers to read left and right columns together, producing word-salad. Use a single-column structure.
- [ ] **No headers or footers** — Many parsers skip or corrupt header/footer content. Name and contact info must be in the main body.
- [ ] **No graphics, icons, or images** — Decorative elements are stripped or cause parsing errors. Remove all non-text elements.
- [ ] **No unusual fonts** — Stick to standard fonts (Arial, Calibri, Times New Roman, Georgia). Unusual fonts sometimes render as symbols.
- [ ] **File format** — DOCX is most reliably parsed. PDF is acceptable if it is text-based (not a scanned image). Flag if the uploaded CV is an image-based PDF — it will fail all ATS systems.

## Section labels (keyword matching)

ATS systems match keywords against section labels. Non-standard labels are ignored.

- [ ] Work history section is labeled "Work Experience", "Experience", or the local-language equivalent
- [ ] Education section is labeled "Education" or "Academic Background"
- [ ] Skills section is labeled "Skills" or "Technical Skills" — not "Expertise" or "Toolkit"
- [ ] If a summary exists, label it "Professional Summary", "Summary", or "Profile" — not "About Me"

## Contact information

- [ ] Name is on its own line at the top
- [ ] Email address is in plain text (not hidden in a hyperlink)
- [ ] Phone number is in plain text with country code if cross-border
- [ ] LinkedIn URL is in plain text (not embedded as a hyperlink only)
- [ ] No contact info is inside a table or header

## Dates

- [ ] All dates follow a consistent format (e.g. MM/YYYY or Month YYYY)
- [ ] Date ranges use a consistent separator (e.g. "–" or "to")
- [ ] "Present" or the local equivalent is used for current roles — not left blank

## Keywords

- [ ] Primary job title from the target posting appears in the professional summary
- [ ] At least 60% of the required skills from the target posting appear somewhere in the CV
- [ ] Acronyms and spelled-out forms both appear where relevant (e.g. "Search Engine Optimization (SEO)")

## Gaps and consistency

- [ ] No unexplained employment gaps longer than 3 months (if gaps exist, add a brief note)
- [ ] Job titles, company names, and dates are consistent across CV and LinkedIn profile
- [ ] No contradictory information (e.g. dates that overlap, skills listed as both "expert" and "beginner")

---

## How to apply

Run through every checkbox before outputting a CV. In the output summary, report:
- Which checks passed
- Which checks failed and what was fixed
- Any checks that could not be resolved without user input (e.g. "The uploaded CV appears to be an image-based PDF. Please export a text-based version.")
