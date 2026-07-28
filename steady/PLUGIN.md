---
name: steady
description: Connect an agent to Steady like a teammate -- a daily check-in filed from the memory feed, and prompt replies to comments addressed to the agent.
credentials:
  steady_token:
    description: A Steady personal access token (steady_pat_..., read+write) for the agent's own Steady account
mcp:
  steady:
    description: Steady's MCP server -- action items, the check-in form, and submission
    url: https://app.steady.space/mcp
    credential: steady_token
---

# steady

Connects an agent to [Steady](https://runsteady.com) the way a teammate is
connected: a daily check-in composed from the agent's memory change feed,
and replies to comments addressed to the agent.

## What you get

- **steady-check-in** -- weekday mornings, consumes the memory change
  feed and files one check-in covering everything since the last one:
  previous work rewritten from events, intentions transcribed from the
  injected schedule, blockers from human-owned asks. Talks to Steady
  through the MCP server.
- **steady-inbox** -- every few hours on workdays (with a poll trigger
  for low-latency replies), answers comments on the agent's check-ins
  and goal updates, turns action requests into memory tasks, and grooms
  the task list. Most runs find nothing and end quickly. Replies post
  through the REST API via the bundled skill -- Steady's MCP server has
  no comment tools yet; when it grows them, this routine switches with a
  frontmatter-only edit.
- **steady-api** skill -- the API reference the inbox works from.

## After installing

1. `openroutines credentials set steady_token` -- a personal access
   token for the agent's own Steady account (Settings → Integrations).
2. Accept the MCP server definition when the install offers it (or paste
   the printed snippet into opencode.json).
3. Adjust the schedules to your workday; both routines assume the
   agent's timezone.
4. `openroutines check`, review the diff, commit, and activate.

The check-in is a memory-feed consumer: it needs nothing beyond what
your other routines already record. If the agent has no other routines
yet, its check-ins will be quiet -- that's correct, not broken.
