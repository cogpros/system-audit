# system-audit

Two-level audit for an agent stack. Level 1 inventories every skill, script, cron, hook, and agent, scores health, and outputs a manifest. Level 2 takes that manifest and designs a custom event bus topology: who emits, who consumes, what event types exist, and what the wiring plan looks like. An agent system without shared awareness is a pile of independent scripts. This skill maps the pile and designs the nervous system.

Pollock 2026.

## What it does

**Level 1: inventory and health.**
- Skill health audit (14-question check per SKILL.md)
- Context weight audit (what loads into every session, flagged over 5KB)
- Script portability audit (hardcoded paths, macOS-only commands, dependencies)
- Dependency map (which scripts are engines for which skills)
- Blast radius analysis before any change
- Output: `system-manifest.json`, one entry per component

**Level 2: bus topology design.**
- Classifies every manifest component as emitter, consumer, both, or none
- Derives custom event types and groups them by source category
- Recommends a bus tier: Tier 1 (emit + query + hub file), Tier 2 (adds SQLite mirror + rotation), Tier 3 (adds compensation + memory digestion)
- Generates a wiring plan with per-component code snippets, a second blast radius pass on the wiring itself, infrastructure script templates, and a topology document with a Mermaid diagram plus an interactive HTML version
- Design rule: every bus call is fire-and-forget. If the bus is down, every script still does its original job.

Level 2 produces the map and the templates. It wires nothing. You decide what gets integrated.

## Step 0: GitNexus

[GitNexus](https://github.com/githubnext/gitnexus) is the prerequisite. This skill proposes structural changes to a live system, and GitNexus provides the call graph that makes blast radius analysis real instead of guessed. Before anything else, the skill checks for a `.gitnexus/` index and runs `npx gitnexus analyze` if it finds none.

If GitNexus cannot be installed, the skill still runs. Every blast radius step has a manual fallback path that traces dependencies with grep. It is slower and can miss indirect callers, and the skill flags that to you before proceeding to Level 2.

## Requirements

- An agent runtime that reads SKILL.md, launches subagents, and reads and writes files. Built in Claude Code; also runs in Cowork or equivalents.
- GitNexus (via `npx`) for the indexed blast radius path. Optional but strongly recommended.
- bash and jq for the Tier 1 bus templates. Tier 2 adds SQLite.

## Install

```bash
mkdir -p ~/.claude/skills/system-audit
git clone https://github.com/cogpros/system-audit.git ~/.claude/skills/system-audit
```

Or clone anywhere and symlink into `~/.claude/skills/`.

## First run

In a session:

```
/system-audit
```

Level 1 runs first: scope, health audits, dependency map, blast radius, fixes ordered by risk, manifest. Review the manifest before asking for Level 2. Level 2 refuses to run without one.

The default scan scope is `~/.claude/` and `~/.openclaw/`. Adapt the paths for other setups. <!-- commit-leak-scan: allow (path already public in SKILL.md) -->

## Files

| File | What it is |
|---|---|
| `SKILL.md` | The full 16-step pipeline, manifest schema, error recovery |
| `IDEA.md` | Design rationale for the agent running the skill |
| `references/bus-topology-reference.md` | Reference bus architecture (the OpenClaw Hugr bus, 2026-04-10) that Level 2 applies to new systems |

## Origin

Built from a live audit session, 2026-04-06. Dustin Pollock and EOM ran the full pipeline by hand, recorded every step, then packaged the recording as this skill. Level 2 added 2026-04-10 after building the OpenClaw event bus.

## Related

- [eyeofhorus](https://github.com/cogpros/eyeofhorus) is the other GitNexus rider. Both skills sit on the same index: this one maps and scores the stack, EyeOfHorus hunts bugs in it.

## License

MIT.
