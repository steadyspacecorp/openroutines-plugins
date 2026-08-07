---
name: slack-report
description: Post the agent's knowledge change feed to a Slack channel as a teammate-style update, via a minimal Slack app's bot token.
credentials:
  slack_bot_token:
    description: The Slack app's bot user OAuth token (xoxb-...), created from the app manifest below with only the chat:write scope
variables:
  slack_channel:
    description: The ID of the channel to post to (C..., from the channel's details pane) -- the bot must be invited to it
---

# slack-report

Points an agent's reporting at a Slack channel. The routine is a knowledge-feed consumer: each workday morning it turns everything the agent recorded since its last report into one short, human update and posts it with `chat.postMessage`.

## What you get

- **slack-report** -- consumes the knowledge change feed and posts a digest: what happened, what's now on someone's plate, what changed in the task list. Nothing new since last time means no post -- the channel never gets a "nothing to report" message.
- **slack-verify** -- a manual-only wiring check: run it after setup and it posts one labeled test message, or tells you exactly which piece (token, invite, channel ID) is wrong. It ships inactive and stays that way; the scheduler never fires it. Run it with `OPENROUTINES_LOG_LEVEL=warn openroutines routines run slack-verify --no-knowledge` (quiet diagnostics, and `--no-knowledge` so a local run leaves the knowledge worktree untouched).
- **slack-post** skill -- how to format and send the message: one plain-text fallback, Block Kit sections, the `ok: true` delivery check, no @channel.

## Why a bot token, not a webhook

Slack marks the standalone incoming-webhooks integration as a legacy custom integration -- "will be deprecated and possibly removed in the future" -- and webhooks inside a modern app mint their URL per channel at install time, so re-pointing one means reinstalling. A bot token from a minimal app is the durable path, and the app manifest below keeps it as narrow as a webhook was: the only scope is `chat:write`, which reaches exactly the channels the bot is explicitly invited to. One invite, one channel.

## Create the Slack app

Go to [api.slack.com/apps](https://api.slack.com/apps) → **Create New App** → **From a manifest**, pick your workspace, and paste:

```yaml
display_information:
  name: OpenRoutines Agent
  description: Posts an autonomous agent's reports to a channel.
  background_color: "#2c2d30"
features:
  bot_user:
    display_name: openroutines-agent
    always_online: false
oauth_config:
  scopes:
    bot:
      - chat:write
settings:
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false
```

Rename the display names to match your agent. Then **Install to Workspace** and copy the **Bot User OAuth Token** (`xoxb-...`) from OAuth & Permissions.

## After installing

1. `openroutines credentials set slack_bot_token` -- the `xoxb-` token from the step above.
2. Invite the bot to the target channel (`/invite @openroutines-agent`) and copy the channel ID from the channel's details pane (starts with `C`).
3. Set the `slack_channel` variable in `openroutines.yml` to that ID.
4. `OPENROUTINES_LOG_LEVEL=warn openroutines routines run slack-verify --no-knowledge` -- posts one labeled test message through the real wiring and diagnoses any failure.
5. Adjust the schedule, then `openroutines check`, review the diff, commit.

Works alongside other consumers: each keeps its own cursor over the same feed, so adding Slack changes nothing about the routines doing the work -- or about any other destination already reporting. Re-pointing the report later is a `slack_channel` edit plus an invite, never a reinstall.
