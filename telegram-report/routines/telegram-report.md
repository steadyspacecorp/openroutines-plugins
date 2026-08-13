---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
schedule: "0 7 * * 1-5"
timeout: 5m
active: true
reports: true
skills: [telegram-send]
credentials: [telegram_bot_token]
---

Send the agent's daily check-in to the team's Telegram chat the way a
teammate would: once, at the start of the workday, covering everything
since the last one. Your input is your pending changes; your output is
at most one message to `$TELEGRAM_CHAT_ID`. The telegram-send skill
covers formatting and sending.

## 1. Decide whether to send

Check your pending changes before anything else. None means nothing has
happened since the last report: stop right there -- read nothing
further, send nothing, consume nothing. A quiet chat is the feature;
never send a "nothing to report" message.

Then your ledger, which holds the last report you delivered. If it
already covers these exact changes -- a send that landed without
consuming -- consume them now without sending again; the team already
read this report.

Send -- unless the day holds no news at all: every pending event a
NO-OP (or none), and your window empty. "Nothing happened, nothing
planned" is not a check-in; stop without consuming, and the NO-OPs roll
into the next real one. News on either side sends as usual.

## 2. Compose

Write for teammates who can't see the machine. They don't have the
changes, ledgers, or task list you do -- they have thirty seconds and a
scrolling chat. Everything below follows from that.

Voice: a teammate at standup, not a status report generator -- plain
words, contractions welcome. One idea per bullet, one to three short
sentences, never burying the result behind its setup. Say what happened
and what it means for the team, not the machinery underneath: "the CSV
export page shipped without a help doc, so I wrote one" beats
"identified an uncovered surface and emitted a documentation PR".

Every reference carries its own context. A PR, issue, or page gets a
link anchored on the words that describe it -- never a naked URL, and
never trailed after the sentence in parentheses; resolving a bare
relay#484 to a titled link is expected derivation. People the events
name stay named -- never anonymized to "a customer". Task ids are your
own bookkeeping -- name the ask, never the id.

The sections divide the news, and each fact has one home: what happened
owns the past, what's next owns what's coming, needs-a-human owns asks
waiting on a person. Say a fact in its home section and nowhere else --
another section may point at it ("flagged it for a human"), never
restate it. An event whose only content is an ask lives under
needs-a-human and gets no what-happened bullet.

- **What happened** -- your new events plus completed or cancelled
  tasks, summarized into a good update. One bullet per outcome, related
  work merged -- except events labeled NO-OP: drop those. If that
  leaves nothing, one line summarizing the NO-OPs. Anything beyond what
  the events say is invention.

  Events are written for you, not for the team -- full facts and
  receipts. The rewrite drops, it doesn't condense evenly:

  > swept open PRs carrying `keep-fresh` and found one: "Ask AI"
  > (relay#2929)… 142 commits behind, mergeable_state `dirty`… merged
  > `main` in; one add/add conflict in `robot.svg` -- both sides added
  > the identical icon independently. Kept main's version: newer and
  > already shipped. Pushed merge commit `c6c6f05b9`… behind_by now 0…
  > commented on the PR naming the conflict and resolution.

  becomes

  > "Swept open PRs for staleness and brought the one match, [Ask
  > AI](…), current with `main` -- resolving a duplicate robot-icon
  > conflict in favor of the version that already shipped."

  and, had the sweep found nothing,

  > "Swept open PRs for staleness; all current."

  The scope, the outcome, and the judgment call survive -- a sweep's
  reach is what makes the line dense, so keep it. What doesn't survive
  is the machine talking about itself: shas, timings, state
  transitions, the blow-by-blow of what you pushed and commented, and
  the wait for a human. Editorial color ("long-idle", "quick win",
  "thanks!") goes with it. A sentence that says what happened next
  rather than why it matters gets deleted, not shortened.

- **What's next** -- required; never blank. It projects the schedule,
  not the task list: one line per in-window routine, each its mission
  in one short, plain sentence -- "Check recent PRs for any needed doc
  updates", not its mechanics -- with any open Agent-owned tasks
  attached to their routine's line. Open tasks neither add a routine to
  the window nor remove one: a task whose routine is out of window
  waits for that routine's fire day, and a task no routine covers gets
  transferred to Human-owned. Before sending, check the counts: one
  line per in-window routine, and every open Agent-owned task
  accounted for.

  "Nothing is scheduled" is valid only when your window is genuinely
  empty (weekend or holiday ahead) -- never shorthand for a quiet or
  blocked day.

- **Needs a human** -- every Human-owned task your changes show as new
  or transferred, plus any task change naming a dependency it waits on,
  each worded as an ask a teammate could act on. Nothing else -- a wait
  that isn't a tracked ask isn't a blocker. Each ask is raised exactly
  once: a consumed transition never re-presents, while the task stays
  canonical until a human settles it. Most days there are none: skip
  the section rather than saying so.

- **Footer** -- when telegram-inbox is active in this agent (its
  frontmatter says `active: true`; a copy in the agent's routines/
  directory wins over the vendored plugin copy), end the message with
  one italic line that invites replies and names the reply cadence in
  plain words from that routine's schedule: "Reply to this message and
  I'll pick it up -- I check this chat hourly on weekdays." When
  telegram-inbox is inactive, no footer -- never invite replies into a
  void.

Keep the whole message under a dozen short lines.

## 3. Send

One `sendMessage`, per the telegram-send skill; delivery is HTTP 200
with `"ok": true` and a `result.message_id`, and nothing else. On
delivery, record the sent report in your ledger, replacing the last one
-- it's both the double-send guard above and the file a person opens to
catch up on this agent without scrolling the chat.

## 4. Consume

A delivered message is this routine's delivery -- consume the changes.
So is a ledger match in the gate: already-delivered changes get
consumed without a second send. A failed send, a closed gate, or an
all-quiet skip delivered nothing: leave everything unconsumed.
