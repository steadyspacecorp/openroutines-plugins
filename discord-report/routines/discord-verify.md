---
# Manual-only: ships inactive and stays inactive; the schedule exists to
# satisfy check and never fires while the routine is parked. Run it with
# `openroutines routines run discord-verify`.
schedule: "0 12 * * 1-5"
active: false
timeout: 5m
events: false
skills: [discord-post]
credentials: [discord_webhook_url]
---

Verify the Discord wiring end to end, with real credentials. This
routine posts exactly one clearly labeled test message and nothing else;
it does not read memory, consume anything, or retry beyond the skill's
rules.

1. **Check the webhook without posting.** A bare GET on
   `$DISCORD_WEBHOOK_URL` returns the webhook object. Report its `name`
   and `channel_id` -- that confirms the URL is live and where it
   points. A 401 or 404 here means the URL is wrong or the webhook was
   deleted: say which and stop -- do not attempt the post.

2. **Post the test message.** Via the discord-post skill, with
   `?wait=true`: content "Wiring check from the agent (run
   $OPENROUTINES_RUN_ID) -- safe to ignore." No embed needed;
   `allowed_mentions` rules still apply.

3. **Report the outcome precisely.** HTTP 200 with a message `id` means
   the wiring works end to end: say so, and quote the id. Otherwise
   diagnose per the skill's error table (429 -> honor retry_after once;
   400 -> quote the validation error).
