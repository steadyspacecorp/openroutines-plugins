---
# Manual-only: ships inactive and stays inactive; the schedule exists to
# satisfy check and never fires while the routine is parked. Run it with
# `openroutines routines run slack-verify`.
schedule: "0 12 * * 1-5"
active: false
timeout: 5m
events: false
skills: [slack-post]
credentials: [slack_bot_token]
---

Verify the Slack wiring end to end, with real credentials. This routine
posts exactly one clearly labeled test message and nothing else; it does
not read memory, consume anything, or retry beyond the skill's rules.

1. **Check the token.** `curl -sS -H "Authorization: Bearer
   $SLACK_BOT_TOKEN" https://slack.com/api/auth.test` and report the
   workspace and bot user it resolves to. `ok: false` here means the
   credential itself: report the `error` field and stop -- do not
   attempt the post.

2. **Post the test message.** Via the slack-post skill, to
   `$SLACK_CHANNEL`: "Wiring check from the agent (run
   $OPENROUTINES_RUN_ID) -- safe to ignore." No embeds or sections
   needed; `allowed_mentions` rules still apply.

3. **Report the outcome precisely.** `ok: true` with a message `ts`
   means the wiring works end to end: say so, and quote the ts. On
   `ok: false`, name the error and what fixes it: `invalid_auth` or
   `token_revoked` -> re-set the slack_bot_token credential;
   `not_in_channel` -> a person must /invite the bot to the channel;
   `channel_not_found` -> the slack_channel variable holds the wrong ID.
