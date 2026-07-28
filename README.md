# OpenRoutines plugins

Reusable capability bundles for [OpenRoutines](https://github.com/steadyspacecorp/openroutines).

Each top-level directory is one plugin containing a `PLUGIN.md` manifest and its grouped routines, skills, and optional memory-ledger stubs. Plugins are executable supply-chain input: review the declared authority, routine prompts, and skill files before installing or updating one.

## Available plugins

- [`github-issues`](github-issues/) -- conservatively triages a GitHub repository's issues and prints a weekly issue-health digest.
- [`steady`](steady/) -- connects an agent to [Steady](https://runsteady.com) like a teammate: a daily check-in filed from the memory feed, and prompt replies to comments addressed to the agent.

## Install

```bash
openroutines plugin add steadyspacecorp/openroutines-plugins --path github-issues
```

Installation shows the plugin's grant summary, vendors it under `plugins/github-issues/` with exact source provenance, and forces its routines inactive. Follow the plugin's setup instructions, review the diff, run `openroutines check`, and activate only the routines you want.

Installed plugins can later be inspected and updated from inside the agent:

```bash
openroutines plugin list
openroutines plugin update github-issues
```

Updates validate and summarize the new upstream version before making changes, then three-way merge it with local edits. Newly added routines remain inactive, and plugin updates never overwrite live memory.

See [AGENTS.md](AGENTS.md) for the plugin format, security boundaries, and validation workflow.
