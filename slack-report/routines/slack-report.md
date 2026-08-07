---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
schedule: "0 7 * * 1-5"
timeout: 5m
active: true
reports: true
skills: [slack-post]
credentials: [slack_bot_token]
---

Report the agent's recent activity to Slack. Your input is ./changes.md,
the knowledge changes since the last report; your output is at most one
`chat.postMessage` call to `$SLACK_CHANNEL`. The slack-post skill covers
formatting and sending.

## Execution discipline

Your first and only initial action is to read `./changes.md`. If it says
`No pending changes`, stop immediately: call no other tools, read no
knowledge files or schedule, and do not look for a ledger. Otherwise, use
only the pending changes, ./schedule.md, and the specific
current-state files needed to compose the report.

## 1. Gate

`No pending changes` means nothing happened since the last report: exit
without posting and without consuming. Never post a "nothing to report" message --
a quiet channel is the feature.

The same goes for a day with no news at all: every pending event a
NO-OP and your window empty. "Nothing happened, nothing planned" is not
a report; exit without consuming, and the NO-OPs roll into the next
real one. News on either side posts as usual.

## 2. Compose

One message, written for teammates who can't see the machine: they
don't have the changes, ledgers, or task list you do -- they have
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

The sections divide the news, and each fact has one home: what
happened owns the past, what's next owns what's coming, needs-a-human
owns asks waiting on a person. Say a fact in its home section and
nowhere else -- another section may point at it ("flagged it for a
human"), never restate it. An event whose only content is an ask lives
under needs-a-human and gets no what-happened bullet.

Sections, built from your pending changes (current-state files supply
state, never history the changes already cover), skipping any with
nothing in it:

- **What happened** -- your new events plus completed or
  cancelled tasks, one bullet per outcome, related work merged. NO-OP
  events (checked, found nothing) collapse into a single trailing
  clause, or drop entirely when there are real outcomes.
- **What's next** -- one plain line per in-window routine: its mission,
  not its mechanics, with open Agent-owned tasks attached to their
  routine's line.
- **Needs a human** -- every Human-owned task your changes show as new
  or transferred, and any task change naming a dependency it waits on,
  worded as an ask a teammate could act on. Most days there are none:
  skip the section rather than saying so.

Keep the whole message under a dozen short lines.

## 3. Deliver, then consume

Post via `chat.postMessage`. Delivery is `"ok": true` in the response
body -- Slack returns HTTP 200 even for failures, so the status code
proves nothing. On `ok: true`, consume the changes. Anything else means the
report did not arrive -- do not consume, and exit; the same changes
return next run.
