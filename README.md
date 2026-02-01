# pole 📊

Live TUI monitor for polecat processes and Gastown activity.

## What

A terminal UI displaying real-time status of:
- All Gastown rigs and their health
- Active polecat (sandboxed Claude CLI) processes
- Epic/ticket dashboards
- Ready queues and completion events
- Log tailing

## Why

Visibility into the agentic ecosystem. See what's running, what's blocked, what's done.

## Stack

- Go + [Bubble Tea](https://github.com/charmbracelet/bubbletea) TUI
- Watches Gastown SQLite + polecat logs
- Real-time updates

## Status

Design phase. See `DESIGN.md` (when created) for architecture.

## Related

- [polecat](https://github.com/rickhallett/polecat) — The sandboxed Claude CLI runner this monitors
- Gastown — The ticket/epic system being visualized

## License

MIT
