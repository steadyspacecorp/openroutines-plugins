# Steady inbox ledger

Ids of comments steady-inbox already replied to — checked before replying
so no comment is answered twice. Add an id only after the reply posts
successfully; prune entries once the comment ages out of the collection
window. Nothing else belongs here: state, not a log.

Format:

```markdown
- YYYY-MM-DD — replied to comment <id> on <check-in|goal-update> <id>
```
