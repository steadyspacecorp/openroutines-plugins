---
schedule: "45 8-17/3 * * 1-5"
trigger:
  poll: https://service.steady.space/api/v2/digest?category=comment&per_page=1
  select: /0/id
  interval: 5m
  credential: steady_token
timeout: 10m
events: false
skills: [steady-api]
credentials: [steady_token]
---

Watch Steady and keep memory current. Replying to comments is your only
Steady write — everything else the agent reports goes through
steady-check-in. Most runs find nothing new: end quickly.

## 1. Gate on the digest

One cheap call decides whether this run has work: fetch the digest
since a day before your newest ledger entry (no ledger yet → 3 days
back), and sort its entries:

- A comment entry that is not yours, not in your ledger, and addressed
  to you (on your resources, or @mentioning you on anyone's) → step 2
  work.
- A teammate's check-in entry you haven't seen → step 3 material.
- Nothing in either bucket → stop. Most runs end here, a few calls in.

Fetch a resource's full thread only when you are about to reply on it —
the thread is reply context and the double-reply check, never a
standing sweep. Fetch a teammate's check-in body only when the digest
surfaced it. Keep no cursor; re-reading the digest is safe — the ledger
is what makes replays harmless.

## 2. Answer comments

Your ledger (memory/ledgers/steady-inbox.md) records the ids of comments
you already replied to — that record is what stops double replies. A
comment is handled when its id is in the ledger, or a comment of yours on
the same resource has a later created_at (comments arrive newest-first —
compare timestamps, never list position). **Every unhandled
comment gets a reply, no exceptions**; skipping one means every future run
re-examines it. After a reply posts successfully, add the replied-to
comment's id to the ledger; a failed post gets no entry, so the next run
retries it. Prune entries once their comment ages out of the collection
window. In scope: every comment addressed to you — on your own check-ins
and goal updates, or mentioning you on anyone's (reply in that resource's
thread). Teammate content that doesn't address you still belongs to step 3
only. For each in-scope, unhandled comment by someone else:

- **Action request** → add an Agent-owned task to memory/tasks.md (stable
  id; source: requester + the commented resource); reply with a brief "on it" that
  names when — the next run of the routine whose domain covers the item
  (./schedule.md shows when each routine next fires), e.g. "On it — I'll
  draft that in Tuesday's roadmap pass."
- **Question or feedback** → answer it in-thread.
- **Answer to a Human-owned task in memory/tasks.md** → resolve the task in
  place: transfer it to Agent-owned if the ask became agent work, or cancel
  it if declined; reply with a brief acknowledgement.
- **Anything else** (FYI, status update, someone claiming work) → reply with
  a brief acknowledgement; if it settles or claims something, step 3 applies
  too.

Post your reply as a comment on the same resource the comment is on.
Casual teammate voice: short and warm, not corporate. One reply may answer several pending comments on a resource. A
failed post just leaves the comment unanswered for the next run.

## 3. Update memory from teammates' work

From the check-ins and goal updates you pulled:

- A human explicitly settled a Human-owned task (a comment, check-in, or
  goal update addressing the ask itself — agent work near the topic does
  not count) → resolve it as in step 2.
- A human's work covers an open Agent-owned task in memory/tasks.md → mark
  it done ([x]), crediting them.
- Refresh memory/context.md with in-flight or claimed work that overlaps
  the agent's own lanes — the domains its routines cover: who, what, firm
  or tentative, date. Leave other lanes out; drop entries older than
  about a week.

Beyond these uses, don't act on teammates' content.

## 4. Groom

- Mark open Agent-owned tasks in memory/tasks.md done when memory/events.md
  or the agent's check-ins show they happened; merge duplicates; delete done
  tasks after about a week.
- A Human-owned task leaves memory/tasks.md only two ways: a human
  explicitly settled the ask, or it has gone unanswered for about three
  weeks and is quietly cancelled. Never remove one for any other reason;
  when in doubt, leave it.
- Keep every memory file small and factual.
