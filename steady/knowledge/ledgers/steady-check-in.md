# Steady check-in ledger

Private run state for the check-in routine: the intentions filed last
time, replaced on each successful submission — the next run judges
previous_completed against this copy. Submission state itself is
authoritatively tracked by Steady's action items, so nothing else
belongs here.

Format:

```markdown
- YYYY-MM-DD: filed intentions —
  - <intention line>
```
