# Steady inbox ledger

State, not a log — two sections. **Handled** holds ids of digest
entries already processed (replies posted, goal movement folded);
checked before acting so nothing is handled twice. Add an id only
after the action succeeds; prune entries once they age out of the
collection window. **Goal board** holds the open goals you're involved
in — your teams', plus any you own or contribute to — one line each;
its header records the last full refresh.

## Handled

```markdown
- YYYY-MM-DD — replied to comment <id> on <check-in|goal-update> <id>
- YYYY-MM-DD — folded goal update <id> into the board
```

## Goal board (refreshed YYYY-MM-DD)

```markdown
- [Title](url) — gist of what it's for. Owner: <name>; involvement:
  <owner|contributor|team>; due YYYY-MM-DD; <on track|at risk|off track>;
  latest: gist of last movement (YYYY-MM-DD)
```
