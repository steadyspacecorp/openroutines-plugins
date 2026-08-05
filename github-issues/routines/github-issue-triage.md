---
# Editing this vendored routine in place may create conflicts when its plugin
# is updated. To override its behavior safely, copy it with the same filename
# into your OpenRoutines agent's routines/ directory and edit that copy.
schedule: "15 * * * *"
timeout: 20m
active: true
skills:
  - github-issues
credentials:
  - github_app_private_key
---
Triage GitHub issues changed in `$GITHUB_REPO` since the cursor in your private ledger.

Use the github-issues skill and obey its authority boundaries. Validate the configured labels before acting. For each changed issue, gather the issue body, recent comments, current labels, and enough repository context to understand whether the report is actionable. Search open and closed issues for plausible duplicates before commenting or changing labels.

Apply only well-supported labels from `$TRIAGE_LABELS`. When essential information is missing, leave one concise, specific request that explains what would make the issue actionable; do not repeat a request already made. Never close an issue, edit its body, assign a person, or mark it as a duplicate. Record those as recommendations instead.

Advance the ledger cursor only after every issue in the selected batch has been handled. Keep unresolved judgments in the ledger so a later run can revisit them without repeating external actions. Record an event summarizing the exact issues reviewed, changes made, and recommendations surfaced. If nothing changed, record that fact without posting anything to GitHub.
