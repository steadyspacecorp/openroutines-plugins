---
name: discord-report
description: Post the agent's memory change feed to a Discord channel as a teammate-style update, via a channel webhook.
credentials:
  discord_webhook_url:
    description: A Discord channel webhook URL (https://discord.com/api/webhooks/...) -- it both authenticates and addresses the channel, so treat it as a secret
---

# discord-report

Points an agent's reporting at a Discord channel. The routine is a memory-feed consumer: twice a workday it turns everything the agent recorded since its last report into one short, human update and posts it through a channel webhook.

## What you get

- **discord-report** -- consumes the memory change feed and posts a digest: what happened, what's now on someone's plate, what changed in the task list. Nothing new since last time means no post -- the channel never gets a "nothing to report" message.
- **discord-verify** -- a manual-only wiring check: run `openroutines routines run discord-verify` after setup and it confirms the webhook (a bare GET, no post) then sends one labeled test message. It ships inactive and stays that way; the scheduler never fires it.
- **discord-post** skill -- how to format and send the message: content plus one embed, the `?wait=true` delivery check, structural mention suppression, no retries past one.

Unlike Slack, Discord's channel webhooks are a first-class, supported feature, so this plugin gets the narrowest possible design: the URL both authenticates and addresses one channel, post-only. One credential, no variables, no app to create.

## After installing

1. In the target channel: Settings → Integrations → Webhooks → New Webhook. Name it after your agent (that's the poster's display name), then copy the URL.
2. `openroutines credentials set discord_webhook_url`
3. `openroutines routines run discord-verify` -- confirms the webhook and posts one labeled test message through the real wiring.
4. Adjust the schedule, then `openroutines check`, review the diff, commit.

`--no-memory` discards memory settlement only; it does not suppress Discord posts or withhold credentials. `openroutines check` is the non-acting validation path.

Works alongside other consumers: each keeps its own cursor over the same feed, so adding Discord changes nothing about the routines doing the work -- or about any other destination already reporting. To re-point the report, create a webhook in the new channel and re-set the credential.
