---
description: Open the live Connection Review Board to review, copy, and act on discovered LinkedIn connections.
---

The user has run /connection-dashboard. Execute the `connection-dashboard` skill: build and open (or refresh) the persistent Cowork artifact that renders connections-queue.json as a review board with per-contact copy, open-profile, and status actions. First verify the Local file system connector's read/write tool names and response shape, substitute them into references/dashboard.html, then create or update the `connection-review-board` artifact. The board reads and writes the queue directly and never sends connection requests.
