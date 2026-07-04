---
name: system-audit
description: "Two-level system audit. Level 1: inventory everything, score health, output manifest. Level 2: design a custom event bus topology from the manifest. For setting up new agent systems or wiring existing ones into a bus."
user-invocable: true
metadata:
  version: "2.0.0"
  author: "dustinpollock"
  license: "MIT"
  origin: "Level 1 built from a live audit session (2026-04-06). Level 2 added 2026-04-10 after building the OpenClaw event bus."
  references:
    - "references/bus-topology-reference.md"
---

# System Audit

Two levels. Level 1 maps the territory. Level 2 designs the nervous system.

**Level 1:** Inventory skills, scripts, crons, hooks, agents. Score health, measure context weight, map dependencies, check portability. Output: `system-manifest.json`.

**Level 2:** Take the manifest, classify every component as emitter/consumer/both/none, design custom event types, generate bus infrastructure templates, and produce a topology document. Output: bus integration report, wiring plan, topology doc + interactive HTML diagram.

## When to Use

- First time setting up: "What do I actually have?" (Level 1)
- Periodic maintenance: "Is everything healthy?" (Level 1)
- Before publishing: "What's portable, what's trapped?" (Level 1)
- Onboarding someone to your system: "Here's the map" (Level 1 + 2)
- After major changes: "Did anything break?" (Level 1)
- **Setting up an event bus: "I have an agent system, I need a bus"** (Level 1 then 2)
- **Understanding integration gaps: "What's wired and what isn't?"** (Level 2)

## The Pipeline

### Step 0: GitNexus Prerequisite

Before anything else, check for a GitNexus index.

1. Look for a `.gitnexus/` directory in the project root (or `~/.openclaw/.gitnexus/` for OpenClaw-style setups).
2. If found, verify freshness: `npx gitnexus status`. If stale, re-index: `npx gitnexus analyze`.
3. If not found, index the project now: `npx gitnexus analyze`. This takes minutes and gives you the full call graph, dependency map, and execution flows for every step that follows.

**Why this is first:** This skill maps a system and proposes structural changes to it. GitNexus gives you the call graph. Without it, Steps 3-4 reconstruct that graph manually with grep, and Steps 10-13 propose wiring changes to a system you cannot fully see. Read IDEA.md for why that matters.

**If GitNexus cannot be installed:** The skill still runs (Path C fallback at every blast radius step), but flag this to the user: "Running without GitNexus. Dependency analysis will be manual and may miss indirect callers. Recommend indexing before proceeding to Level 2."

### Step 1: Scope and Metrics (autoresearch:plan)

Define what you're auditing and how you'll measure progress.

**Default scope:** Everything in `~/.claude/` and `~/.openclaw/` (or equivalent agent infrastructure).

**Default metrics:**
- Manifest coverage (components cataloged)
- Portability score (% usable by 2+ platforms)
- Skill health average (out of 14)

### Step 2: Health Audit

Run two parallel audits:

**Skill Doctor (14-question health check):**
Launch an agent with this prompt:
> "For each skill in ~/.claude/skills/, read the SKILL.md and score against 14 questions (1 point each): (1) SKILL.md exists with valid YAML frontmatter, (2) name field present and lowercase, (3) description present, (4) user-invocable set, (5) trigger/when-to-use section, (6) workflow section, (7) dependencies declared, (8) version in metadata, (9) author in metadata, (10) references/ subdirectory, (11) clear actionable instructions, (12) no hardcoded paths, (13) error recovery docs, (14) new user could understand it. Output: name | score/14 | failed questions."

**Context Weight Audit:**
Launch an agent with this prompt:
> "Measure byte size of everything that loads into context every session: MEMORY.md, CLAUDE.md, skills registry (~200 chars per entry), hooks (settings.json), each SKILL.md file. Flag anything over 5KB. Report total fixed cost per session."

**Script Portability Audit:**
Launch an agent with this prompt:
> "Audit all scripts for: hardcoded user paths (Y/N), macOS-only commands (Y/N), API dependencies, tool dependencies, internal script dependencies. Summarize: how many hardcoded, how many macOS-only, top 5 most-depended-on scripts."

### Step 3: Dependency Map

Launch an agent with this prompt:
> "Map which scripts are engines for which skills. Produce three lists: (A) Skills with script engines (correctly paired), (B) Skills with no script engine (pure prompt, not portable), (C) Scripts with no skill wrapper. Also map internal dependency chains."

### Step 4: Blast Radius

Before making any changes, analyze what could break. Detect which tool is available, then use it.

**Tool detection (run first):**
1. Check if GitNexus is indexed: look for `.gitnexus/` directory in the project root. If yes, use GitNexus + Hurt Locker path.
2. Check if the user has their own blast radius tool: look for scripts with "blast", "impact", "radius", or "dependency" in the name. If found, ask the user if that's their tool and use it.
3. If neither exists, use the manual path.

**Path A: GitNexus + Hurt Locker (has .gitnexus/ index)**

Launch an agent with this prompt:
> "Run gitnexus_impact on each planned change target. For each: report direct callers (d=1), indirect deps (d=2), risk level. Then run the Hurt Locker skill for a full forward/reverse impact map."

**Path B: User's own blast radius tool**

Ask the user: "Found {tool_name}. Is this your blast radius tool? How do I invoke it?"
Then use it for the same analysis.

**Path C: Manual analysis (no tooling)**

Launch an agent with this prompt:
> "For each planned change from the audit findings, manually trace: grep for every import/source/call of the target script or function. List what depends on it, what breaks if it goes wrong, is it reversible, risk level (LOW/MEDIUM/HIGH). This is slower but catches the same issues."

Key hazards to check (all paths):
- Scripts inside single-quoted heredocs (Python) won't expand `$HOME`
- Trigger extraction needs CLAUDE.md keyword stubs (triggers fire on natural language, skills fire on /name)
- Moving skill directories can break scripts that reference the old path

### Step 5: Do the Work

Ordered by risk (lowest first):

1. **Delete dead weight.** Empty directories, archived skills, ghost registry entries.
2. **Fix hardcoded paths.** Replace `/Users/<username>` with `$HOME` (bash) or `os.path.expanduser("~")` (Python heredocs).
3. **Extract triggers into skills.** Move full protocol from CLAUDE.md to SKILL.md. Leave keyword stubs in CLAUDE.md.
4. **Fix skill health.** Add missing frontmatter fields, error recovery docs, dependencies sections.
5. **Build the manifest.** Catalog every component into system-manifest.json.

### Step 6: Review (PRISM)

Run PRISM on the new/modified skills and the manifest schema. Check for:
- Completeness (does the manifest capture everything?)
- Consistency (are fields used the same way across types?)
- Broken references (stubs pointing to non-existent skills)

### Step 7: Bug Hunt (EyeOfHorus)

Run before shipping. Check:
- `bash -n` on all modified scripts
- YAML validity on all SKILL.md files
- No remaining hardcoded paths
- Manifest is valid JSON
- No broken references to deleted/moved components

### Step 8: Commit

Stage and commit all changes. Push to git.

### Step 9: This Step

You just ran this skill. The output is the audit. The audit is the skill. Recursion with ambition.

---

## Level 2: Bus Topology Generator

Level 2 takes the manifest from Level 1 and designs a custom event bus for the system. For people who don't have a bus yet and need one.

**Prerequisite:** `system-manifest.json` must exist from a Level 1 run. If it doesn't, error: "Run Level 1 first."

**Reference architecture:** `references/bus-topology-reference.md` (the OpenClaw Hugr bus, built 2026-04-10).

### Step 10: Classify Components

Read `system-manifest.json`. For each component, assign a bus role.

Launch an agent with this prompt:
> "Read system-manifest.json. Read references/bus-topology-reference.md for the classification heuristics. For each component in the manifest, assign a bus role using these rules:
>
> - hook -> Emitter (map hook_event to bus event type)
> - cron / oc-job -> Emitter (emit completion/failure)
> - script (engine for skill/cron) -> Emitter
> - script (reads state) -> Consumer
> - agent -> Emitter + Consumer
> - skill -> None (unless it produces artifacts)
> - dashboard -> Consumer
>
> For each emitter, derive event types from what it does. For each consumer, classify into a layer: awareness, action, query, archival, or dashboard.
>
> Output JSON: an array of objects, each with: name, manifest_type, bus_role (emitter/consumer/both/none), event_types (array, for emitters), consumer_layer (string, for consumers), reasoning (one line).
>
> Save to bus-integration-report.json."

### Step 11: Design Event Types

Launch an agent with this prompt:
> "Read bus-integration-report.json. Extract all unique event_types from emitters. For each event type, define: type name, source pattern, one-line description of what it means.
>
> Group by source category (session, git, cron, monitoring, task, sync).
>
> Also determine the bus tier needed:
> - Tier 1 (Minimum): emit-event.sh + bus.jsonl + bus-query.sh. Always needed.
> - Tier 2 (Recommended): Add bus-to-sqlite.sh + bus-rotate.sh. Needed if crons exist.
> - Tier 3 (Advanced): Add bus-compensate.sh + bus-to-memory.sh + wake mode. Needed if agents exist.
>
> Output: event-type-registry.json with the event types, and a 'tier' field indicating recommended tier (1, 2, or 3)."

### Step 12: Generate Wiring Plan

Launch an agent with this prompt:
> "Read bus-integration-report.json and event-type-registry.json. For each component with bus_role of emitter or both that is not yet wired (all of them, since this is a new bus), generate wiring instructions:
>
> For emitters: where to add the emit-event.sh call (which script, after which line/action), what event type, what data payload.
>
> For consumers: what they need to read (bus.jsonl directly, or bus.sqlite via bus-query.sh), which events they care about, how to integrate the query.
>
> Output: wiring-plan.json. Array of objects: component_name, role, event_type, integration_point (file + location), code_snippet (the actual line(s) to add), tier_required."

### Step 13: Blast Radius (Wiring Impact)

Before generating infrastructure or wiring anything, analyze what the bus integration will touch. Same three-path detection as Level 1 Step 4.

**Path A: GitNexus + Hurt Locker**

Launch an agent with this prompt:
> "Read wiring-plan.json. For each component that will be wired, run gitnexus_impact on the integration_point file. Report: what else calls or sources that file, what breaks if the emit-event.sh call fails or hangs, what happens if bus.jsonl is unavailable. Risk level per component. Flag any HIGH risk items for user review before proceeding."

**Path B: User's blast radius tool**

Feed the wiring plan to their tool. Same questions: what touches these files, what breaks if the new bus calls fail.

**Path C: Manual analysis**

Launch an agent with this prompt:
> "Read wiring-plan.json. For each integration_point, grep the codebase for every script that imports, sources, or calls that file. Trace the dependency chain. For each wiring target, answer: (1) What else depends on this file? (2) If the added emit-event.sh call fails, does the original script still complete? (3) If bus.jsonl is missing or locked, does the emitter hang or fail gracefully? Risk level: LOW (isolated script), MEDIUM (called by other scripts), HIGH (in a critical path like session start/stop). Output: blast-radius-report.json."

**Design rule:** Every emit-event.sh call should be fire-and-forget. If the bus is down, the emitting script must still complete its primary job. Flag any wiring plan entry where failure would block the original script.

### Step 14: Generate Infrastructure Templates

Launch an agent with this prompt:
> "Read event-type-registry.json for the tier level and event types. Read references/bus-topology-reference.md for the script patterns.
>
> Generate the bus infrastructure scripts for the recommended tier. For each script:
> - Use $HOME-relative paths (no hardcoded user paths)
> - Use the custom event types from event-type-registry.json
> - Include inline comments explaining each section
> - Make the scripts portable (bash, no macOS-only commands unless unavoidable)
>
> Tier 1 (always): emit-event.sh, bus-query.sh
> Tier 2 (if recommended): bus-to-sqlite.sh, bus-rotate.sh
> Tier 3 (if recommended): bus-compensate.sh, bus-to-memory.sh
>
> Do NOT write these to disk. Output each script's content as a fenced code block in a file called bus-infrastructure-templates.md, with a heading per script and notes on where to place it."

### Step 15: Generate Topology Document

Launch an agent with this prompt:
> "Read bus-integration-report.json, event-type-registry.json, and system-manifest.json. Generate two files:
>
> **1. {project}-bus-topology.md** — Markdown topology document:
> - Title: '{Project} Event Bus Topology'
> - Mermaid flowchart LR diagram with:
>   - Emitters subgraph (grouped by type: hooks, git, crons, agents, scripts)
>   - Central BUS node showing bus.jsonl path and event count
>   - Consumers subgraph (grouped by layer: awareness, action, query, archival, dashboard)
>   - Edge labels with event type names
> - Table: Event Types (type, source, meaning)
> - Table: Cron Schedule (if crons exist)
> - Table: Agents (if agents exist, showing: agent, role, emits, consumes)
> - Table: SQL Views (if Tier 2+)
> - Origin section with generation date
>
> **2. {project}-bus-topology.html** — Interactive version:
> - Dark theme (#0d1117 background, #c9d1d9 text)
> - Mermaid.js loaded from CDN (https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js)
> - Scroll to zoom (mouse-centered), click-drag to pan
> - Preset zoom buttons (50%, 75%, 100%, 150%, 200%) + reset
> - Legend showing agents and navigation hints
> - Same Mermaid diagram as the markdown version
> - Self-contained, shareable (needs internet for Mermaid CDN)
>
> Build the Mermaid syntax programmatically from the data. Do not hardcode nodes."

### Step 16: Review and Present

Present the complete Level 2 output to the user:

1. **Bus Integration Report** — Summary: N components classified, X emitters, Y consumers, Z not applicable.
2. **Event Type Registry** — List the custom event types with descriptions.
3. **Recommended Tier** — Which tier and why.
4. **Wiring Plan** — How many components to wire, estimated effort.
5. **Topology Diagram** — Show the Mermaid preview, point to the HTML file for interactive version.

Ask: "This is the bus design. Want to proceed with wiring, or adjust the plan first?"

Level 2 does NOT wire anything automatically. It produces the map and the templates. The user (or a follow-up session) does the actual integration.

---

## Level 2 Output Files

| File | Contents |
|---|---|
| `bus-integration-report.json` | Every component classified with bus role, event types, consumer layer |
| `event-type-registry.json` | Custom event types for this system, grouped by category, with tier |
| `wiring-plan.json` | Per-component wiring instructions with code snippets |
| `blast-radius-report.json` | Per-component impact analysis of bus wiring, risk levels |
| `bus-infrastructure-templates.md` | Generated scripts (emit-event.sh, bus-query.sh, etc.) as code blocks |
| `{project}-bus-topology.md` | Topology document with Mermaid diagram and tables |
| `{project}-bus-topology.html` | Interactive zoomable topology diagram |

---

## Manifest Schema

```json
{
  "meta": {
    "version": 1,
    "generated": "YYYY-MM-DD",
    "generator": "system-audit",
    "component_count": N
  },
  "components": [
    {
      "name": "string",
      "type": "skill|script|cron|hook|trigger|oc-job|agent",
      "path": "string (relative to ~) or null",
      "description": "one line",
      "dependencies": ["other component names"],
      "platforms": ["claude-cli", "openclaw", "cowork", "any"],
      "primitive": "string or null",
      "health_score": "number or null (skills only, out of 14)",
      "context_weight_bytes": "number or null (skills only)",
      "portability": "portable|cli-only|macos-only|partial",
      "status": "active|disabled|archived|dead",
      "schedule": "crontab expression (crons/oc-jobs only)",
      "hook_event": "SessionStart|SessionEnd|PostToolUse|PreToolUse (hooks only)"
    }
  ]
}
```

## Portability

This skill works in Claude Code, Cowork, or any agent that can launch subagents and read/write files. The audit targets (skills, scripts, crons, hooks) are standard Claude Code / OpenClaw infrastructure. Adapt the scan paths for other setups.

## Error Recovery

**Level 1:**
- If a subagent fails, re-run just that step. Each step is independent once its prerequisites are met.
- If the manifest has invalid JSON, run `python3 -c "import json; json.load(open('path'))"` to find the error.
- If `bash -n` fails on a script, the hardcoded path fix likely broke a heredoc. Check for `$HOME` inside single-quoted heredocs.

**Level 2:**
- If no manifest exists: "Run Level 1 first." Do not attempt classification without inventory.
- If classification produces zero emitters: the system likely has no hooks/crons/agents. Level 2 is premature. Recommend building the system first.
- If Mermaid diagram is too complex (>40 nodes): split into subdiagrams by layer (emitters, consumers) with a summary diagram linking them.
- If generated scripts reference tools the target system doesn't have (e.g., SQLite on a minimal install): drop to the tier that works. Tier 1 has no dependencies beyond bash and jq.
- If the HTML topology won't render: check Mermaid syntax. Common issue: special characters in node labels need quotes.

## Origin

Built from a live audit session, 2026-04-06. Dustin Pollock and EOM ran the full pipeline by hand, recorded every step, then packaged the recording as this skill. The process wrote itself.

Inspired by jeremyknows/watson-toolkit (packaging model) and jeremyknows/claude-context-audit (context weight methodology).

## Related tools

- [eyeofhorus](https://github.com/cogpros/eyeofhorus). Graph-powered bug hunter built on the same GitNexus index. This skill maps and scores the stack, EyeofHorus hunts bugs in it.
