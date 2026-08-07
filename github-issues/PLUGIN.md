---
name: github-issues
description: Triage a GitHub repository's issues as they change and publish a weekly digest of themes, stale work, and decisions that need a person.
credentials:
  github_app_private_key:
    description: Private key PEM for a GitHub App installed on the target repository with read/write Issues permission and read Metadata permission
    type: github_app
variables:
  github_repo:
    description: The target repository in owner/repo form, for example steadyspacecorp/openroutines
  triage_labels:
    description: Comma-separated labels the steward may apply, for example bug,enhancement,documentation,needs-info
  stale_after_days:
    description: Number of inactive days after which an open issue should be surfaced for review, for example 30
---

# github-issues

Gives an agent a disciplined GitHub issue-steward workflow: keep new and changed issues organized, ask for missing information without pretending certainty, and turn the issue tracker into a useful weekly signal instead of a backlog-shaped mystery.

## What you get

- **github-issue-triage** -- reviews issues changed since its last successful pass, searches for likely duplicates, applies only configured labels, asks focused follow-up questions when an issue is not actionable, and records its cursor and unresolved judgments in a private ledger.
- **github-issue-digest** -- once a week, summarizes issue volume, recurring themes, stale work, and decisions that need a person. It reads GitHub directly; it does not consume the agent's knowledge feed, so adding it does not affect other reporting consumers.
- **github-issues** skill -- a conservative `gh`-based workflow for querying, labeling, commenting, duplicate detection, and producing evidence-backed summaries.

## Required GitHub App

Create a GitHub App with:

- Repository metadata: read
- Issues: read and write

Install it on exactly the repository named by `github_repo`. OpenRoutines' `github_app` credential type requires the App to have one installation and injects a short-lived `GH_TOKEN` plus the App's Git identity into each run.

## After installing

1. Add the typed credential metadata to `openroutines.yml`:

   ```yaml
   credentials:
     github_app_private_key:
       type: github_app
       app_id: "YOUR_APP_ID"
   ```

2. Set `github_repo`, `triage_labels`, and `stale_after_days` in the `variables:` map.
3. `openroutines credentials set github_app_private_key`
4. Adjust both schedules and review the policy in `skills/github-issues/SKILL.md`.
5. Run `openroutines routines test github-issue-triage` and `openroutines routines test github-issue-digest`.
6. Run `openroutines check`, review the complete diff, then activate the routines you want.

The steward never closes issues, edits issue bodies, assigns people, or marks duplicates automatically. Those are higher-judgment actions left as explicit recommendations unless you deliberately change the vendored policy.
