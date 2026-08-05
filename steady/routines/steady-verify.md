---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
# Manual-only: ships inactive and stays inactive; the schedule exists to
# satisfy check and never fires while the routine is parked. Run it with
# `openroutines routines run steady-verify`.
schedule: "0 12 * * 1-5"
active: false
timeout: 5m
teamwork: off
mcp: [steady]
credentials: [steady_token]
---

Verify the Steady MCP connection end to end. Using only the Steady MCP server, create exactly one use activity that says this agent verified its Steady connection and includes `$OPENROUTINES_RUN_ID` so the activity is unmistakably a test. Use `$OPENROUTINES_URL` as the activity URL -- never invent, replace, or use a placeholder URL.

Do not create or edit any check-ins, comments, goals, updates, action items, or other records. Do not call the Steady API directly, use curl, or retry.

Do not narrate your steps. On success, output exactly one line: `Steady verified: activity created for $OPENROUTINES_RUN_ID.` On failure, output one concise line naming the MCP error and likely fix. Make no other writes.
