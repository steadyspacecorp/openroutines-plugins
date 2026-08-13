---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
# Manual-only: ships inactive and stays inactive; the schedule exists to
# satisfy check and never fires while the routine is parked. Run it with
# `openroutines routines run telegram-verify`.
schedule: "0 12 * * 1-5"
active: false
timeout: 5m
teamwork: off
skills: [telegram-send]
credentials: [telegram_bot_token]
---

Verify the Telegram wiring end to end, with real credentials. This
routine sends exactly one clearly labeled test message and nothing
else; it does not read knowledge, consume anything, or retry beyond the
skill's rules.

## Execution discipline

Do exactly the verification below: load `telegram-send`, check the
token and chat, then, only when both succeed, send one test message.
Do not inspect or print environment variables, schedules, routines,
knowledge, or unrelated files; never echo the token or any URL
containing it. Do not narrate between actions. Keep the final result to
one short sentence naming the bot, the chat, and the delivered message
id, or the single failure and its remedy.

1. **Check the token.** GET
   `https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getMe` and report
   the bot's username. A `401` means the credential itself: re-set
   telegram_bot_token from @BotFather. Say so and stop -- do not
   attempt the send.

   The bot's display name is what the chat sees, and it should be your
   own name -- the agent name your standing instruction opens with,
   exactly as written there. GET `.../getMyName`; if it differs, POST
   `.../setMyName` with body `{"name": "<your name>"}`. A 200 with
   `"ok": true` confirms it; say what changed. Telegram rate-limits
   name changes hard -- on `429`, report that the name stays as-is for
   now, and continue.

2. **Check the chat.** GET `.../getChat?chat_id=$TELEGRAM_CHAT_ID` and
   report the chat's title (or the person's name). A `400` with "chat
   not found" means the telegram_chat_id variable is wrong or the bot
   was never added; a `403` means it was removed. Say which and stop.

   If telegram-inbox is active in this agent, also GET
   `.../getWebhookInfo` -- the reply loop reads `getUpdates`, and a
   webhook set on the bot makes every read return `409`. A non-empty
   webhook URL may belong to something else using this bot: report it
   and let a human clear it with `.../deleteWebhook`; do not delete it
   yourself. Report the result in the same final sentence.

3. **Send the test message.** Via the telegram-send skill, to
   `$TELEGRAM_CHAT_ID`. Introduce yourself by name -- you know who you
   are and what your job is from your standing context -- warm and
   brief, in the spirit of: "👋 Hi, I'm <your name>! Quick check that I
   can message you here. If you can see this, we're all set -- nothing
   for you to do." Append the run id in parentheses for diagnostics.
   Plain text is fine; the skill's one-chat conduct still applies.

4. **Report the outcome precisely.** HTTP 200 with `"ok": true` and a
   `result.message_id` means the wiring works end to end: say so, and
   quote the id. Otherwise diagnose per the skill's error table (401 ->
   the token; 400 chat not found -> the chat id; 403 -> the bot was
   removed or blocked; 429 -> quote `retry_after`).
