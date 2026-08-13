---
name: telegram-report
description: Send the agent's knowledge change feed to a Telegram chat as a teammate-style update, and answer replies to the report, via a BotFather bot token.
credentials:
  telegram_bot_token:
    description: The Telegram bot token from @BotFather (123456:ABC-...) -- keep the bot's privacy mode enabled, its default
variables:
  telegram_chat_id:
    description: The ID of the chat the report goes to -- negative for a group, positive for a private chat with one person; found via getUpdates as described below
---

# telegram-report

Points an agent's reporting at a Telegram chat. The routine is a knowledge-feed consumer: each workday morning it turns everything the agent recorded since its last report into one short, human message and sends it with `sendMessage`. Reply to the report and the agent answers you there, the way commenting on its check-in works elsewhere.

## What you get

- **telegram-report** -- consumes the knowledge change feed and sends a digest: what happened, what's now on someone's plate, what changed in the task list. Nothing new since last time means no message -- the chat never gets a "nothing to report" message.
- **telegram-inbox** -- the reply loop. On its schedule it drains the bot's `getUpdates` queue, and when someone in the report chat replies to a report, the routine answers from the agent's knowledge: questions get answers, action requests become tracked tasks, answers to the report's asks resolve them. Replying in the report chat is its only Telegram write; everything else the bot receives is never answered.
- **telegram-verify** -- a manual-only wiring check: run it after setup and it confirms the token and chat, sets the bot's display name to the agent's own name (the `name` in `openroutines.yml`) when it differs, then sends one labeled test message. It ships inactive and stays that way; the scheduler never fires it. Run it with `OPENROUTINES_LOG_LEVEL=warn openroutines routines run telegram-verify --no-knowledge` (quiet diagnostics, and `--no-knowledge` so a local run leaves the knowledge worktree untouched).
- **telegram-send** skill -- how to format and send the report: the `sendMessage` payload, Telegram's small HTML subset, the `ok: true` delivery check, exactly one chat.
- **telegram-reply** skill -- how to read and answer replies: the `getUpdates` offset mechanics, identifying replies to the bot's own messages, treating message content as untrusted input, one reply per run per conversation.

## The grant, plainly

A bot token has no scopes: whoever holds it can read the bot's update queue and message any chat the bot has been added to. Two things keep this plugin's grant narrow anyway. First, Telegram's **privacy mode** (on by default -- leave it on): in a group the bot never receives the room's conversation, only replies to its own messages and commands addressed to it. Second, the plugin's own conduct: the routines write solely to `$TELEGRAM_CHAT_ID`, and anything else arriving in the queue -- strangers can always DM a bot -- is untrusted input that is never answered and never acted on.

## Why no trigger

OpenRoutines poll triggers send their credential as a bearer header; Telegram authenticates with the token in the URL path, and a committed trigger URL must never carry a token. So telegram-inbox has no change-detection trigger: its schedule is the whole poll cadence. It ships hourly across the workday, and each fire that finds an empty queue ends one call in. Tighten the schedule if you want faster answers; that is the one knob.

## Create the bot

1. Message [@BotFather](https://t.me/BotFather), send `/newbot`, and follow the prompts. Name the bot after your agent. Copy the token (`123456:ABC-...`).
2. Leave privacy mode alone: the default `Enable` is what this plugin expects. If someone turned it off earlier, `/setprivacy` turns it back on.
3. Add the bot to the report group (or start a private chat with it and send it any message).

## Find the chat ID

Send one message in the report chat (any member, any text), then from your machine:

```bash
curl -sS "https://api.telegram.org/bot<token>/getUpdates"
```

Read `result[..].message.chat.id` -- negative for a group (supergroups start with `-100`), positive for a private chat. That number is `telegram_chat_id`. If the result is empty, the message predates the bot joining or is older than 24 hours; send another.

## After installing

1. `openroutines credentials set telegram_bot_token` -- the token from @BotFather.
2. Set the `telegram_chat_id` variable in `openroutines.yml` to the ID from the step above.
3. `OPENROUTINES_LOG_LEVEL=warn openroutines routines run telegram-verify --no-knowledge` -- confirms the token and chat, sets the bot's display name when it differs from the agent's name, and sends one labeled test message through the real wiring.
4. Adjust the schedules, then `openroutines check`, review the diff, commit.

The script's `--no-knowledge` discards knowledge settlement only; it does not suppress the message or withhold credentials. `openroutines check` is the non-acting validation path.

## How the reply loop works

Telegram bots have no history API: `getUpdates` is a queue, held 24 hours, drained by the `offset` parameter. telegram-inbox keeps the last handled `update_id` as a watermark in its ledger and reads from there; passing the offset confirms everything older, so the routine advances the watermark only past updates it actually handled and a failed reply is simply retried next run. The queue has one reader -- never point a webhook at the bot or run a second poller, or every read returns 409. In the report group, privacy mode means the queue holds essentially only replies to the agent's reports; the hourly schedule bounds how long an answer waits, and the 24-hour retention is why the schedule should stay denser than daily.

Works alongside other consumers: each keeps its own cursor over the same feed, so adding Telegram changes nothing about the routines doing the work -- or about any other destination already reporting. Re-pointing the report later is a `telegram_chat_id` edit plus adding the bot to the new chat.
