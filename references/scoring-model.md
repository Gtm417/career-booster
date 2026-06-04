# Suitability scoring model

Used by `job-finder` and `job-analyzer`. Apply this model identically in both skills. Do not redefine it locally.

## Score calculation

Suitability score is an integer 0–100 computed as a weighted sum across five factors.

| Factor | Weight | What to assess |
|---|---|---|
| Skills match | 40% | How many required skills from the posting match the user's profile (`skills.technical`, `skills.tools`, `skills.soft`). Partial matches count proportionally. Nice-to-haves count at 50% weight vs. must-haves. |
| Experience match | 25% | Does the user's seniority level and total years of experience satisfy the posting's requirement? Over-qualified counts as 80% (not 100%). Under-qualified by 1–2 years counts as 60%. More than 2 years under counts as 20%. |
| Location / work model match | 15% | Does the posting's location and work model (remote / hybrid / on-site) align with the user's `targets.locations` and `targets.workModel`? Exact match = 100%. Flexible/negotiable = 80%. Mismatch = 0%. |
| Industry match | 10% | Does the company's industry match any entry in `targets.industries`? Match = 100%. Adjacent industry = 50%. No overlap = 0%. |
| Role title match | 10% | How closely does the posted job title match the user's `targets.roles`? Exact title = 100%. Synonym or close variant = 80%. Same function, different level = 50%. Different function = 0%. |

## Thresholds and labels

| Score range | Label | Recommendation |
|---|---|---|
| 75–100 | Strong match | Apply |
| 50–74 | Partial match | Consider |
| 0–49 | Weak match | Skip |

## Threshold enforcement

- If a job scores below 50, output the score and label clearly before any further action.
- State the primary reason for the low score.
- Ask the user whether to proceed. Do not proceed without explicit confirmation.
- Do not apply a sub-50 job without user override.

## Tie-breaking

When ranking multiple jobs with equal scores, break ties by posting date (most recent first).
