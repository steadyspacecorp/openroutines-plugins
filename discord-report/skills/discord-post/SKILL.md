---
name: discord-post
description: Post a message to Discord through a channel webhook -- payload shape, embed formatting, the ?wait=true delivery check, structural mention suppression, and the rules that keep an unattended poster well-behaved. Use when a routine needs to send anything to Discord via $DISCORD_WEBHOOK_URL.
---

# Posting to Discord via channel webhook

The webhook URL arrives as `$DISCORD_WEBHOOK_URL`. It both authenticates
and addresses the channel -- never print it, never include it in a
message.

## Sending

Always post with `?wait=true`: without it Discord answers 204 No Content
before the message is created, which proves acceptance, not delivery.

Write scratch files under `$TMPDIR` (the run's writable tmp) -- the
sandbox makes `/tmp` itself read-only, so `/tmp/...` paths fail.

```bash
curl -sS -o "$TMPDIR/discord-resp" -w "%{http_code}" -X POST \
  "$DISCORD_WEBHOOK_URL?wait=true" \
  -H "Content-Type: application/json" \
  -d @"$TMPDIR/payload.json"
```

Treat exactly HTTP 200 with a response body containing the created
message's `id` as delivered. Anything else means not delivered:

- `429` -- rate limited; the body's `retry_after` says how long (in
  seconds). Wait that long and retry once, then give up for the run.
- `404` -- the webhook was deleted; a person must recreate it. Raise
  this as a Human-owned task rather than retrying.
- `400` -- the payload is malformed (usually a length limit); fix and
  retry once.

Do not retry more than once in a run.

## Payload shape

A one-line summary in `content` -- it is the notification preview, so
lead with the day's headline outcome, never a generic label -- the
structured digest in one embed's `description`, and mention suppression
hardcoded:

```json
{
  "content": "Docs caught up for the 2.1 release -- one ask for the team",
  "embeds": [
    {
      "title": "Daily check-in",
      "description": "**What happened**\n- Wrote the missing help doc for the [CSV export page](https://example.com/pr/42) that shipped with 2.1\n\n**Needs a human**\n- Renew the staging TLS cert -- it expires Friday",
      "color": 5793266
    }
  ],
  "allowed_mentions": { "parse": [] }
}
```

Always include `"allowed_mentions": { "parse": [] }`: it makes
`@everyone`, `@here`, and user pings inert no matter what text ends up
in the message -- the report is composed from memory content, so the
no-ping rule is enforced by the payload, not by care.

Discord messages use ordinary markdown -- `[text](url)` links, `**bold**`,
`-` bullets. Prefer masked links over bare URLs so the message doesn't
sprout link-preview embeds. Limits: `content` 2000 characters, embed
`title` 256, embed `description` 4096, at most 10 embeds. One embed is
the format; if a digest would overflow 4096 characters, tighten the
digest -- do not paginate across embeds or messages.

## Conduct

- Never mention users, roles, `@everyone`, or `@here` in the text either
  -- suppression makes them inert, but dead mentions are still noise.
- One message per run, maximum. Batch, don't stream.
- No secrets, tokens, or internal URLs the channel's audience shouldn't
  see; when unsure, name the thing without linking it.
