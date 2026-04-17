# CodeSleuth AI — Platform Documentation

> Version: v0.1.0 · Powered by CodeSleuth AI

---

## What Is This?

CodeSleuth AI is a **7-agent software engineering pipeline**. A user describes a software product they want built, and the pipeline carries it from raw idea to a verified, security-reviewed, production-ready codebase — automatically, in one continuous run.

Each agent has a single, well-bounded responsibility. Agents hand off structured artifacts to one another and are not allowed to skip or override upstream decisions.

---

## Documentation Index

| Document | Description |
|----------|-------------|
| [platform.md](./platform.md) | Full platform architecture, pipeline flow, artifact system, skill registry, commands, and operational concepts |
| [agent-0-orchestrator.md](./agent-0-orchestrator.md) | Orchestrator — pipeline director and state manager |
| [agent-1-discovery.md](./agent-1-discovery.md) | Product Discovery — requirements and specification |
| [agent-2-planner.md](./agent-2-planner.md) | Technical Planner — architecture and design documents |
| [agent-3-builder.md](./agent-3-builder.md) | Application Builder — continuous implementation |
| [agent-4-security.md](./agent-4-security.md) | Security Agent — adversarial review and BLOCK authority |
| [agent-5-verifier.md](./agent-5-verifier.md) | Codebase Verifier — QA and release gate |
| [agent-6-critic.md](./agent-6-critic.md) | Codebase + Product Critic — final survival review |
| [commands.md](./commands.md) | All available user commands with full descriptions |

---

## Quick Start

The pipeline starts when the Orchestrator receives a kickoff message containing:

- **Project name** — The application to build (e.g., "Invoice Manager SaaS")
- **Platform** — One or more of: `web`, `ios`, `android`, `windows`, `macos`, `linux`
- **Subtype** — e.g., `nextjs`, `express-react`, `swiftui`, `jetpack-compose`
- **Directory** — The target workspace path

The Orchestrator displays a session banner and immediately begins Discovery. No further configuration is required.

---

## Pipeline at a Glance

```
Kickoff Message
      │
      ▼
[0] Orchestrator ── sets design contract, detects multi-platform, checks budget
      │
      ▼
[1] Product Discovery ── 15-phase structured interview → feature-spec.md + HANDOFF.json v1
      │
      ▼
[2] Technical Planner ── TDD + INTERFACES.md + SCHEMA.md + TASK-GRAPH.md + HANDOFF.json v2
      │
      ▼
[3] Application Builder ── continuous build loop → all tasks DONE → SHIP DECLARATION
      │
      ▼
[4] Security Agent ── 20-domain review → APPROVED / CONDITIONAL / BLOCKED
      │
      ▼
[5] Codebase Verifier ── 12-phase verification → SHIP / NO-SHIP per platform
      │
      ▼
[6] Codebase + Product Critic ── 12-category survival review → CRITICISM.md + final verdict
```

---

## Key Conventions

- **Red = Stop. Green = Ship.** No agent claims completion while gates are failing.
- **Files are memory.** State lives in artifact files, not conversation history.
- **HANDOFF.json is the contract.** Every agent reads and updates it as it receives and passes work.
- **Plugin-First skill resolution.** Skills are discovered via `capability-registry.json` — not hardcoded per-agent rules.
- **BLOCK authority is real.** The Security Agent can halt the entire pipeline.
