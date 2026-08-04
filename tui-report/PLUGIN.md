---
name: tui-report
description: Print an async check-in to the terminal, on demand -- what the agent did in the last day, what it will do in the next, and whether it is blocked.
---

# tui-report

An on-demand check-in for the operator's own terminal. Run it locally and it syncs the agent's latest memory, then prints what the agent did in the last 24 hours, what it will do in the next 24, and whether anything waits on a human -- the same standup shape the reporting plugins post to Slack or Steady, but read straight off the synced memory, with no destination and no credentials.

## What you get

- **tui-report** (routine) -- reads `memory/events.md`, `memory/tasks.md`, and the run's `schedule.md`, and prints a three-section check-in: last 24 hours, next 24 hours, blocked. It ships inactive and stays that way; the scheduler never fires it -- it exists to be run by hand.
- **bin/tui-report** (operator script) -- the way you actually run it: syncs memory first, wraps the invocation (quiet log level, `--no-memory`), and colorizes the report's landmark lines when stdout is a TTY, plain when piped. It installs ready to run at `plugins/tui-report/bin/tui-report`; routines never see it.

Unlike the reporting consumers, this routine keeps no cursor over the memory feed. It reports a time window, not "everything since last report", so running it twice in a row shows the same picture, and it never affects what slack-report, discord-report, or the Steady check-in will deliver.

## Using it

```bash
plugins/tui-report/bin/tui-report
```

The script begins with `openroutines sync`, so the report always reads the agent's latest pushed memory; a sync refusal (a conflict, a rewritten branch) stops the report rather than printing a stale picture.

The routine prints plain text in a fixed shape -- rules of `─` characters, a name-and-timestamp header, three uppercase section titles -- and the script colorizes those landmark lines at display time (cyan sections, red BLOCKED, dim rules; works with macOS/BSD sed and GNU sed). Color is a display decision the script makes, not something the model has to get right, so redirected output stays clean and the run's trailer and any surviving log lines pass through unchanged.

The script sets `OPENROUTINES_LOG_LEVEL=warn`, which is what keeps the terminal readable: a run's diagnostic log (opencode's INFO stream, passed through line by line) shares the terminal with the echoed report, and warn silences it at the source while still surfacing anything degraded or failing. The report itself is not log lines and prints at any level.

It also passes `--no-memory`, always. The routine itself writes nothing, but a manual run otherwise settles its run record into the local memory worktree -- and local commits on the `memory` branch diverge from what the deployed agent pushes, which the next `openroutines sync` will refuse to reconcile. `--no-memory` keeps the local checkout a pure reader.

No credentials or MCP servers are needed beyond the model provider key every local run already uses.

## After installing

1. `openroutines check`, review the diff -- the script included -- and commit.
2. Leave the routine inactive -- activating it would have the deployed agent print a report into its container logs on a schedule, which is not what this plugin is for.
