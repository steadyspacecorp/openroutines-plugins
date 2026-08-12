---
name: agentmail-report
description: Email the agent's knowledge change feed to a recipient as a teammate-style update, sent from the agent's own AgentMail inbox via an inbox-scoped API key.
credentials:
  agentmail_api_key:
    description: An AgentMail API key scoped to the sending inbox with only the message_send and inbox_read permissions, minted per the steps below
variables:
  agentmail_inbox_id:
    description: The ID of the AgentMail inbox the agent sends from (from the inbox's details in the console or the create-inbox response)
  agentmail_report_to:
    description: The email address the report goes to -- one address; use a group or list address to reach a team
---

# agentmail-report

Points an agent's reporting at an email address. The routine is a knowledge-feed consumer: each workday morning it turns everything the agent recorded since its last report into one short, human email and sends it from the agent's own [AgentMail](https://docs.agentmail.to) inbox.

## What you get

- **agentmail-report** -- consumes the knowledge change feed and emails a digest: what happened, what's now on someone's plate, what changed in the task list. Nothing new since last time means no email -- the recipient never gets a "nothing to report" message.
- **agentmail-verify** -- a manual-only wiring check: run it after setup and it confirms the key and inbox (a bare GET, no send) then sends one labeled test email. It ships inactive and stays that way; the scheduler never fires it. Run it with `OPENROUTINES_LOG_LEVEL=warn openroutines routines run agentmail-verify --no-knowledge` (quiet diagnostics, and `--no-knowledge` so a local run leaves the knowledge worktree untouched).
- **agentmail-send** skill -- how to format and send the email: subject as the day's headline, matching plain-text and HTML bodies, the `message_id` delivery check, exactly one recipient.

## Why an inbox-scoped key

AgentMail's default API keys are organization-wide: one key reaches every inbox in the account, reading and sending alike. Keys can instead be minted per inbox with per-permission grants, and this plugin expects one scoped to the sending inbox with only `message_send` and `inbox_read`. That key can send from its one inbox and confirm the wiring, and nothing else -- it cannot read mail, touch other inboxes, or create new ones. The organization key is used once, below, on your machine; it never enters the agent.

## Create the inbox and key

1. Sign up at [console.agentmail.to](https://console.agentmail.to) and copy an organization API key from the console.

2. Create the agent's inbox. The `display_name` is the sender name recipients see, so name it after your agent:

   ```bash
   curl -sS -X POST https://api.agentmail.to/v0/inboxes \
     -H "Authorization: Bearer <org-api-key>" \
     -H "Content-Type: application/json" \
     -d '{"username": "my-agent", "display_name": "My Agent"}'
   ```

   Note the `inbox_id` in the response.

3. Mint the inbox-scoped key:

   ```bash
   curl -sS -X POST https://api.agentmail.to/v0/inboxes/<inbox_id>/api-keys \
     -H "Authorization: Bearer <org-api-key>" \
     -H "Content-Type: application/json" \
     -d '{"name": "openroutines-report", "permissions": {"message_send": true, "inbox_read": true}}'
   ```

   Copy the `api_key` value from the response.

## After installing

1. `openroutines credentials set agentmail_api_key` -- the inbox-scoped key from the step above.
2. Set the `agentmail_inbox_id` and `agentmail_report_to` variables in `openroutines.yml`.
3. `OPENROUTINES_LOG_LEVEL=warn openroutines routines run agentmail-verify --no-knowledge` -- confirms the key and inbox and sends one labeled test email through the real wiring.
4. Adjust the schedule, then `openroutines check`, review the diff, commit.

The script's `--no-knowledge` discards knowledge settlement only; it does not suppress email or withhold credentials. `openroutines check` is the non-acting validation path.

Works alongside other consumers: each keeps its own cursor over the same feed, so adding email changes nothing about the routines doing the work -- or about any other destination already reporting. Re-pointing the report later is an `agentmail_report_to` edit, nothing more.
