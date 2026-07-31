---
schedule: "0 9,15 * * 1-5"
timeout: 5m
active: true
events: false
consumes: memory
skills: [slack-post]
credentials: [slack_bot_token]
---

Report the agent's recent activity to Slack. Your input is the inbox of
memory changes since the last report; your output is at most one
`chat.postMessage` call to `$SLACK_CHANNEL`. The slack-post skill covers
formatting and sending.

## Execution discipline

Your first and only initial action is to read `inbox.md`. If it says
`No pending changes`, stop immediately: call no other tools, read no
memory files or schedule, and do not look for a ledger. Otherwise, use
only the pending inbox changes and the specific current-state files
needed to compose the report.

## 1. Gate

An empty inbox means nothing happened since the last report: exit without
posting and without consuming. Never post a "nothing to report" message --
a quiet channel is the feature.

## 2. Compose

One message, teammate voice, built from the inbox (use the memory files
for current state, not to re-tell history the inbox already covers):

- **What happened** -- the inbox's new events, grouped and compressed:
  outcomes first, related work merged into one line, links on meaningful
  phrases where the event carried one. NO-OP events (checked, found
  nothing) collapse into a single trailing clause, or drop entirely when
  there are real outcomes.
- **Needs a human** -- any Human-owned task the inbox shows as new or
  transferred, each with its stable id so replies can reference it.
- **Task changes** -- completed or cancelled tasks, one line each.

Keep the whole message under a couple dozen lines. Skip any section with
nothing in it.

## 3. Deliver, then consume

Post via `chat.postMessage`. Delivery is `"ok": true` in the response
body -- Slack returns HTTP 200 even for failures, so the status code
proves nothing. On `ok: true`, consume the inbox. Anything else means the
report did not arrive -- do not consume, and exit; the same changes
return next run.
