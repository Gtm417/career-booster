---
description: Schedule a daily job that automatically finds new LinkedIn connections and adds them to your review queue.
argument-hint: "[count] [time, e.g. 08:00]"
---

The user has run /setup-daily-connections. Execute the `setup-daily-connections` skill: provision (or update) a recurring daily scheduled task that runs the `linkedin-outreach` skill in unattended mode to discover new LinkedIn connections and append them to connections-queue.json. If the user provided a count and/or time as arguments, use them; otherwise default to 5 connections at 08:00 local time. Never send connection requests — discovery and persistence only. Confirm the schedule and surface `/connection-dashboard` as the next step.
