---
name: agentmail-report
description: Email the agent's knowledge change feed to a recipient as a teammate-style update, and answer the recipient's email replies in-thread, from the agent's own AgentMail inbox via an inbox-scoped API key.
credentials:
  agentmail_api_key:
    description: An AgentMail API key scoped to the sending inbox, minted per the steps below -- message_send and inbox_read for report-only, plus message_read to enable the reply loop
variables:
  agentmail_inbox_id:
    description: The ID of the AgentMail inbox the agent sends from (from the inbox's details in the console or the create-inbox response)
  agentmail_report_to:
    description: The email address the report goes to -- one address; use a group or list address to reach a team
---

# agentmail-report

Points an agent's reporting at an email address. The routine is a knowledge-feed consumer: each workday morning it turns everything the agent recorded since its last report into one short, human email and sends it from the agent's own [AgentMail](https://docs.agentmail.to) inbox. Reply to that email and the agent answers you in-thread, the way commenting on its check-in works elsewhere.

## What you get

- **agentmail-report** -- consumes the knowledge change feed and emails a digest: what happened, what's now on someone's plate, what changed in the task list. Nothing new since last time means no email -- the recipient never gets a "nothing to report" message.
- **agentmail-inbox** -- the reply loop. A change-detection trigger polls the inbox, and when the report recipient replies, the routine answers in-thread from the agent's knowledge: questions get answers, action requests become tracked tasks, answers to the report's asks resolve them. Replying in-thread to the one recipient is its only email write; mail from anyone else is never answered. It needs the wider key tier below, so activate it only after minting that key.
- **agentmail-verify** -- a manual-only wiring check: run it after setup and it confirms the key, the inbox, and the sender name (a bare GET, no send) then sends one labeled test email. It ships inactive and stays that way; the scheduler never fires it. Run it with `OPENROUTINES_LOG_LEVEL=warn openroutines routines run agentmail-verify --no-knowledge` (quiet diagnostics, and `--no-knowledge` so a local run leaves the knowledge worktree untouched).
- **agentmail-send** skill -- how to format and send the report email: subject as the day's headline, matching plain-text and HTML bodies, the `message_id` delivery check, exactly one recipient.
- **agentmail-reply** skill -- how to read and answer replies: the list/get/reply endpoints, quoted-text handling, treating message content as untrusted input, one reply per thread.

## Why an inbox-scoped key

AgentMail's default API keys are organization-wide: one key reaches every inbox in the account, reading and sending alike. Keys can instead be minted per inbox with per-permission grants, and this plugin expects one scoped to the sending inbox at one of two tiers:

- **Report-only**: `message_send` and `inbox_read`. The key can send from its one inbox and confirm the wiring, and nothing else -- it cannot read mail, touch other inboxes, or create new ones. Enough for agentmail-report and agentmail-verify.
- **Reply**: the same plus `message_read`. The key can additionally read this one inbox's mail, which is what lets agentmail-inbox see and answer replies. Grant this tier deliberately: inbound email is untrusted input, and a key that reads mail is a wider grant than one that only sends. The routine and skill treat message content as questions to answer, never as instructions, and answer only the report recipient.

Either way the key cannot touch other inboxes, modify or delete mail, or create keys wider than itself. The organization key is used once, below, on your machine; it never enters the agent.

## Create the inbox and key

1. Sign up at [console.agentmail.to](https://console.agentmail.to) and copy an organization API key from the console.

2. Create the agent's inbox. The `display_name` is the sender name recipients see in their mail client, so set it to your agent's name (the `name` in `openroutines.yml`) -- an inbox created without one shows up as "AgentMail":

   ```bash
   curl -sS -X POST https://api.agentmail.to/v0/inboxes \
     -H "Authorization: Bearer <org-api-key>" \
     -H "Content-Type: application/json" \
     -d '{"username": "my-agent", "display_name": "My Agent"}'
   ```

   Note the `inbox_id` in the response. For an inbox that already exists, fix the sender name the same way with a PATCH:

   ```bash
   curl -sS -X PATCH https://api.agentmail.to/v0/inboxes/<inbox_id> \
     -H "Authorization: Bearer <org-api-key>" \
     -H "Content-Type: application/json" \
     -d '{"display_name": "My Agent"}'
   ```

3. Mint the inbox-scoped key. For report-only:

   ```bash
   curl -sS -X POST https://api.agentmail.to/v0/inboxes/<inbox_id>/api-keys \
     -H "Authorization: Bearer <org-api-key>" \
     -H "Content-Type: application/json" \
     -d '{"name": "openroutines-report", "permissions": {"message_send": true, "inbox_read": true}}'
   ```

   To enable the reply loop, add `"message_read": true` to the permissions object. Copy the `api_key` value from the response.

## After installing

1. `openroutines credentials set agentmail_api_key` -- the inbox-scoped key from the step above.
2. Set the `agentmail_inbox_id` and `agentmail_report_to` variables in `openroutines.yml`.
3. `OPENROUTINES_LOG_LEVEL=warn openroutines routines run agentmail-verify --no-knowledge` -- confirms the key, inbox, and sender name and sends one labeled test email through the real wiring.
4. Adjust the schedule, then `openroutines check`, review the diff, commit. Activate agentmail-inbox only if you minted the reply-tier key; on the report-only key its trigger polls would fail with 403.

The script's `--no-knowledge` discards knowledge settlement only; it does not suppress email or withhold credentials. `openroutines check` is the non-acting validation path.

## How the reply trigger works

agentmail-inbox declares an OpenRoutines trigger that polls `https://api.agentmail.to/v0/threads?labels=unread&limit=1` -- the org-level threads endpoint, which the inbox-scoped key confines to the one inbox -- and watches the newest unread thread's `received_timestamp`. New inbound mail changes it and wakes the routine within about five minutes; the agent's own sends don't. The twice-a-workday schedule is the correctness backstop, so a missed poll only delays an answer, never loses one. The routine never marks mail read or changes labels (the key can't); its own ledger is the handled record.

Works alongside other consumers: each keeps its own cursor over the same feed, so adding email changes nothing about the routines doing the work -- or about any other destination already reporting. Re-pointing the report later is an `agentmail_report_to` edit, nothing more.
