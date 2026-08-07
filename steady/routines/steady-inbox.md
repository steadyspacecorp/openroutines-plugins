---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
schedule: "45 8,17 * * 1-5"
trigger:
  poll: https://service.steady.space/api/v2/digest?category=comment&per_page=1
  select: /0/id
  interval: 5m
  credential: steady_token
timeout: 10m
teamwork: off
mcp: [steady]
credentials: [steady_token]
---

Watch Steady and keep knowledge current. Replying to comments is your only
Steady write. Most runs find nothing new: end quickly.

## 1. Gate on the digest

Fetch the digest since a day before your newest ledger entry (no
ledger yet → 3 days back) and sort its entries:

- A comment entry that is not yours, not in your ledger, and addressed
  to you (on your resources, or @mentioning you on anyone's) → step 2.
- A teammate's check-in or goal update entry you haven't seen → step 3.
- Nothing in either bucket and the goal board isn't due (step 4) →
  stop. Most runs end here, one call in.

When a reply needs older context than the digest shows, pull the
parent by date.

## 2. Answer comments

A comment is handled when its id is in your ledger or a comment of
yours on the same resource has a later timestamp (compare timestamps, not list
position). Reply to every unhandled comment by someone else, no
exceptions:

- **Action request** → record an Agent-owned task (stable id; source:
  requester + resource); reply "on it," naming when — the next fire of
  the routine whose domain covers it, per the schedule.
- **Question or feedback** → answer in-thread.
- **Answer to a Human-owned task** → resolve the
  task in place: delete it if the human settled it, transfer it to
  Agent-owned if the ask became agent work, cancel it if declined;
  acknowledge briefly.
- **Anything else** (FYI, status update, someone claiming work) →
  acknowledge briefly; if it settles or claims something, step 3
  applies too.

Post the reply on the resource the comment is on — one reply may
answer several pending comments there. Casual teammate voice: short
and warm, not corporate. After a reply posts, ledger the comment's
id; a failed post gets no entry.

## 3. Update knowledge from teammates' work

From the teammate check-in and goal update entries the digest
surfaced:

- A human explicitly settled a Human-owned task (the ask itself, not
  agent work near the topic) → resolve it as in step 2.
- A human's work covers an open Agent-owned task → mark it done
  ([x]), crediting them.
- Refresh your standing context with in-flight or claimed work overlapping
  your own lanes — the domains your routines cover: who, what, firm or
  tentative, date; drop entries older than about a week.
- A goal entry → update its goal's line on the board (step 4) from the
  entry and ledger its id; a goal the board doesn't know gets one pull
  to fill in the rest.

Beyond these uses, don't act on teammates' content.

## 4. Groom

- Mark open Agent-owned tasks done when your recorded events or your
  check-ins show they happened; merge duplicates; delete done tasks
  after about a week.
- A Human-owned task is settled only two ways: a human
  explicitly settled the ask, or about three weeks unanswered → quiet
  cancel. When in doubt, leave it.
- The goal board — a ledger section listing the open goals you're
  involved in: your teams', plus any you own or contribute to. Per
  goal: title, gist, owner, your involvement (owner, contributor, or
  just your team), due date, confidence, latest movement with date.
  When its header date is over a week old or there is no board,
  re-pull those goals and true up every line, dropping closed and
  archived goals. Only this full re-pull moves the header date.
- Keep every knowledge file small and factual.
