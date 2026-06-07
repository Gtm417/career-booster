---
description: Open the Connection Review Board to review, copy, and act on discovered LinkedIn connections.
---

The user has run /connection-dashboard. Execute the `connection-dashboard` skill exactly as written.

Key points (do NOT deviate — these were proven by testing):
- The board uses an EMBED + PASTE model. The artifact sandbox cannot read/write local files (local MCP connectors return 400) and has no chat write-back API.
- Read connections-queue.json AGENT-SIDE (path = profile `connectionQueuePath`), bake the `connections` array into references/dashboard.html via the `__CONNECTIONS_JSON__` and `__QUEUE_PATH__` tokens, then create/update the `connection-review-board` artifact with `mcp_tools: []`.
- Do NOT add Local file system / Desktop Commander calls to the artifact. Do NOT have the artifact write the queue.
- Status changes are persisted when the user pastes the board's "Apply status changes…" command back into chat; the skill then writes them to the queue and rebuilds the board.

If the user passed an argument (e.g. a complaint that the board was rebuilt incorrectly), treat it as context: rebuild the board per the embed+paste design above.
