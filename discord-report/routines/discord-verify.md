---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
# Manual-only: ships inactive and stays inactive; the schedule exists to
# satisfy check and never fires while the routine is parked. Run it with
# `openroutines routines run discord-verify`.
schedule: "0 12 * * 1-5"
active: false
timeout: 5m
teamwork: off
skills: [discord-post]
credentials: [discord_webhook_url]
---

Verify the Discord wiring end to end, with real credentials. This
routine posts exactly one clearly labeled test message and nothing else;
it does not read memory, consume anything, or retry beyond the skill's
rules.

## Execution discipline

Do exactly the verification below: load `discord-post`, GET the webhook,
then, only when that succeeds, POST one test message. Do not inspect or
print environment variables, schedules, routines, memory, or unrelated
files; never echo the webhook URL. Do not narrate between actions. Keep
the final result to one short sentence naming the webhook, channel id,
and delivered message id, or the single failure and its remedy.

1. **Check the webhook without posting.** A bare GET on
   `$DISCORD_WEBHOOK_URL` returns the webhook object. Report its `name`
   and `channel_id` -- that confirms the URL is live and where it
   points. A 401 or 404 here means the URL is wrong or the webhook was
   deleted: say which and stop -- do not attempt the post.

2. **Post the test message.** Via the discord-post skill, with
   `?wait=true`. Introduce yourself by name -- you know who you are
   and what your job is from your standing context -- warm and brief,
   in the spirit of: "👋 Hi, I'm <your name>! Quick check that I can post
   here. If you can see this, we're all set -- nothing for you to do."
   Append the run id in parentheses for diagnostics. No embed needed;
   the `allowed_mentions` suppression still applies.

3. **Report the outcome precisely.** HTTP 200 with a message `id` means
   the wiring works end to end: say so, and quote the id. Otherwise
   diagnose per the skill's error table (429 -> honor retry_after once;
   400 -> quote the validation error).
