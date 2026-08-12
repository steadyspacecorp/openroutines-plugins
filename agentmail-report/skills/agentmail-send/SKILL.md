---
name: agentmail-send
description: Send an email from the agent's AgentMail inbox -- payload shape, subject and body formatting, the message_id delivery check, and the rules that keep an unattended sender well-behaved. Use when a routine needs to send anything by email via $AGENTMAIL_API_KEY.
---

# Sending email via AgentMail

The API key arrives as `$AGENTMAIL_API_KEY`, the sending inbox as
`$AGENTMAIL_INBOX_ID`, and the recipient as `$AGENTMAIL_REPORT_TO`.
Never print the key, never include it in a message. The key is scoped
to this one inbox with only `message_send` and `inbox_read`: it can
send from `$AGENTMAIL_INBOX_ID`, and this skill sends solely to
`$AGENTMAIL_REPORT_TO`.

## Sending

Write scratch files under `$TMPDIR` (the run's writable tmp) -- the
sandbox makes `/tmp` itself read-only, so `/tmp/...` paths fail.

```bash
curl -sS -o "$TMPDIR/agentmail-resp" -w "%{http_code}" -X POST \
  "https://api.agentmail.to/v0/inboxes/$AGENTMAIL_INBOX_ID/messages/send" \
  -H "Authorization: Bearer $AGENTMAIL_API_KEY" \
  -H "Content-Type: application/json" \
  -d @"$TMPDIR/payload.json"
```

Treat exactly HTTP 200 with a response body containing a `message_id`
as delivered. Anything else means not delivered -- error bodies carry
`message` and often a `fix` field; quote both when reporting:

- `401` / `403` -- the key is invalid, revoked, or lacks
  `message_send`; the credential needs re-setting. Raise this as a
  Human-owned task rather than retrying.
- `404` -- `$AGENTMAIL_INBOX_ID` is wrong, the inbox is gone, or the
  key is scoped to a different inbox; same treatment.
- `400` -- the payload is malformed; read the validation error, fix,
  and retry once.
- `429` -- rate limited; wait briefly and retry once, then give up for
  the run.

Do not retry more than once in a run.

## Payload shape

A `subject` that leads with the day's headline outcome -- it is all
most recipients see before deciding to open, so never a generic label
-- then the same digest twice: `text` for plain-text clients and
notification previews, `html` for everyone else. Same facts in both;
neither is a summary of the other.

```json
{
  "to": "$AGENTMAIL_REPORT_TO",
  "subject": "Docs caught up for the 2.1 release -- one ask for the team",
  "text": "What happened\n- Wrote the missing help doc for the CSV export page that shipped with 2.1 (https://example.com/pr/42)\n\nNeeds a human\n- Renew the staging TLS cert -- it expires Friday",
  "html": "<h3>What happened</h3><ul><li>Wrote the missing help doc for the <a href=\"https://example.com/pr/42\">CSV export page</a> that shipped with 2.1</li></ul><h3>Needs a human</h3><ul><li>Renew the staging TLS cert -- it expires Friday</li></ul>"
}
```

(Substitute the real recipient when building the payload; JSON does not
expand `$AGENTMAIL_REPORT_TO`.)

Keep the HTML plain semantic markup -- `<h3>` section headings,
`<ul>`/`<li>` bullets, `<a>` links, `<strong>` for emphasis. No CSS, no
images, no tables for layout; email clients disagree on all of it and
the message is a dozen lines. In the `text` body, links go in
parentheses after the words that describe them.

Each report is a fresh send -- a new thread, never a reply chained onto
yesterday's. The date in the inbox list does the threading.

## Conduct

- Send to exactly `$AGENTMAIL_REPORT_TO` -- never another address,
  never cc or bcc.
- One email per run, maximum. Batch, don't stream.
- No attachments; the report is its own artifact.
- No secrets, tokens, or internal URLs the recipient shouldn't see;
  when unsure, name the thing without linking it.
