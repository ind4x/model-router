# model-router

`model-router` is a production-grade agent skill and routing specification for orchestrating **Gemini 4** multi-agent swarms from a **Gemini 4 Neon** coordinator.

Running a multi-agent swarm on every prompt wastes tokens and latency on cold-start overhead, while running every task on the heaviest reasoning model burns quota unnecessarily. `model-router` enforces a gated three-tier coding ladder (**Hydrogen**, **Neon**, **Carbon**) plus an isolated domain-specialist lane (**Argon**), pairing each task with the right model, effort level (`low`, `medium`, `high`, `xhigh`), and workspace isolation mode.

---

## Core Features

- **Inspect-First Triage:** Measures actual codebase complexity before selecting a model or effort level instead of guessing from a short prompt.
- **Tier 0 Bypass Gate:** Keeps single-file edits, quick fixes, package updates, and tightly coupled sequential work directly in Neon with zero swarm cold-start overhead.
- **Three-Tier Coding Ladder:** Separates high-frequency parallel fan-out (**Hydrogen**), core multi-module engineering (**Neon**), and heavyweight architectural reasoning (**Carbon**).
- **Isolated Domain Specialist Lane:** Restricts **Argon** strictly to security/cyber audits, threat modeling, and long-document, legal, or financial synthesis so it never adds routing noise to daily coding.
- **Architect -> Execute Pattern:** Spawns a read-only **Carbon (`high`)** planner to author a concrete specification on massive migrations before fanning out implementation to **Neon** or **Hydrogen** workers.
- **In-Place Escalation:** Upgrades stalled workers in place with `swarm(action="model")` instead of killing and restarting them, preserving full conversation history and codebase context.

---

## Routing Architecture

The orchestrator inspects the target codebase first, then either bypasses the swarm completely or dispatches workers into the matching tier:

```text
                [ Incoming Task / User Prompt ]
                               │
                               ▼
              ┌─────────────────────────────────┐
              │   Gemini 4 Neon (Orchestrator)  │  <- Inspect files, diffs, stack traces
              └────────────────┬────────────────┘
                               │
                     ┌─────────┴─────────┐
                     ▼                   ▼
          [ Tier 0: Bypass ]      [ Swarm Dispatch ]
          Direct in Neon          (Explicit model + effort)
          (Single-file / Q&A)            │
                        ┌────────────────┼────────────────┬────────────────┐
                        ▼                ▼                ▼                ▼
               ┌────────────────┐ ┌─────────────┐ ┌──────────────┐ ┌──────────────┐
               │ Tier 1: Fan-out│ │Tier 2: Core │ │Tier 3: Heavy │ │Tier 4: Niche │
               │    Hydrogen    │ │    Neon     │ │    Carbon    │ │    Argon     │
               │  low / medium  │ │medium / high│ │ high / xhigh │ │medium / high │
               └───────┬────────┘ └──────┬──────┘ └──────┬───────┘ └──────┬───────┘
                       └─────────────────┴───────────────┴────────────────┘
                                                 │
                                                 ▼
                             [ In-Place Escalation (if stalled) ]
                             [ Review, Merge, Verify, Clean Up  ]
```

---

## Model Lineup

Each Gemini 4 model fills a distinct operational role in the routing policy:

| Model | Role | Use When | Avoid For |
| :--- | :--- | :--- | :--- |
| **Gemini 4 Neon** | Orchestrator and core engineering (~32% lower quota than Carbon at High) | Default daily driver: repo-level edits, refactors, standard debugging, multi-step coordination | Trivial high-volume fan-out (use Hydrogen) or extreme deadlocks (use Carbon) |
| **Gemini 4 Hydrogen** | Fast, high-frequency execution | Batched or parallel lightweight work: boilerplate, unit test suites, dependency/config sweeps, mechanical multi-file edits, verification runs | Single quick edits (swarm cold-start costs more than Neon) or subtle concurrency/architecture bugs |
| **Gemini 4 Carbon** | Heavyweight reasoning and escalation | Deep architectural planning, high-risk migrations, or complex root-cause bugs where Neon stalls | Routine coding, simple edits, or high-volume mechanical changes |
| **Gemini 4 Argon** | Niche domain specialist | Security/cyber audits, threat modeling, long-document, legal, or financial analysis | General daily coding (superseded by Carbon, Neon, and Hydrogen) |

---

## Routing Matrix

Use this matrix after inspecting the target files to pick the execution mode, model, effort level, and workspace isolation:

| Tier | Task Profile | Execution Mode | Model & Effort | Workspace |
| :--- | :--- | :--- | :--- | :--- |
| **0. Bypass** | Quick fix, single-file edit, Q&A, tightly coupled work | Direct (no swarm) | **Neon** (current session) | None |
| **1. Fast Fan-out** | Parallel mechanical edits, test writing across modules, lint/QA sweeps, scaffolding | `swarm` (`tasks` or `n`) | **Hydrogen** (`low` mechanical, `medium` code/tests) | `share` (writers) / `inherit` (verifiers) |
| **2. Core Engineering** | Independent parallel features, multi-module refactors, standard debugging | `swarm` | **Neon** (`medium` default, `high` tricky logic) | `share` |
| **3. Heavy Reasoning** | Deep architecture, massive migrations, or hard bugs where Neon stalls | `swarm` (Architect or Escalation) | **Carbon** (`high` default, `xhigh` only if stuck) | `inherit` (planning) / `share` (edits) |
| **4. Domain Specialist** | Security/cyber audit, threat model, long-document, legal, or financial synthesis | `swarm` | **Argon** (`medium` / `high`) | `inherit` (read-only) |

---

## 5-Stage Routing Workflow

When triaging and delegating a task, execute in five linear passes:

1. **Inspect Before Routing (Stage 1):** Read the relevant files, stack traces, or diffs before choosing a tier or effort level.
2. **Apply the Tier 0 Bypass Gate (Stage 2):** Execute directly in Neon for single-file changes, quick fixes, simple Q&A, or tightly coupled work under three steps.
3. **Select Tier, Model, and Workspace (Stage 3):** Match the task to Tier 1 through Tier 4, verify exact model IDs with `swarm(action="models", all=true)` if needed, and assign `share` for concurrent writers or `inherit` for read-only analysts and verifiers.
4. **Brief and Spawn (Stage 4):** Set a swarm title and write self-contained worker briefs specifying owned files, constraints, objective verification commands, and definition of done. On large ambiguous projects, run a **Carbon (`high`)** architect first before fanning out implementation.
5. **Escalate, Verify, and Clean Up (Stage 5):** Upgrade stalled workers in place via `swarm(action="model")` rather than restarting them. Review and land all changes and run verification before calling `swarm(action="kill")`.

---

## Installation

### Antigravity (Global)

Clone the repository directly into Antigravity's global skill discovery directory:

```bash
git clone https://github.com/ind4x/model-router.git ~/.gemini/config/skills/model-router
```

### Antigravity (Workspace)

Copy `SKILL.md` into your project's `.agents/skills/` directory to scope the skill to a single repository:

```bash
mkdir -p .agents/skills/model-router
cp path/to/model-router/SKILL.md .agents/skills/model-router/
```

### Project and Global Rules (`GEMINI.md`)

Reference the skill in your `GEMINI.md` so the orchestrator activates it automatically on complex tasks:

```markdown
- **Agent & Model Delegation (`model-router`)**: The main agent and orchestrator runs on **Gemini 4 Neon**. For multi-step, parallel, heavy-reasoning, or domain-specialist tasks, activate the `model-router` skill (`~/.gemini/config/skills/model-router/SKILL.md`) to route work across **Hydrogen**, **Neon**, **Carbon**, and **Argon** via `swarm` with explicit model/effort pinning, while bypassing `swarm` for small single-file tasks.
```

---

## Repository Structure

```text
model-router/
├── SKILL.md      # Main Antigravity skill definition with YAML frontmatter and routing protocol
├── README.md     # Architecture overview, routing matrix, workflow, and installation guide
├── LICENSE       # MIT License
└── .gitignore    # Standard editor and OS ignore rules
```

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Author

- **Author:** Youssef Fellah ([ind4x_](https://github.com/ind4x))
