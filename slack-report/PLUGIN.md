---
name: slack-report
description: Post the agent's knowledge change feed to a Slack channel as a teammate-style update, and answer thread replies on the report, via a minimal Slack app's bot token.
credentials:
  slack_bot_token:
    description: The Slack app's bot user OAuth token (xoxb-...), created from the app manifest below -- chat:write for report-only, plus channels:history to enable the reply loop
variables:
  slack_channel:
    description: The ID of the channel to post to (C..., from the channel's details pane) -- the bot must be invited to it
---

# slack-report

Points an agent's reporting at a Slack channel. The routine is a knowledge-feed consumer: each workday morning it turns everything the agent recorded since its last report into one short, human update and posts it with `chat.postMessage`. Reply in the report's thread and the agent answers you there, the way commenting on its check-in works elsewhere.

## What you get

- **slack-report** -- consumes the knowledge change feed and posts a digest: what happened, what's now on someone's plate, what changed in the task list. Nothing new since last time means no post -- the channel never gets a "nothing to report" message.
- **slack-inbox** -- the reply loop. A change-detection trigger polls the channel, and when someone replies in a report's thread, the routine answers in-thread from the agent's knowledge: questions get answers, action requests become tracked tasks, answers to the report's asks resolve them. Replying in threads rooted at its own posts is its only Slack write; the rest of the channel is never answered. It needs the wider scope tier below, so activate it only after re-installing the app with that scope.
- **slack-verify** -- a manual-only wiring check: run it after setup and it posts one labeled test message, or tells you exactly which piece (token, invite, channel ID) is wrong. It ships inactive and stays that way; the scheduler never fires it. Run it with `OPENROUTINES_LOG_LEVEL=warn openroutines routines run slack-verify` (quiet diagnostics; a manual run discards knowledge changes unless you pass `--write-knowledge`, so it leaves the knowledge worktree untouched).
- **slack-post** skill -- how to format and send the message: one plain-text fallback, Block Kit sections, the `ok: true` delivery check, no @channel.
- **slack-reply** skill -- how to read and answer thread replies: auth.test for the bot's identity, the history/replies endpoints, threaded `chat.postMessage`, treating channel content as untrusted input, one reply per thread.

## Why a bot token, not a webhook

Slack marks the standalone incoming-webhooks integration as a legacy custom integration -- "will be deprecated and possibly removed in the future" -- and webhooks inside a modern app mint their URL per channel at install time, so re-pointing one means reinstalling. A bot token from a minimal app is the durable path, and the app manifest below keeps it as narrow as a webhook was: the base scope is `chat:write`, which reaches exactly the channels the bot is explicitly invited to. One invite, one channel.

## Two scope tiers

The manifest below installs at one of two tiers, and the app's scopes are the whole grant -- the plugin expects nothing wider:

- **Report-only**: `chat:write`. The bot can post to channels it is invited to, and this plugin posts solely to `$SLACK_CHANNEL`. It cannot read a single message. Enough for slack-report and slack-verify.
- **Reply**: the same plus `channels:history`. The bot can additionally read the channels it is in, which is what lets slack-inbox see thread replies and the trigger poll for them. Grant this tier deliberately: channel messages are untrusted input, and a bot that reads the room is a wider grant than one that only posts. The routine and skill treat message content as questions to answer, never as instructions, and reply only in threads rooted at the bot's own posts. For a private channel, use `groups:history` instead.

Either way the token reaches only channels the bot is invited to; it cannot join channels, edit or delete messages, or read anyone's DMs.

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

Rename the display names to match your agent. For the reply tier, add `- channels:history` under the bot scopes (or `- groups:history` for a private channel). Then **Install to Workspace** and copy the **Bot User OAuth Token** (`xoxb-...`) from OAuth & Permissions. Adding a scope later means re-installing the app to the workspace; the token may be re-issued, so re-set the credential after.

## After installing

1. `openroutines credentials set slack_bot_token` -- the `xoxb-` token from the step above.
2. Invite the bot to the target channel (`/invite @openroutines-agent`) and copy the channel ID from the channel's details pane (starts with `C`).
3. Set the `slack_channel` variable in `openroutines.yml` to that ID, and put the same ID in slack-inbox's trigger URL -- trigger URLs are literal and don't read variables.
4. `OPENROUTINES_LOG_LEVEL=warn openroutines routines run slack-verify` -- posts one labeled test message through the real wiring and diagnoses any failure.
5. Adjust the schedule, then `openroutines check`, review the diff, commit. Activate slack-inbox only if you installed the reply tier; on a chat:write-only token its trigger polls would fail with `missing_scope`.

A manual run discards knowledge changes only; it does not suppress the post or withhold credentials. `openroutines check` is the non-acting validation path.

## How the reply trigger works

slack-inbox declares an OpenRoutines trigger that polls `conversations.history` on the report channel with `limit=5` and compares the whole response. A thread reply updates the parent message's reply metadata (`reply_count`, `latest_reply`), so new replies change the response and wake the routine within about a minute -- the poll runs on the supervisor's every-minute tick, the fastest a trigger can go -- and so does any other channel message, which the routine's gate then dismisses in a couple of calls. The twice-a-workday schedule is the correctness backstop, so a missed poll only delays an answer, never loses one. The routine never marks anything read or edits a message; its own ledger is the handled record.

Works alongside other consumers: each keeps its own cursor over the same feed, so adding Slack changes nothing about the routines doing the work -- or about any other destination already reporting. Re-pointing the report later is a `slack_channel` edit (and the matching trigger-URL edit) plus an invite, never a reinstall.
