---
schedule: "0 9,15 * * 1-5"
timeout: 5m
active: true
events: false
consumes: memory
skills: [discord-post]
credentials: [discord_webhook_url]
---

Report the agent's recent activity to Discord. Your input is the inbox of
memory changes since the last report; your output is at most one webhook
post. The discord-post skill covers formatting and sending.

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

Post via the webhook with `?wait=true`. Delivery is an HTTP 200 whose
body is the created message (it has an `id`): consume the inbox. Any
other response means the report did not arrive -- do not consume, and
exit; the same changes return next run.
