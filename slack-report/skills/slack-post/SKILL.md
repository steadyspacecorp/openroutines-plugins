---
name: slack-post
description: Post a message to a Slack channel with chat.postMessage -- payload shape, Block Kit formatting, the ok:true delivery check, and the rules that keep an unattended poster well-behaved. Use when a routine needs to send anything to Slack via $SLACK_BOT_TOKEN.
---

# Posting to Slack via chat.postMessage

The bot token arrives as `$SLACK_BOT_TOKEN` and the target channel ID as
`$SLACK_CHANNEL`. Never print the token, never include it in a message.
The token's only scope is `chat:write`: it can post solely to channels
the bot has been invited to, and this skill posts solely to
`$SLACK_CHANNEL`.

## Sending

Write scratch files under `$TMPDIR` (the run's writable tmp) -- the
sandbox makes `/tmp` itself read-only, so `/tmp/...` paths fail.

```bash
curl -sS -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d @"$TMPDIR/payload.json" > "$TMPDIR/slack-resp"
```

Delivery is `"ok": true` in the response body -- **Slack returns HTTP 200
even for failures**, so never treat the status code as success. On
`"ok": false` read the `error` field; the common ones:

- `not_in_channel` -- the bot was never invited (or was removed); a
  person must `/invite` it. Raise this as a Human-owned task rather than
  retrying.
- `channel_not_found` -- `$SLACK_CHANNEL` is wrong or the channel is
  gone; same treatment.
- `invalid_auth` / `token_revoked` -- the credential needs re-setting.

Do not retry more than once in a run.

## Payload shape

Always include the channel, a `text` fallback (used by notifications and
screen readers -- lead with the day's headline outcome, never a generic
label), then `blocks` for structure:

```json
{
  "channel": "$SLACK_CHANNEL",
  "text": "Docs caught up for the 2.1 release -- one ask for the team",
  "blocks": [
    { "type": "header", "text": { "type": "plain_text", "text": "Daily check-in" } },
    { "type": "section", "text": { "type": "mrkdwn", "text": "*What happened*\n• Wrote the missing help doc for the <https://example.com/pr/42|CSV export page> that shipped with 2.1" } },
    { "type": "section", "text": { "type": "mrkdwn", "text": "*Needs a human*\n• Renew the staging TLS cert -- it expires Friday" } }
  ]
}
```

(Substitute the real channel ID when building the payload; JSON does not
expand `$SLACK_CHANNEL`.)

Slack mrkdwn is not markdown: links are `<url|text>`, bold is `*text*`,
bullets are literal `•` characters. Keep any single section block under
3000 characters; split long lists across blocks.

## Conduct

- Never use `@channel`, `@here`, or user pings -- an unattended agent
  earns attention with content, not interruptions.
- One message per run, maximum. Batch, don't stream.
- No secrets, tokens, or internal URLs the channel's audience shouldn't
  see; when unsure, name the thing without linking it.
