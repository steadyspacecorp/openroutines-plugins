# OpenRoutines plugins

Reusable capability bundles for [OpenRoutines](https://github.com/steadyspacecorp/openroutines).

Each top-level directory is one plugin containing a `PLUGIN.md` manifest and its grouped routines, skills, and optional knowledge-ledger stubs. Plugins are executable supply-chain input: review the declared authority, routine prompts, and skill files before installing or updating one.

## Available plugins

- [`agentmail-report`](agentmail-report/) -- reports the agent's intentions and progress to an email address as a teammate-style update, sent from the agent's own [AgentMail](https://agentmail.to) inbox.
- [`discord-report`](discord-report/) -- reports the agent's intentions and progress to a Discord channel as a teammate-style update, via a channel webhook.
- [`github-docs`](github-docs/) -- watches documentation repositories for changes, keeps a running account of what changed and what it affects, and flags docs filed in the wrong folder or carrying the wrong tags -- the base of a knowledgebot.
- [`github-issues`](github-issues/) -- conservatively triages a GitHub repository's issues and prints a weekly issue-health digest.
- [`slack-report`](slack-report/) -- reports the agent's intentions and progress to a Slack channel as a teammate-style update, via a minimal Slack app's bot token.
- [`steady`](steady/) -- connects an agent to [Steady](https://runsteady.com) like a teammate: a daily check-in filed from the knowledge feed, and prompt replies to comments addressed to the agent.
- [`telegram-report`](telegram-report/) -- reports the agent's intentions and progress to a Telegram chat as a teammate-style update, and answers replies to the report, via a BotFather bot token.

## Install

```bash
openroutines plugin add steadyspacecorp/openroutines-plugins --path github-issues
```

Installation shows the plugin's grant summary, vendors it under `.openroutines/plugins/github-issues/` with exact source provenance, and forces its routines inactive. Follow the plugin's setup instructions, review the diff, run `openroutines check`, and activate only the routines you want.

Installed plugins can later be inspected and updated from inside the agent:

```bash
openroutines plugin list
openroutines plugin update github-issues
```

Updates validate and summarize the new upstream version before making changes, then three-way merge it with local edits. Newly added routines remain inactive, and plugin updates never overwrite live knowledge.

See [AGENTS.md](AGENTS.md) for the plugin format, security boundaries, and validation workflow.
