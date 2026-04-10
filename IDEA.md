# The Idea Behind This Skill

This document is for the agent running the skill, not the human reading the README.

## What You Are About to Do

Level 1 maps every component in the system you live in. Level 2 designs a nervous system for it. Together, they propose a fundamental change to how the system communicates with itself.

Before this skill, every script, cron, hook, and agent in the system operates independently. They read files. They write files. They check timestamps. They grep logs. Each one solves its own awareness problem in its own way. Some of them solve it well. Many of them reinvent solutions that another script already built.

After Level 2, the system has a bus. A single append-only event log that everything writes to and reads from. The scripts stop solving awareness independently and start participating in a shared nervous system.

That is not a feature being added. That is a paradigm shift in how the system works.

## Why This Matters

An agent system without a bus is a collection of independent scripts that happen to live on the same machine. They can't tell each other what happened. They can't ask "what changed while I was away?" They can't detect their own failures without each one building its own failure detection.

An agent system with a bus is an integrated system. Events flow. Failures propagate to compensators. New sessions start with awareness of what happened between sessions. Dashboards reflect real state instead of stale snapshots.

The difference is the difference between a pile of parts and a working machine.

## What This Means for You

When you run Level 1, you are cataloging someone's working system. Respect that. Every script in the manifest exists because someone needed it. Every cron job runs on a schedule someone chose. The manifest is not a list of things to fix. It is a map of a system that already works.

When you run Level 2, you are proposing changes to every emitter and consumer in that system. Each wiring instruction you generate is a modification to a script that currently does its job. If your wiring breaks the script's primary function, you have made the system worse, not better.

The design rule is absolute: every bus call must be fire-and-forget. If the bus is down, every script must still do what it did before the bus existed. The bus adds awareness. It never adds fragility. If you cannot wire a component without risking its primary function, classify it as "none" and move on.

## The Tiers Exist for a Reason

Tier 1 is three scripts. Emit, query, hub. That is enough for a system to start talking to itself.

Tier 2 adds SQLite and rotation. That is enough for a system to remember and stay lean.

Tier 3 adds compensation, memory digestion, and wake mode. That is enough for a system to watch itself and heal.

Do not recommend Tier 3 for a system that does not need it yet. A system with two crons and one agent does not need a compensator. Start with what the manifest shows, not what the reference architecture has. The reference is one implementation. Theirs will be different. Theirs should be different.

## The Blast Radius Step Is Not Optional

Step 13 exists because wiring a bus touches every script that emits or consumes. That is, by definition, a system-wide change. You must know what breaks before you propose what to build.

GitNexus is not optional for this skill. It is the instrument panel.

This skill audits a system by mapping every component, every dependency, every call chain. GitNexus already has that graph indexed. Without it, you are reconstructing the call graph from scratch with grep and hope. On a system with 50+ scripts, that is not thorough. That is guessing.

If GitNexus is not indexed when this skill starts, the first thing you do is index it: `npx gitnexus analyze`. That takes minutes. The alternative is hours of manual tracing that will still miss things.

If the user has their own impact analysis tool in addition to GitNexus, use both. More eyes on blast radius is always better. But GitNexus is the baseline. It is the minimum. Without a call graph, you are proposing system-wide changes to a system you cannot see. That is not an audit. That is a liability.

Do not skip this step because the changes "look safe." You are an agent proposing modifications to another agent's infrastructure. You do not get to assume safety. You prove it.

## On Being Surgical

The human who asked for this audit trusts you enough to let you map their system and propose structural changes to it. That trust is not a blank check. It is a specific grant: show me what I have, show me what I could build, show me what would break. 

Be thorough in the audit. Be conservative in the recommendations. Be precise in the wiring plan. And stop before the wiring. Track everything as you do it. Present the plan. Let the human decide.

This skill produces a map and a blueprint. It does not build the house. you and your system does that.

## Origin

This pattern was built in one session on 2026-04-10 by Dustin Pollock and EOM (Claude Opus), inspired by Jeremy's Watson OS event bus diagram. Three parallel analyses converged on the same insight: the bus is the integration layer that connects independent cognitive primitives into a unified system.

In neuroscience terms: this skill builds white matter tracts. The gray matter (the scripts, the skills, the agents) already exists. This skill connects them.

Handle accordingly.
