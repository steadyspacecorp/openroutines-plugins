---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
# Manual-only: ships inactive and stays inactive; the schedule exists to
# satisfy check and never fires while the routine is parked. Run it with
# `openroutines routines run agentmail-verify`.
schedule: "0 12 * * 1-5"
active: false
timeout: 5m
teamwork: off
skills: [agentmail-send]
credentials: [agentmail_api_key]
---

Verify the AgentMail wiring end to end, with real credentials. This
routine sends exactly one clearly labeled test email and nothing else;
it does not read knowledge, consume anything, or retry beyond the
skill's rules.

## Execution discipline

Do exactly the verification below: load `agentmail-send`, GET the
inbox, then, only when that succeeds, send one test email. Do not
inspect or print environment variables, schedules, routines, knowledge,
or unrelated files; never echo the API key. Do not narrate between
actions. Keep the final result to one short sentence naming the sending
address, the recipient, and the delivered message id, or the single
failure and its remedy.

1. **Check the key and inbox without sending.** GET
   `https://api.agentmail.to/v0/inboxes/$AGENTMAIL_INBOX_ID` with the
   bearer key returns the inbox object. Report its `email` and
   `display_name` -- that confirms the key works and which sender the
   report will arrive as. A 401 or 403 here means the credential
   itself: re-set agentmail_api_key (and confirm the key carries
   `inbox_read`). A 404 means `$AGENTMAIL_INBOX_ID` is wrong or the key
   is scoped to a different inbox. Say which and stop -- do not attempt
   the send.

2. **Send the test email.** Via the agentmail-send skill, to
   `$AGENTMAIL_REPORT_TO`. Introduce yourself by name -- you know who
   you are and what your job is from your standing context -- warm and
   brief, in the spirit of: subject "👋 Hi from <your name> -- testing
   the wiring, nothing to do", body "Quick check that I can email you
   here. If you can see this, we're all set -- nothing for you to do."
   Append the run id in parentheses for diagnostics. Plain text alone
   is fine; skip the HTML body. The skill's one-recipient conduct still
   applies.

3. **Report the outcome precisely.** HTTP 200 with a `message_id` means
   the wiring works end to end: say so, and quote the id. Otherwise
   diagnose per the skill's error table (403 -> the key was revoked or
   lacks `message_send`; 404 -> the inbox id or key scope; 400 -> quote
   the validation error and its `fix` field).
