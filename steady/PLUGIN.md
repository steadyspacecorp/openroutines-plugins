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
  and keeps its memory current with what teammates are working on.

## After installing

1. `openroutines credentials set steady_token` -- a personal access
   token for the agent's own Steady account (Settings → Agents).
2. Accept the MCP server definition when the install offers it (or paste
   the printed snippet into opencode.json).
3. Adjust the schedules to your workday; both routines assume the
   agent's timezone.
4. `openroutines check`, review the diff, commit, and activate.

The check-in is a memory-feed consumer: it needs nothing beyond what
your other routines already record. If the agent has no other routines
yet, its check-ins will be quiet -- that's correct, not broken.
