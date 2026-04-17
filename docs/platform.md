# CodeSleuth AI — Platform Architecture

> Version: v0.1.0

---

## Overview

CodeSleuth AI is a **7-agent software engineering pipeline** that converts a product idea into a verified, security-reviewed codebase. It is designed to operate as a single unified assistant — the user never manually routes between agents.

The pipeline is:

```
Orchestrator (0) → Discovery (1) → Planner (2) → Builder (3) → Security (4) → Verifier (5) → Critic (6)
```

---

## Supported Platforms

Each project targets one or more of the following platforms:

| Platform | Key Technologies |
|----------|-----------------|
| 🌐 Web (Next.js) | Next.js 14+ App Router, TypeScript strict, shadcn/ui, Tailwind CSS, Framer Motion, Prisma, PostgreSQL, Redis |
| 🌐 Web (Express + React) | Vite+React SPA, Express.js, TypeScript, Prisma, Helmet, Passport.js |
| 📱 iOS | SwiftUI, Swift, MVVM, StoreKit 2, Keychain, APNs |
| 🤖 Android | Jetpack Compose, Kotlin, Hilt, Room, Google Play Billing |
| 🪟 Windows | Electron+React, Tauri+React, or WinUI 3 / .NET MAUI |
| 🍎 macOS | SwiftUI+AppKit, Electron+React, or Tauri+React |
| 🐧 Linux | GTK4/libadwaita, Electron+React, or Tauri+React |

---

## Artifact System

Agents communicate via structured files in the `artifacts/` directory. These are the authoritative state files — **not conversation history**.

| Artifact | Produced By | Consumed By | Description |
|----------|------------|-------------|-------------|
| `artifacts/discovery/feature-spec.md` | Discovery (1) | Planner (2) | Full compiled product specification |
| `artifacts/discovery/HANDOFF.json` | Discovery (1) | All downstream | Machine-readable pipeline contract; updated by each agent |
| `artifacts/build/TDD.md` | Planner (2) | Builder (3) | Technical Design Document — architecture, stack, data model |
| `artifacts/build/INTERFACES.md` | Planner (2), updated by Builder (3) | Builder (3) | Every module's exported functions, types, and middleware |
| `artifacts/build/SCHEMA.md` | Planner (2) | Builder (3), Security (4) | Locked data model with entities, relationships, indexes |
| `artifacts/build/TASK-GRAPH.md` | Planner (2), updated by Builder (3) | Builder (3) | Dependency graph of all implementation tasks with file ownership |
| `artifacts/build/CHECKPOINT.md` | Builder (3) | Builder (3) recovery | Rolling build state snapshot, overwritten every 3–5 tasks |
| `artifacts/build/DECISIONS.md` | Builder (3) | Critic (6) | Log of all non-trivial implementation decisions and rationale |
| `artifacts/build/phases.md` | Builder (3), Verifier (5) | Verifier (5) | Gate pass/fail results per task and per phase |
| `artifacts/build/FEATURES.md` | Planner (2), updated by Builder (3) | Verifier (5) | Canonical list of implemented features per platform |
| `artifacts/security/SECURITY_REPORT.md` | Security (4) | Verifier (5), Critic (6) | Full 20-domain security review output |
| `artifacts/critique/CRITICISM.md` | Critic (6) | End user | Final product + engineering critique with per-platform verdicts |
| `artifacts/PIPELINE_MEMORY.md` | Orchestrator post-run hook | Future runs | Accumulated pipeline patterns and improvement proposals |

### HANDOFF.json Schema (v1 → v5)

The HANDOFF.json file is the shared contract that evolves through the pipeline:

```json
{
  "spec_version": "1.0.0",
  "pipeline_stage": "discovery_complete",
  "platform_scope": ["web"],
  "web_stack": "nextjs",
  "design_contract": {
    "active_reference": "common-design-threads | user-override | generated | none",
    "override_description": null,
    "theme_name": null
  },
  "entity_model": {},
  "api_contracts": [],
  "ipc_contracts": [],
  "native_capabilities": {},
  "auth_model": {},
  "feature_parity_matrix": {},
  "store_targets": [],
  "monetization": {},
  "i18n_scope": [],
  "security_review_status": "pending | approved | blocked",
  "verification_status": "pending | ship | no-ship",
  "open_blockers": [],
  "concept_validation": "proceed | revise | rethink",
  "orchestration_mode": "single | multi",
  "monorepo_strategy": "turborepo | expo | tauri | nx | polyrepo"
}
```

---

## Skill System (Plugin-First)

Agents extend their capabilities by invoking **skills** — reusable capability packages with a standardized `SKILL.md` file.

### How Skills Are Resolved

At runtime, the Orchestrator uses the **Capability Registry** as the exclusive mechanism for skill discovery:

| File | Purpose |
|------|---------|
| `scripts/capability-registry.json` | Machine-readable registry with full manifest data |
| `CAPABILITY_REGISTRY.md` | Human-readable registry for reference |
| `scripts/match-skills.py` | Scores skills against any task context for testing |
| `scripts/build-capability-registry.sh` | Rebuilds registry from skill manifests |

### Scoring Algorithm

```
score(skill, context) =
  (agent_match × 3)      // skill targets the current agent
  + (phase_match × 2)    // skill targets the current pipeline phase
  + (domain_match × 2)   // skill domains intersect task domains
  + (capability_match × 1) // skill capabilities intersect task needs
  + (priority_bonus)     // critical=3, high=2, medium=1, low=0
```

### Invocation Thresholds

| Trust Level | Auto-invoke threshold | Suggest threshold |
|------------|----------------------|------------------|
| 🔐 Pipeline skills | ≥ 5 | 3–4 |
| 🌐 Community skills | ≥ 7 (no side effects) | 3–6 |
| 🌐 Community with side effects | Never auto-invoke | ≥ 3 (suggest only) |

### Skill Collections

| Collection | Location | Description |
|-----------|----------|-------------|
| Engineering | `skills/claude-skills/engineering/` | Code review, CI/CD, debugging, security |
| Engineering Team | `skills/claude-skills/engineering-team/` | QA, senior SecOps, performance profiling |
| Product Team | `skills/claude-skills/product-team/` | UX research, product strategy, analytics |
| C-Level Advisor | `skills/claude-skills/c-level-advisor/` | CFO, competitive intelligence |
| Superpowers | `skills/superpowers/skills/` | Brainstorming, verification, systematic debugging |
| Community | `skills/community/` | Third-party skills |

---

## Design Contract System

For web UI projects, CodeSleuth enforces a **design contract** — a set of concrete implementation rules covering spacing, typography, color, animation, components, and section architecture.

### How It Works

1. **Orchestrator** detects whether a design contract applies at kickoff.
2. **Discovery** captures any user-provided design overrides.
3. **Planner** generates TDD Section 5A — concrete implementation rules derived from the active contract.
4. **Builder** treats Section 5A as an immutable implementation constraint (equal authority to the API contract).
5. **Verifier** checks conformance in Phase 6A (UI/UX Design Contract Verification).
6. **Critic** critiques the visual quality across 8 dimensions in Category L.

### Contract States

| State | Banner Display | When |
|-------|---------------|------|
| Default contract active | `common-design-threads.md (default)` | Web UI project, no user design direction |
| User override | `User-provided: [brief description]` | User gave explicit visual direction |
| Generated | `User-provided direction (will generate...)` | User gave partial brand inputs |
| Not applicable | `N/A (non-web / no UI)` | CLI, API, or non-web project |

---

## Multi-Platform Fork Architecture

When a project targets 2+ platforms, the Orchestrator can fork the pipeline into parallel tracks with isolated state.

### Fork Execution Order

1. Discovery runs **once** (shared)
2. Planning runs **per platform** → separate TDD, INTERFACES, SCHEMA, TASK-GRAPH per platform
3. Builder runs **per platform**
4. Security runs **per platform**
5. Verifier runs **per platform**
6. Critic runs **once** with all verification reports merged

### Multi-Platform Verdict

```
MULTI-PLATFORM VERDICT
──────────────────────
 🌐 web:      ✅ SHIP
 📱 ios:      ✅ SHIP
 🤖 android:  ⚠️ NO-SHIP (3 blocking issues)
 ────────────
 UNIFIED:     ⚠️ PARTIAL SHIP (web + ios ready, android blocked)
```

A platform that passes can ship independently with user approval. Blocked platforms route back to their Builder.

---

## Gate System

### Hard Gates (Cannot Be Bypassed)

| Gate | Rule |
|------|------|
| G1 | No Technical Planning until Discovery is complete (feature-spec.md + HANDOFF.json exist) |
| G2 | No building until the Technical Plan exists (TDD + INTERFACES.md + SCHEMA.md + TASK-GRAPH.md) |
| G3 | Red = Stop. A failing gate puts the Builder in repair mode. No new features while any gate is red. |
| G4 | Security has veto power. A BLOCKED verdict routes back to Builder. Builder must fix before advancing. |
| G5 | Verifier is proof-based. A NO-SHIP verdict routes back to Builder. The Verifier does not accept assertions. |

### CI Gates by Platform

| Platform | Gates |
|---------|-------|
| Web (Next.js) | `pnpm lint` · `pnpm typecheck` · `pnpm test` · `pnpm test:e2e` · `pnpm build` |
| Web (Express+React) | `pnpm lint` + `pnpm typecheck` + `pnpm test` in both `apps/api` and `apps/web` |
| iOS | `swiftlint --strict` · `xcodebuild build` · `xcodebuild test` |
| Android | `./gradlew lint` · `./gradlew ktlintCheck` · `./gradlew test` |
| Windows (Electron) | `npm run lint` · `npm run typecheck` · `npm test` · `npm run build` |
| macOS | `swiftlint --strict` · `xcodebuild build` · `codesign --verify` · `spctl --assess` |
| Linux (Tauri) | `cargo clippy -- -D warnings` · `cargo test` · `npm run lint` |

---

## Pipeline Memory (Self-Improving)

After every completed pipeline run, the Orchestrator captures patterns into `artifacts/PIPELINE_MEMORY.md`.

### What Gets Captured

- Gate pass/fail rates per stage
- Repair attempt counts and success rates
- Time-per-phase metrics
- Skills that were invoked and their outcomes
- Patterns that recur across 3+ runs (automatically promoted to proposals)

### Improvement Proposals

When a pattern is seen in ≥ 3 runs, it becomes a proposal (`PROP-NNN`) visible via `!pipeline-review`. Proposals can be applied to agent definitions via `!pipeline-promote PROP-NNN`.

---

## Token Budget Management

The Orchestrator can manage a credit budget across the pipeline.

| Mode | Behavior |
|------|---------|
| Off | No budget enforcement |
| Active | Tracks credits per stage, enforces compression at 90% and hard-stop at 100% |

For multi-platform forks:
- Shared stages (Discovery, Critic) consume 15% of the total budget off the top
- Remaining budget is split equally among platforms (by default)
- Custom splits: `!budget-split web 500 ios 300 android 200`
- Surplus reallocation: `!budget-rebalance [from] [to] [credits]`
