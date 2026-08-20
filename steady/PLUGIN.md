---
name: steady
description: Connect an agent to Steady -- a daily check-in on its work, and prompt replies to comments addressed to it.
credentials:
  steady_token:
    description: A Steady personal access token (steady_pat_..., read+write) for the agent's own Steady account
mcp:
  steady:
    description: Steady's MCP server -- the digest, comments, action items, check-ins, and goals
    url: https://app.steady.space/mcp
    credential: steady_token
---

# steady

Connects an agent to [Steady](https://runsteady.com): a daily check-in
on its work, and replies to comments addressed to it.

## What you get

- **steady-check-in** -- files the agent's daily check-in: what it
  worked on, what it plans next, and where it waits on a teammate.
- **steady-inbox** -- the agent's side of the conversation: answers
  comments addressed to it, turns action requests into tracked tasks,
  and keeps its knowledge current with teammates' work and the goals the
  agent is involved in.
- **steady-verify** -- a manual-only wiring check that creates one clearly labeled use activity through Steady's MCP server. It ships inactive and stays that way; the scheduler never fires it. Run it with `OPENROUTINES_LOG_LEVEL=warn openroutines routines run steady-verify` (quiet diagnostics; a manual run discards knowledge changes unless you pass `--write-knowledge`, so it leaves the knowledge worktree untouched).

## After installing

1. `openroutines credentials set steady_token` -- a personal access
   token for the agent's own Steady account (Settings → Agents).
2. Accept the MCP server definition when the install offers it (or paste
   the printed snippet into opencode.json).
3. `OPENROUTINES_LOG_LEVEL=warn openroutines routines run steady-verify` -- creates one labeled use activity through the real MCP wiring so you can confirm it in Steady.
4. Adjust the schedules to your workday; both active routines assume the
   agent's timezone.
5. `openroutines check`, review the diff, commit, and activate.

`steady-verify` performs a real external write. A manual run discards knowledge changes only; it does not suppress that activity or withhold credentials. `openroutines check` is the non-acting validation path.

The check-in is a knowledge-feed consumer: it needs nothing beyond what
your other routines already record. If the agent has no other routines
yet, its check-ins will be quiet -- that's correct, not broken.
