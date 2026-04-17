# Commands Reference

> All commands available within the CodeSleuth AI pipeline.

Commands can be issued at any time during a pipeline session. The Orchestrator intercepts them and routes execution to the appropriate internal system.

---

## Session & State Commands

### `!state`

**What it does:** Displays the current pipeline stage and all active artifact names.

**When to use:** When you want to know where the pipeline is, what has been produced, and what comes next.

**Expected output:**

```
📊 Pipeline State
─────────────────
Stage:     Building (Agent 3)
Platform:  🌐 web · nextjs
Artifacts: feature-spec.md ✅ | HANDOFF.json v2 ✅ | TDD.md ✅ | INTERFACES.md ✅
           SCHEMA.md ✅ | TASK-GRAPH.md (12/28 done)
Blockers:  None
```

---

### `!status`

**What it does:** Shows the current task and gate status within the active agent.

**When to use:** During a build to see which task is executing and whether gates are green.

---

### `!reset`

**What it does:** Wipes all artifacts and restarts the pipeline from Agent 1 (Discovery).

**When to use:** When the project has changed fundamentally and you want a clean start.

**Guardrails:** Irreversible — all compiled spec and plan artifacts are cleared.

---

## Discovery Commands (Agent 1)

### `!braindump`

**What it does:** Switches Discovery into Context Ingestion Mode (Phase 0). The agent accepts freeform, unstructured input without asking questions. When you type `!done`, it processes the dump and summarizes themes, platform signals, constraints, conflicts, and design direction signals.

**When to use:** When you have messy notes, half-formed ideas, competitor links, or existing docs you want to paste in before structured discovery begins.

**Input:** Paste anything — the agent will not interrupt.

**Output after `!done`:**

```
📋 Context Captured
- Key Themes: [list]
- Platform Signals: [list]
- Constraints: [list]
- Conflicts to Resolve: [list]
- References: [list]
- Design Direction Signals: [any brand/aesthetic cues detected]
```

---

### `!compile`

**What it does:** Triggers Agent 1 to compile the full Feature Specification. Writes `artifacts/discovery/feature-spec.md` and `artifacts/discovery/HANDOFF.json`. Then runs the Concept Validation Gate before advancing to Agent 2.

**When to use:** When you've answered enough discovery questions and are ready to lock in the spec.

**Prerequisites:** Sufficient discovery context across Phases 2–15. The agent will warn if critical sections are incomplete.

**Concept Validation Gate verdicts:**

| Verdict | Meaning |
|---------|---------|
| ✅ PROCEED | Spec is solid — advances to Planner |
| ⚠️ REVISE | One or more dimensions need strengthening before planning |
| ❌ RETHINK | Core concept or scope has fundamental issues |

**Guardrails:** Do NOT compile before completing at least Phases 2–6. An incomplete spec will produce a weak TDD and poor implementation tasks.

---

### `!platforms`

**What it does:** Displays the currently declared platform scope and subtype as captured in the spec state.

---

## Planning Commands (Agent 2)

### `!plan`

**What it does:** Invokes Agent 2 to generate the full Technical Plan Pack (TECH_PLAN_PACK). Produces the TDD, INTERFACES.md, SCHEMA.md, TASK-GRAPH.md, FEATURES.md, and updates HANDOFF.json to v2.

**When to use:** After `!compile` produces a PROCEED verdict.

**Prerequisites:** `feature-spec.md` and `HANDOFF.json` v1 must exist with a `concept_validation: "proceed"` field.

**Gate (G1):** This command is blocked if FEATURE_SPEC does not exist.

---

## Build Commands (Agent 3)

### `!build`

**What it does:** Invokes Agent 3 to begin implementing the plan. The Builder runs continuously — executing tasks in TASK-GRAPH dependency order, running gates after each task, and only stopping when: all tasks are DONE (→ SHIP DECLARATION), a sudo password is required, a critical blocker persists after 5 repair attempts, or a required secret is missing.

**When to use:** After the Technical Plan Pack is complete.

**Prerequisites:** TECH_PLAN_PACK must exist (TDD + INTERFACES.md + SCHEMA.md + TASK-GRAPH.md).

**Gate (G2):** This command is blocked if TECH_PLAN_PACK does not exist.

**Important:** Do not interrupt the Builder mid-task. It does not stop between tasks to ask permission.

---

### `!retry`

**What it does:** Instructs the Builder to retry the current task from scratch if it is in a BLOCKED state.

**When to use:** When the Builder has reported BLOCKED after 5 repair attempts and you believe the root cause has been resolved (e.g., you provided a missing environment variable or fixed an external dependency).

---

### `!skip`

**What it does:** Skips the current blocked task and moves to the next eligible task. The skipped task is marked as `SKIPPED` in TASK-GRAPH.md.

**When to use:** With caution. Only skip if you are certain the blocked task's output is not required by downstream tasks. Skipping a foundational task can cascade into failures.

**Side effect:** The skipped task remains in TASK-GRAPH.md. It can be re-attempted later.

---

## Security Commands (Agent 4)

### `!security`

**What it does:** Invokes Agent 4 to run the full 20-domain security review across all in-scope platforms. Produces a Security Review Report with Critical / High / Medium findings and a Go/No-Go launch gate verdict.

**When to use:** After the Builder has declared SHIP or when you need an adversarial security check.

**Prerequisites:** TECH_PLAN_PACK and at least one completed build (artifacts/build/CHECKPOINT.md or equivalent evidence).

**Output: Three verdicts:**

| Verdict | Meaning |
|---------|---------|
| ✅ APPROVED | All critical checks pass. Advances to Verifier. |
| ⚠️ CONDITIONAL | No critical blockers, but conditions must be met before ship. |
| ❌ BLOCKED | One or more critical findings open. Pipeline does not advance. Routes back to Builder. |

### `!security-review`

Same as `!security` — full 20-domain review.

### `!security-review --platform=[name]`

Runs the security review for a single platform only (e.g., `!security-review --platform=ios`).

### `!security-review --domain=[N]`

Reviews only the specified domain number (1–20). Useful for re-checking a specific area after a fix.

### `!launch-gate`

Runs only the Go/No-Go launch gate checklist without a full review. Useful for confirming that all previously identified findings have been resolved.

### `!threat-model`

Generates a threat model for the current spec. Uses STRIDE analysis to enumerate attack surfaces, threat actors, and mitigations per platform.

### `!dep-audit`

Triggers the automated dependency audit using the `dependency-auditor` skill. Scans package managers across all platforms and produces a CVE findings table with severity ratings and upgrade paths.

---

## Verification Commands (Agent 5)

### `!verify`

**What it does:** Invokes Agent 5 to run the full 12-phase verification pipeline across all in-scope platforms. Verifies against the original TDD — not the Builder's summary. Produces a per-platform SHIP or NO-SHIP verdict.

**When to use:** After Security has APPROVED (or CONDITIONALLY APPROVED).

**Prerequisites:** Security review must have been completed.

**Output:**

| Verdict | Meaning |
|---------|---------|
| ✅ SHIP | All phases pass on this platform. Advances to Critic. |
| ⚠️ NO-SHIP | One or more phases fail. Routes back to Builder for remediation. |

**Guardrail:** A NO-SHIP on any platform = NO-SHIP for cross-platform releases (unless partial ship is explicitly approved).

---

## Critic Commands (Agent 6)

### `!critic`

**What it does:** Invokes Agent 6 to run a full product + engineering survival review. Covers all 12 review categories (A–L), all in-scope platforms, and produces CRITICISM.md.

**When to use:** After Verification has passed (at least one SHIP verdict).

**Prerequisites:** At least a Verification report and access to the codebase.

### `!criticize`

Same as `!critic` — full review with output to both chat and CRITICISM.md.

### `!criticize --file-only`

Produces the full review in CRITICISM.md with minimal chat output.

### `!criticize --verbose`

Maximum detail in both chat and CRITICISM.md.

### `!criticize --platform=[name]`

Focuses the critique on a single platform (e.g., `!criticize --platform=android`).

### `!criticize --focus=[area]`

Emphasizes a specific review area. Valid values: `security`, `product`, `performance`, `monetization`, `design`.

---

## Pipeline Intelligence Commands (Orchestrator)

### `!pipeline`

**What it does:** Displays the current overall pipeline status — which agent is active, which stages are complete, and what gates have been run.

---

### `!pipeline-health`

**What it does:** Displays pipeline memory health statistics from `artifacts/PIPELINE_MEMORY.md`.

**Output:**

```
📊 Pipeline Memory Health
─────────────────────────────
 Runs: [N] total ([first date] — [last date])
 Ship rate: [N]/[N] ([X]%)
 Patterns: [N] positive | [N] negative | [N] neutral
 Proposals: [N] pending | [N] approved | [N] applied | [N] rejected
 Regressions: [N] active alerts
 Credits: avg [X] | min [X] | max [X]
 Memory file: artifacts/PIPELINE_MEMORY.md
```

**When to use:** To understand how the pipeline has been performing across multiple runs and whether improvement proposals are waiting.

---

### `!pipeline-review`

**What it does:** Scans PIPELINE_MEMORY.md for patterns that have appeared in ≥ 3 runs and displays them as actionable improvement proposals.

**Output per proposal:**

```
PROP-001: [title]
  Category:  [category]
  Risk:      Low | Medium | High
  Evidence:  [N] runs ([timestamps])
  Target:    [file path to modify]
  Change:    [description of improvement]
  Action:    Approve with: python3 scripts/pipeline-promote.py PROP-001 --approve
```

---

### `!pipeline-promote [PROP-NNN]`

**What it does:** Displays the proposal details and diff preview, then instructs you to run the promotion script directly. The Orchestrator does **not** auto-run the promotion — this is a user-executed safety gate.

**Workflow:**

1. Orchestrator shows proposal details and diff preview
2. Orchestrator provides the command: `python3 scripts/pipeline-promote.py PROP-NNN --confirm`
3. You run the command to apply the change
4. Orchestrator confirms the result

---

### `!pipeline-extract [description]`

**What it does:** Extracts a recurring pattern from pipeline runs into a new community skill. Prompts for skill name, description, target agents, capabilities, and domains, then runs the extraction script.

**Output:** Creates the community skill directory structure and SKILL.md template.

**Follow-up:** Edit the SKILL.md to add the full protocol, then run `bash scripts/build-capability-registry.sh` to register the skill.

---

### `!skills`

**What it does:** Displays the current registry summary — total skill count, per-agent breakdown, and top-scored skills for the current pipeline context.

---

## Multi-Platform Commands

### `!fork`

**What it does:** Confirms multi-platform fork mode. Triggers the platform scaffolding script and begins Discovery with multi-platform awareness. Planning and Builder then run once per platform in sequence with isolated state.

**Prerequisites:** The Orchestrator must have detected 2+ platforms (shown in the session banner).

**Runs:** `python3 scripts/scaffold-platform-state.py --platforms [platforms]`

---

### `!nofork`

**What it does:** Overrides fork detection and runs all platforms through a single sequential pipeline (legacy behavior). All platforms share one set of state files.

**When to use:** When you have multiple declared platforms but want a simpler, unified plan (e.g., a web + mobile app where the mobile version is minor).

---

### `!pipelines`

**What it does:** Displays the current status of all forked pipelines side by side.

**Output:**

```
🔀 Pipeline Status (Multi-Platform)
─────────────────────────────────────────────
 Platform    Stage         Tasks      Credits    Blockers
 ─────────────────────────────────────────────
 🌐 web      building     12/28      45.20      —
 📱 ios      planning      0/0        8.30      —
 🤖 android  planning      0/0        8.10      —
 ─────────────────────────────────────────────
 🔗 shared   discovery    complete    12.00      —
 📊 total    —            12/28      73.60      0
```

`!pipelines [platform]` shows detailed status for a specific platform.

---

## Budget Commands

### `!budget`

**What it does:** Displays the current token/credit budget status — total budget, credits consumed per stage, current tier, and remaining credits.

---

### `!budget-split [platform] [credits] ...`

**What it does:** Assigns a custom credit budget to each platform in a multi-pipeline fork.

**Example:** `!budget-split web 500 ios 300 android 200`

---

### `!budget-rebalance [from] [to] [credits]`

**What it does:** Moves unused credits from one platform's budget to another after that platform completes under budget.

**Example:** `!budget-rebalance ios web 150`
