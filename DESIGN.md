# pole — Design Document

> Live TUI monitor for polecat processes and Gastown activity.

*Generated from polecat task ha-b4j*

---

## 1. Screen Layout Wireframes (ASCII)

### Main Dashboard

```
╭─────────────────────────────────────────────────────────────────────────────╮
│ pole v0.1.0                                            ↻ 2s │ ? help │ q ╳ │
├─────────────────────────────────────────────────────────────────────────────┤
│ RIGS                          │ READY QUEUE                                 │
│ ─────────────────────────     │ ─────────────────────────────────           │
│ ● halhq       12 open  3 run  │ → ha-1m2  P1  Design: HAL Autonomy          │
│ ● waasp_py     8 open  1 run  │   ha-b4j  P1  Design: pole TUI              │
│ ○ wasp         2 open  0 run  │   wpy-6   P1  E2E Integration Tests         │
│ ○ research     7 open  0 run  │   bf-2    P1  Gemini API integration        │
│                               │                                             │
├───────────────────────────────┼─────────────────────────────────────────────┤
│ ACTIVE POLECATS               │ RECENT COMPLETIONS                          │
│ ─────────────────────────     │ ─────────────────────────────────           │
│ 🦡 1836239  ha-1m2   4m 23s   │ ✓ wpy-kst.3  2m ago   27.5k tokens          │
│ 🦡 1836240  ha-b4j   4m 21s   │ ✓ bf-setup   5m ago    8.2k tokens          │
│ 🦡 1836241  wpy-6    4m 18s   │ ✓ ha-sync   12m ago    3.1k tokens          │
│                               │ ✗ res-036   15m ago   timeout               │
╰─────────────────────────────────────────────────────────────────────────────╯
```

### Rig Detail View

```
╭─────────────────────────────────────────────────────────────────────────────╮
│ halhq                                                  ↻ 2s │ b back │ q ╳ │
├─────────────────────────────────────────────────────────────────────────────┤
│ EPICS                         │ TASKS                                       │
│ ─────────────────────────     │ ─────────────────────────────────           │
│ ▶ ha-bosun    ████████░░ 80%  │ ● ha-bosun-1  P1  --from-gastown    ✓       │
│   ha-pole     ██░░░░░░░░ 20%  │ ● ha-bosun-2  P2  query SQLite      open    │
│   ha-auto     ░░░░░░░░░░  0%  │ ● ha-bosun-3  P2  ticket mgmt       open    │
│                               │ ○ ha-bosun-4  P2  spawn wrapper     open    │
│                               │ ○ ha-bosun-5  P2  standup enhance   open    │
├───────────────────────────────┴─────────────────────────────────────────────┤
│ TASK DETAIL: ha-bosun-1                                                     │
│ ─────────────────────────────────────────────────────────────────           │
│ Title: bosun swarm --from-gastown: pull tasks from Gastown tickets          │
│ Status: ✓ closed  Priority: P1  Created: 2026-02-01 19:36                   │
│ Description: Add flag to bosun swarm that queries Gastown SQLite...         │
╰─────────────────────────────────────────────────────────────────────────────╯
```

---

## 2. Data Sources (SQLite Queries)

### Gastown Database

**Location:** `~/gt/{rig_name}/.beads/beads.db`

```sql
-- List open tasks
SELECT id, title, status, priority, issue_type, created_at
FROM issues 
WHERE status = 'open' AND deleted_at IS NULL
ORDER BY priority, created_at;

-- Count by status per rig
SELECT status, COUNT(*) as count
FROM issues
WHERE deleted_at IS NULL
GROUP BY status;

-- Get blocked tasks
SELECT id, title, await_type, await_id
FROM issues
WHERE await_type IS NOT NULL AND await_type != '';
```

### Polecat Process Data

**Active processes:** `pgrep -f "polecat\|claude -p"`

**Log files:** `/tmp/polecat-*/*.log`

**Run metadata (future):** `~/.polecat/run/{pid}.json`

---

## 3. Refresh Strategy

| Data Type | Interval | Method |
|-----------|----------|--------|
| Rig list | 30s | Filesystem scan of ~/gt/ |
| Task counts | 5s | SQLite aggregate query |
| Task details | 2s | SQLite query (on visible rig) |
| Active polecats | 1s | `pgrep` + `/proc/{pid}/stat` |
| Polecat logs | Real-time | `fsnotify` file watcher |
| Completions | 5s | Log file scan for exit codes |

### Implementation Notes

- Use goroutines with ticker for periodic refresh
- fsnotify for real-time log tailing
- Cache rig list, invalidate on filesystem change
- Debounce rapid updates (100ms)

---

## 4. Key Bindings

### Global

| Key | Action |
|-----|--------|
| `q` | Quit |
| `?` | Toggle help overlay |
| `r` | Force refresh |
| `1` | Dashboard view |
| `2` | Rig detail view |
| `3` | Log view |

### Navigation

| Key | Action |
|-----|--------|
| `j` / `↓` | Move down |
| `k` / `↑` | Move up |
| `h` / `←` | Collapse / back |
| `l` / `→` | Expand / enter |
| `Enter` | Select / drill down |
| `b` / `Esc` | Back to previous view |
| `g` | Go to top |
| `G` | Go to bottom |

### Logs

| Key | Action |
|-----|--------|
| `/` | Search |
| `n` | Next match |
| `N` | Previous match |
| `f` | Toggle filter |
| `F` | Toggle follow mode |
| `y` | Yank (copy) line |

---

## 5. Bubble Tea Component Structure

```
App (root model)
├── Router (view state machine)
├── Views/
│   ├── DashboardView
│   │   ├── RigList (table)
│   │   ├── ReadyQueue (list)
│   │   ├── ActivePolecats (table)
│   │   └── RecentCompletions (list)
│   ├── RigDetailView
│   │   ├── EpicList (list + progress)
│   │   ├── TaskList (table)
│   │   └── TaskDetail (panel)
│   ├── ProcessDetailView
│   │   ├── ProcessInfo (panel)
│   │   └── LogEmbed (viewport)
│   └── LogView
│       ├── LogStream (viewport)
│       ├── FilterBar (textinput)
│       └── SearchOverlay (modal)
├── Components/
│   ├── Table (sortable, selectable)
│   ├── List (scrollable)
│   ├── Panel (bordered content)
│   ├── StatusBar (bottom info)
│   ├── Help (overlay)
│   ├── Progress (bar/spinner)
│   └── Spinner (activity indicator)
└── Data/
    ├── Store (central state)
    ├── RigStore (rig + task data)
    ├── ProcessStore (polecat tracking)
    ├── LogBuffer (ring buffer)
    └── RefreshManager (tickers)
```

### State Flow

```
User Input → Router → Active View → Update Model → Render
                ↓
           Data Layer → SQLite/Filesystem → Store
```

---

## Implementation Priority

1. **P1:** Dashboard view with rig list + active polecats
2. **P1:** Real-time log tailing for active processes
3. **P2:** Rig detail view with task list
4. **P2:** Process detail view
5. **P3:** Search and filter
6. **P3:** Help overlay and polish

---

*Design complete. Ready for implementation.*
