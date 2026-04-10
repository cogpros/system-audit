# Event Bus Reference Architecture

Reference implementation from the OpenClaw Hugr (Pollock, 2026-04-10). This is the pattern Level 2 applies to new systems.

## Core Pattern

```
Emitters --> bus.jsonl (append-only) --> Consumers
```

Single write API (`emit-event.sh`). One hub file. Multiple consumer layers.

## Event Schema

Every event in bus.jsonl follows this shape:

```json
{
  "ts": "ISO-8601 UTC",
  "type": "event_type",
  "source": "component:subcomponent",
  "data": { "...context-specific fields..." }
}
```

## Consumer Layers

| Layer | Purpose | Example |
|---|---|---|
| Awareness | "What happened while I was away?" | memory-pulse-wake.sh reads last 6h on session start |
| Action | "Something failed, fix or escalate" | bus-compensate.sh escalates breakers via Discord |
| Query | "Let me look things up" | bus-to-sqlite.sh mirrors to SQL, bus-query.sh wraps CLI |
| Archival | "Keep the bus lean, persist the signal" | bus-rotate.sh archives old events, bus-to-memory.sh digests |
| Dashboard | "Show me the state" | Vedrfolnir reads bus.sqlite |

## Infrastructure Scripts

### Tier 1: Minimum Viable Bus

**emit-event.sh** — Single write API. All emitters call this.
```bash
# Usage: emit-event.sh <type> <source> [json_data]
# Appends one JSON line to bus.jsonl
# Validates type is known, adds timestamp
```

**bus-query.sh** — CLI reader.
```bash
# Subcommands: latest, since <time>, failures, state, daily, count, raw
# Reads bus.jsonl directly (no SQLite needed)
```

### Tier 2: Recommended (add when crons exist)

**bus-to-sqlite.sh** — Mirrors JSONL to SQLite. Creates views:
- `v_latest_state` — Current state per component
- `v_failure_streaks` — Consecutive failure counts
- `v_baseline_metrics` — 30-day normals
- `v_recent` — Last 6 hours
- `v_daily_summary` — Event counts by type today

**bus-rotate.sh** — Archives events older than N days to monthly files.

### Tier 3: Advanced (add when agents exist)

**bus-compensate.sh** — Reads failure events, escalates via notification channel.

**bus-to-memory.sh** — Digests events into daily markdown files for agent context.

**memory-pulse-wake.sh** — Session start hook. Feeds awareness layer into agent context.

## Event Type Patterns

| Component type | Typical event types |
|---|---|
| Session hooks | session_start, session_stop |
| Agent subagents | subagent_complete |
| Git hooks | commit_shipped |
| Cron jobs | cron_completed, plus specific types (health_check, memory_ingested) |
| Monitoring scripts | divergence, breaker_open, auto_repair, escalation |
| Task systems | task_completed |
| Sync jobs | bus_synced, compensation_complete |

## Classification Heuristics

How to decide what role a component plays:

| Manifest type | Default role | Reasoning |
|---|---|---|
| hook | Emitter | Hooks fire on events. Emit what happened. |
| cron / oc-job | Emitter | Scheduled work. Emit completion/failure. |
| script (engine) | Emitter | If it does work, it should announce it. |
| script (reader) | Consumer | If it reads state, it consumes from the bus. |
| agent | Emitter + Consumer | Agents emit session events, consume wake context. |
| skill | None (usually) | Skills are invoked, not bus-driven. Exception: skills that produce artifacts. |
| dashboard | Consumer | Reads bus state for display. |

## Topology Document Structure

The generated topology doc follows this outline:

1. Title + subtitle (project name, date, one-line purpose)
2. Mermaid flowchart LR
   - Emitters subgraph (grouped by type: hooks, git, crons, agents)
   - BUS node (center)
   - Consumers subgraph (grouped by layer: awareness, action, query, archival, dashboard)
   - Edges with event type labels
3. SQL Views table (if Tier 2)
4. Event Types table (type, source, meaning)
5. Cron Schedule table (if crons exist)
6. Agents table (agent, role, emits, consumes)
7. Origin section

## Origin

Built in one session (2026-04-09/10) after Jeremy (jeremyknows) shared the Watson OS event bus topology diagram. Three Mimir runs converged: the bus is Primitive 17 (Binding), the integration layer connecting independent cognitive primitives. In TBI terms: white matter tracts.

Reference implementation: `~/.openclaw/events/bus.jsonl` + 8 infrastructure scripts + 12 rewired consumers + 2 CC hooks + global git hook.
