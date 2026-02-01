# pole — Live Polecat TUI Monitor

A terminal UI for monitoring Gastown activity across all rigs in real-time.

## Current Task

**Design exercise** — create DESIGN.md with architecture and wireframes before implementation.

Tracked by: `ha-b4j` in ~/gt/halhq

## Context

This TUI will display:
- All Gastown rigs and their status
- Active polecat processes
- Epic/ticket dashboards
- Ready queues
- Completion events
- Log tailing

## Tech Stack Options

- **Go + bubbletea/lipgloss** — consistent with ecosystem (wasp, wacli, sweep, drifter)
- **Python + textual** — rapid prototyping

## Gastown Commands

```bash
export PATH="$HOME/go/bin:$PATH"
bd ready          # Ready tasks
bd list --all     # All tasks
bd status         # Rig overview
bd show <id>      # Task details
```

Rigs are in: ~/gt/{halhq,waasp_py,wasp,halhq/research,...}

## When Done

1. Write DESIGN.md with full specification
2. Close ticket: `cd ~/gt/halhq && bd close ha-b4j --reason "Design complete, ready for review"`
