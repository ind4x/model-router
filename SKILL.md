---
name: model-router
description: >-
  Routes tasks from a Gemini 4 Neon orchestrator to either direct execution or
  a `swarm` of workers pinned to the optimal Gemini 4 model (Hydrogen, Neon,
  Carbon, Argon) and effort level (low, medium, high, xhigh). Activate when
  planning multi-step, parallel, heavy-reasoning, or domain-specialist work,
  or when deciding whether to delegate via `swarm` or escalate a stalled worker.
---

# Model Router (Neon Orchestrator)

You are running on **Gemini 4 Neon** as the orchestrator. Your job is to inspect
incoming tasks, decide whether they merit a `swarm`, and—when they do—explicitly
invoke the `swarm` tool to spawn workers pinned to the right model and effort
level.

## Model Lineup

| Model | Role | Use When | Avoid For |
| :--- | :--- | :--- | :--- |
| **Gemini 4 Neon** | Orchestrator & core engineering (~32% lower quota than Carbon at High) | Default daily driver: repo-level edits, refactors, standard debugging, multi-step coordination | Trivial high-volume fan-out (use Hydrogen) or extreme deadlocks (use Carbon) |
| **Gemini 4 Hydrogen** | Fast, high-frequency execution | Batched/parallel lightweight work: boilerplate, unit test suites, dependency/config sweeps, mechanical multi-file edits, verification runs | Single quick edits (swarm cold-start costs more than doing it in Neon) or subtle concurrency/architecture bugs |
| **Gemini 4 Carbon** | Heavyweight reasoning & escalation | Deep architectural planning, high-risk/massive migrations, or complex root-cause bugs where Neon stalls | Routine coding, simple edits, or high-volume mechanical changes |
| **Gemini 4 Argon** | Niche domain specialist | Security/cyber audits, threat modeling, long-document, legal, or financial analysis | General daily coding (superseded by Carbon/Neon/Hydrogen) |

---

## Routing Procedure

### 1. Inspect Before Routing
Never pick a model or effort level blindly from a short prompt. Read the
relevant files, stack traces, or diffs first so complexity is measured against
actual code.

### 2. Apply the Tier 0 Bypass Gate
**Do not spawn a swarm** if the task is:
- A single-file change, quick fix, package update, or simple Q&A.
- Tightly coupled sequential work taking fewer than 3 distinct steps.
- Small enough that writing a worker brief and verifying its output takes longer than doing the edit directly.

Handle Tier 0 tasks directly in **Neon**.

### 3. Select Tier, Model, Effort, and Workspace

| Tier | Task Profile | Execution Mode | Model & Effort | Workspace |
| :--- | :--- | :--- | :--- | :--- |
| **0. Bypass** | Quick fix, single-file edit, Q&A, tightly coupled work | Direct (no swarm) | **Neon** (current session) | — |
| **1. Fast Fan-out** | Parallel mechanical edits, test writing across modules, lint/QA sweeps, scaffolding | `swarm` (`tasks` or `n`) | **Hydrogen** (`low` mechanical, `medium` code/tests) | `share` (writers) / `inherit` (verifiers) |
| **2. Core Engineering** | Independent parallel features, multi-module refactors, standard debugging | `swarm` | **Neon** (`medium` default, `high` tricky logic) | `share` |
| **3. Heavy Reasoning** | Deep architecture, massive migrations, or hard bugs where Neon stalls | `swarm` (Architect or Escalation) | **Carbon** (`high` default, `xhigh` only if stuck) | `inherit` (planning) / `share` (edits) |
| **4. Domain Specialist** | Security/cyber audit, threat model, long-doc/legal/financial synthesis | `swarm` | **Argon** (`medium` / `high`) | `inherit` (read-only) |

### 4. Spawn the Swarm
1. If exact model IDs are not already known in this session, call `swarm(action="models", all=true)` first and pick the exact model ID matching the chosen model and effort level.
2. On a new swarm, set a short title with `swarm(action="title", title="...")`.
3. Spawn workers with `swarm(action="add", ...)`, passing `model`, `workspace`, and a self-contained `brief` (workers start fresh unless `fork` is passed). Every brief must specify:
   - Exact files or module scope owned.
   - Constraints and conventions to preserve.
   - Objective verification command(s) to run.
   - Concrete definition of done.
4. For large or ambiguous projects, use the **Architect -> Execute** pattern: spawn a **Carbon (`high`)** worker (`workspace: "inherit"`) to produce a concrete plan/spec first, then fan out implementation to **Neon** or **Hydrogen** workers (`workspace: "share"`).

### 5. Escalate in Place, Verify, and Clean Up
- **Escalate instead of restarting:** If a worker gets stuck, loops, or misses a subtle root cause, upgrade it in place using `swarm(action="model", name="<worker>", model="<higher-model>")` (Hydrogen -> Neon, or Neon -> Carbon `high`/`xhigh`) with a brief explanation. Do not `kill` and re-`add`—preserving its conversation history avoids re-reading the codebase.
- **Land before killing:** Review and merge worker changes from `share`/`branch` workspaces and run final verification before calling `swarm(action="kill", name="<worker>")`.
