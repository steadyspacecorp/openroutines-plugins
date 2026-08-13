---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
# Manual-only: ships inactive and stays inactive; the schedule exists to
# satisfy check and never fires while the routine is parked. Run it with
# `openroutines routines run slack-verify`.
schedule: "0 12 * * 1-5"
active: false
timeout: 5m
teamwork: off
skills: [slack-post]
credentials: [slack_bot_token]
---

Verify the Slack wiring end to end, with real credentials. This routine
posts exactly one clearly labeled test message and nothing else; it does
not read knowledge, consume anything, or retry beyond the skill's rules.

1. **Check the token.** `curl -sS -H "Authorization: Bearer
   $SLACK_BOT_TOKEN" https://slack.com/api/auth.test` and report the
   workspace and bot user it resolves to. `ok: false` here means the
   credential itself: report the `error` field and stop -- do not
   attempt the post.

   If slack-inbox is active in this agent, also GET
   `https://slack.com/api/conversations.history?channel=$SLACK_CHANNEL&limit=1`
   with the bearer token -- that is the reply trigger's poll path, and
   it needs the app's `channels:history` scope. `ok: false` with
   `missing_scope` means the app is report-only: either re-install it
   at the reply tier per PLUGIN.md or deactivate slack-inbox. Report
   the result in the same final sentence; do not print message
   contents.

2. **Post the test message.** Via the slack-post skill, to
   `$SLACK_CHANNEL`. Introduce yourself by name -- you know who you are
   and what your job is from your standing context -- warm and brief,
   in the spirit of: "👋 Hi, I'm <your name>! Quick check that I can post
   here. If you can see this, we're all set -- nothing for you to do."
   Append the run id in parentheses for diagnostics. Plain text is
   fine; the skill's no-pings conduct still applies.

3. **Report the outcome precisely.** `ok: true` with a message `ts`
   means the wiring works end to end: say so, and quote the ts. On
   `ok: false`, name the error and what fixes it: `invalid_auth` or
   `token_revoked` -> re-set the slack_bot_token credential;
   `not_in_channel` -> a person must /invite the bot to the channel;
   `channel_not_found` -> the slack_channel variable holds the wrong ID.
