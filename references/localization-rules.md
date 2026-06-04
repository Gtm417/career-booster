# CV and application localization rules

Apply these rules in `cv-builder`, `cv-optimizer`, `cv-tailor`, and `application-packager`. Reference this file; do not restate these rules in individual skill files.

---

## General rules (all markets)

- Match the language of every output document to the language of the target job posting.
- If no posting language is available, use the primary language of the target market.
- Never auto-translate silently — present translated documents for user review before storing.
- Date formats: use the local convention (e.g. MM/YYYY for most European markets, MM/YYYY for US/UK).
- Phone number: include the country code (+48, +972, etc.) when the user is applying cross-border.

---

## Poland

- **Language:** Polish for Polish-company postings; English for international companies in Poland.
- **Length:** 1–2 pages.
- **Photo:** Not required but common. Ask the user whether to include a placeholder.
- **Personal data:** Include a RODO clause at the end of every CV. See `references/rodo-clauses.md` for the exact text.
- **Date of birth:** Not required; omit unless the user specifically requests it.
- **Job boards:** Pracuj.pl, LinkedIn, NoFluffJobs, JustJoin.it.

## Israel

- **Language:** Hebrew for Hebrew-language postings; English for English-language postings. Many postings are bilingual — produce the CV in the primary language of the posting.
- **Length:** 1 page maximum regardless of seniority.
- **Photo:** Common but not required. Ask.
- **Personal data:** Israeli ID number is never required on a CV. Omit date of birth.
- **Job boards:** LinkedIn, AllJobs, Jobmaster, Drushim.

## Germany

- **Language:** German for German-language postings; English for English-language postings.
- **Length:** 2 pages for most roles; up to 3 for senior academic/technical roles.
- **Photo:** Expected. Flag to the user that German CVs traditionally include a professional photo. Add a placeholder section if the user confirms.
- **Personal data:** Date of birth and nationality are traditionally included in German CVs (though legally optional). Ask the user whether to include them.
- **Anschreiben:** Cover letters are formally expected and are called Anschreiben. Write in formal German unless the posting is in English.
- **Job boards:** LinkedIn, Xing, StepStone, Indeed.de.

## United Kingdom

- **Language:** English.
- **Length:** 2 pages maximum; 1 page for early career.
- **Photo:** Do not include. Remove if present in the base CV.
- **Personal data:** Remove date of birth, nationality, and marital status. These are legally protected and their inclusion can count against the applicant.
- **Pronoun for summary:** Second person or first person — both acceptable. Be consistent.
- **Job boards:** LinkedIn, Reed, Totaljobs, CV-Library.

## United States

- **Language:** English.
- **Length:** 1 page for under 10 years' experience; 2 pages for senior/executive roles.
- **Photo:** Do not include.
- **Personal data:** Remove date of birth, nationality, marital status, and any other protected information.
- **Document name:** "Resume" not "CV" — use correct terminology in all communications.
- **Job boards:** LinkedIn, Indeed, Glassdoor, Dice (tech roles).

## Other markets

For markets not listed here:
1. Research and apply the local CV convention (photo norms, personal data norms, standard length, document language).
2. Note which convention was applied in the output summary.
3. Flag anything uncertain to the user before finalizing.

---

## How to apply

1. Identify the target market from the job posting or the user's `targets.markets` field.
2. Look up the rules for that market in this file.
3. Apply all applicable rules before outputting the document.
4. In the output summary, list which localization rules were applied.
5. Flag any rules that required user input (e.g. "photo included — confirm you have a professional photo to use").
