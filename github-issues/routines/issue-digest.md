---
schedule: "0 14 * * 5"
timeout: 20m
active: true
skills:
  - github-issues
credentials:
  - github_app_private_key
events: false
---
Produce a weekly issue-health digest for `$GITHUB_REPO` and print it to the container logs.

Use the github-issues skill in read-only mode. Compare the current open-issue set with issues created, closed, and meaningfully updated during the last seven days. Identify recurring themes with links to representative issues, issues inactive for at least `$STALE_AFTER_DAYS` days, likely duplicates that need a person's judgment, and concrete product or maintenance decisions blocking progress.

Write for a teammate who needs signal, not an inventory. Include counts only when they clarify a trend. Name every issue by number, title, and URL so each claim can be checked. Distinguish observed facts from your inference. If the week was quiet, say so plainly and mention only material exceptions.

Do not modify GitHub, memory, or the triage ledger. This routine reports the repository's current state directly and has no delivery destination beyond its logs until the agent owner deliberately adds one.
