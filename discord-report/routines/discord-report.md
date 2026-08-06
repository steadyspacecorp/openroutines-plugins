---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
schedule: "0 7 * * 1-5"
timeout: 5m
active: true
teamwork: off
consumes: memory
skills: [discord-post]
credentials: [discord_webhook_url]
---

Report the agent's recent activity to Discord. Your input is the inbox of
memory changes since the last report; your output is at most one webhook
post. The discord-post skill covers formatting and sending.

## Execution discipline

Your first and only initial action is to read `inbox.md`. If it says
`No pending changes`, stop immediately: call no other tools, read no
memory files or schedule, and do not look for a ledger. Otherwise, use
only the pending inbox changes, ./schedule.md, and the specific
current-state files needed to compose the report.

## 1. Gate

An empty inbox means nothing happened since the last report: exit without
posting and without consuming. Never post a "nothing to report" message --
a quiet channel is the feature.

## 2. Compose

One message, written for teammates who can't see the machine: they
don't have your inbox, your ledgers, or your task list -- they have
thirty seconds and a scrolling channel. A teammate at standup, not a
status report generator: plain words, contractions welcome; one idea
per bullet, one to three short sentences, the result never buried
behind its setup. Say what happened and what it means for the team, not
the machinery underneath: "the CSV export page shipped without a help
doc, so I wrote one" beats "identified an uncovered surface and emitted
a documentation PR".

Compression drops, it doesn't condense evenly. The scope, the outcome,
and the judgment call survive; the machine talking about itself dies --
shas, ids, file paths, milestone chains, time estimates, state
transitions, and the blow-by-blow of what you edited and pushed. A line
that says what happened next rather than why it matters gets deleted,
not shortened.

Every reference carries its own context: a PR, issue, or page gets a
link anchored on the words that describe it -- never a naked URL, never
a bare filename in code formatting. People the events name stay named.
Task ids are your own bookkeeping -- name the ask, never the id.

Sections, built from the inbox (memory files supply current state,
never history the inbox already covers), skipping any with nothing in
it:

- **What happened** -- the inbox's new events plus completed or
  cancelled tasks, one bullet per outcome, related work merged. NO-OP
  events (checked, found nothing) collapse into a single trailing
  clause, or drop entirely when there are real outcomes.
- **What's next** -- one plain line per routine in ./schedule.md's
  in-window table: its mission, not its mechanics, with open Agent-owned
  tasks attached to their routine's line.
- **Needs a human** -- every Human-owned task the inbox shows as new or
  transferred, and any task change naming a dependency it waits on,
  worded as an ask a teammate could act on. Most days there are none:
  skip the section rather than saying so.

Keep the whole message under a dozen short lines.

## 3. Deliver, then consume

Post via the webhook with `?wait=true`. Delivery is an HTTP 200 whose
body is the created message (it has an `id`): consume the inbox. Any
other response means the report did not arrive -- do not consume, and
exit; the same changes return next run.
