# SSI Guide — Social Selling Index

## What SSI measures

LinkedIn's Social Selling Index (SSI) is a 0–100 score across four pillars, each worth 0–25 points. It is updated daily. It reflects both profile completeness and behavioral activity on LinkedIn.

SSI is available at `linkedin.com/sales/ssi` (requires login to the audited account).

---

## Overall score thresholds

| Score | Status | Interpretation |
|---|---|---|
| 70–100 | 🟢 Strong | Top quartile. Good recruiter visibility and network positioning. |
| 50–69 | 🟡 Average | Competitive but leaving visibility on the table. |
| 30–49 | 🔴 Weak | Below average. Recruiters and InMail are less likely to surface this profile. |
| < 30 | 🔴 Critical | Very low. Profile and activity need significant investment. |

---

## Pillar 1: Professional Brand (0–25)

**What it measures:** Profile completeness and content quality. Driven primarily by: complete headline, About, Experience, Skills, Photo, and published content (articles, posts).

| Score | Status |
|---|---|
| 20–25 | 🟢 Strong |
| 15–19 | 🟡 Average |
| < 15 | 🔴 Weak |

**Fastest fixes by score gap:**
- Score < 15: Complete the About section, add 3+ Experience descriptions, pin top 3 skills, add a professional photo.
- Score 15–19: Write or share 1–2 LinkedIn posts/articles. Add Featured items. Get 2+ endorsements on top skills.
- Score 20–24: Publish a long-form article or project post. Add 3+ more endorsements.

**Note:** This pillar is the one most directly improvable via `/optimize-linkedin`. Recommend it when Brand score < 20.

---

## Pillar 2: Find the Right People (0–25)

**What it measures:** Proactive searching behavior — using LinkedIn search to find prospects, recruiter profiles, and connections. Not derivable from profile content alone.

| Score | Status |
|---|---|
| 20–25 | 🟢 Strong |
| 15–19 | 🟡 Average |
| < 15 | 🔴 Weak |

**Fastest fixes:**
- Score < 15: Search for target companies and hiring managers on LinkedIn 3–5 times per week. Use filters (location, title, industry).
- Score 15–19: Save searches and revisit them weekly. Use LinkedIn's "People Also Viewed" to find adjacent contacts.
- Score 20–24: Use advanced filters and boolean search strings. Engage with profiles after finding them (don't just view).

**Note:** This pillar cannot be improved by profile edits. It is purely behavioral. Recommend `/find-connections` to give the user a structured outreach cadence.

---

## Pillar 3: Engage with Insights (0–25)

**What it measures:** Content engagement — liking, commenting, sharing, and posting on LinkedIn. Reacting to industry news and posts.

| Score | Status |
|---|---|
| 20–25 | 🟢 Strong |
| 15–19 | 🟡 Average |
| < 15 | 🔴 Weak |

**Fastest fixes:**
- Score < 15: Comment substantively on 3–5 posts per week (not just "great post!"). Share 1 industry article per week with a 2-sentence personal take.
- Score 15–19: Write one original post per week (250–500 words or a short insight). React to content from target companies and hiring managers.
- Score 20–24: Aim for 3+ original posts per week. Use polls, carousels, or document posts for higher engagement.

**Note:** Purely behavioral. Cannot be fixed via profile edits. Mention this to the user if score is low — they may not realize LinkedIn tracks activity.

---

## Pillar 4: Build Relationships (0–25)

**What it measures:** Connection quality and growth. Driven by: number of connections (especially 2nd degree to target companies), InMail response rate, and connection acceptance rate.

| Score | Status |
|---|---|
| 20–25 | 🟢 Strong |
| 15–19 | 🟡 Average |
| < 15 | 🔴 Weak |

**Fastest fixes:**
- Score < 15: Send 5–10 personalized connection requests per week to recruiters, hiring managers, and alumni at target companies.
- Score 15–19: Follow up on accepted connections with a brief message. Engage with their content after connecting.
- Score 20–24: Ask 2–3 existing connections for recommendations. Reconnect with dormant contacts before applying.

**Note:** Recommend `/find-connections` when this pillar is below 15. A profile audit cannot fix this — it requires outreach.

---

## Industry rank and network rank

- **Industry rank %**: Where the user's SSI stands relative to all LinkedIn members in the same industry. Lower % = better (e.g., top 10% means only 10% score higher).
- **Network rank %**: Same comparison but within the user's 1st and 2nd degree connections.

**When to flag:**
- Industry rank worse than 50%: mention that profile and activity are below the industry median.
- Network rank worse than 50%: mention that the user's immediate network is outperforming them — a signal to increase activity.

---

## SSI and profile audit relationship

| SSI gap | Root cause | Fix |
|---|---|---|
| Low Brand | Profile incomplete or poorly written | `/optimize-linkedin` |
| Low Find | Not searching LinkedIn regularly | `/find-connections` for structure |
| Low Engage | Not posting or commenting | Advise content cadence |
| Low Relationships | Few connections or low response rate | `/find-connections` + personalized outreach |

Always cross-reference SSI pillar weaknesses with the profile scorecard. A low Brand score with a high profile scorecard is unusual — it may mean the profile was recently updated and SSI hasn't caught up yet (24–48 hour lag).
