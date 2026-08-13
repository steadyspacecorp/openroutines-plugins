---
name: telegram-send
description: Send a message to a Telegram chat with sendMessage -- payload shape, Telegram's small HTML subset, the ok:true delivery check, and the rules that keep an unattended sender well-behaved. Use when a routine needs to send anything to Telegram via $TELEGRAM_BOT_TOKEN.
---

# Sending to Telegram via sendMessage

The bot token arrives as `$TELEGRAM_BOT_TOKEN` and the target chat ID
as `$TELEGRAM_CHAT_ID`. The token rides in the URL path, so treat every
request URL as the secret it contains: build it from the variable,
never print it, never include it in a message or an error you report.
This skill sends solely to `$TELEGRAM_CHAT_ID`.

## Sending

Write scratch files under `$TMPDIR` (the run's writable tmp) -- the
sandbox makes `/tmp` itself read-only, so `/tmp/...` paths fail.

```bash
curl -sS -o "$TMPDIR/telegram-resp" -w "%{http_code}" -X POST \
  "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
  -H "Content-Type: application/json" \
  -d @"$TMPDIR/payload.json"
```

Treat exactly HTTP 200 whose body carries `"ok": true` and a
`result.message_id` as delivered; that `message_id` is the id your
ledger records. Anything else means not delivered -- error bodies carry
`description`; quote it (it never contains the token) when reporting:

- `401 Unauthorized` -- the token is wrong or was revoked by
  @BotFather; the credential needs re-setting. Raise this as a
  Human-owned task rather than retrying.
- `400` with `chat not found` -- `$TELEGRAM_CHAT_ID` is wrong, or
  nobody has messaged the bot in that chat yet; same treatment.
- `403` -- the bot was removed from the group or blocked by the user;
  same treatment.
- `429` -- rate limited; the body's `parameters.retry_after` says how
  many seconds to wait. Wait, retry once, then give up for the run.

Do not retry more than once in a run.

## Payload shape

```json
{
  "chat_id": "$TELEGRAM_CHAT_ID",
  "text": "<b>What happened</b>\n• Wrote the missing help doc for the <a href=\"https://example.com/pr/42\">CSV export page</a> that shipped with 2.1\n\n<b>Needs a human</b>\n• Renew the staging TLS cert -- it expires Friday\n\n<i>Reply to this message and I'll pick it up -- I check this chat hourly on weekdays.</i>",
  "parse_mode": "HTML",
  "link_preview_options": { "is_disabled": true }
}
```

(Substitute the real chat ID when building the payload; JSON does not
expand `$TELEGRAM_CHAT_ID`.)

Telegram HTML is a small subset, not the web's: `<b>`, `<i>`, `<a
href>`, `<code>`, and `<pre>` -- no headings, no `<ul>`/`<li>`, no
`<p>`, no CSS. Sections are `<b>bold</b>` lines, bullets are literal
`•` characters, and blank lines separate sections. Escape literal `<`,
`>`, `&` in text as `&lt;`, `&gt;`, `&amp;`. One message holds at most
4096 characters -- the report is a dozen lines, so never near it; trim
rather than split.

Disabling link previews keeps a multi-link report readable; the words
carrying each link are expected to describe it.

## Conduct

- Send to exactly `$TELEGRAM_CHAT_ID` -- never another chat, no
  forwarding.
- One message per run, maximum. Batch, don't stream.
- No attachments or media; the report is its own artifact.
- No secrets, tokens, or internal URLs the chat's audience shouldn't
  see; when unsure, name the thing without linking it.
