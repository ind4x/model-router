# model-router

An Antigravity agent skill for routing tasks from a **Gemini 4 Neon** orchestrator to either direct execution or a `swarm` of workers pinned to the optimal Gemini 4 model (**Hydrogen**, **Neon**, **Carbon**, **Argon**) and effort level (`low`, `medium`, `high`, `xhigh`).

---

## Why This Exists

Running a multi-agent swarm on every prompt wastes tokens and latency on cold-start overhead; running everything on a single heavy model wastes quota. `model-router` enforces a gated 3-tier coding ladder plus a domain-specialist lane:

- **Inspect before routing:** Measure complexity against actual code before picking a model or effort level.
- **Tier 0 bypass:** Keep single-file edits, quick fixes, and tightly coupled work directly in Neon with zero swarm overhead.
- **Surgical heavy reasoning:** Use Carbon for upfront architecture (Architect -> Execute) or in-place escalation (`swarm(action="model")`) when a worker stalls.
- **Isolated domain routing:** Reserve Argon strictly for security/cyber, long-document, legal, or financial analysis instead of general coding.

---

## Routing Matrix

| Tier | Task Profile | Execution Mode | Model & Effort | Workspace |
| :--- | :--- | :--- | :--- | :--- |
| **0. Bypass** | Quick fix, single-file edit, Q&A, tightly coupled work | Direct (no swarm) | **Neon** (current session) | — |
| **1. Fast Fan-out** | Parallel mechanical edits, test writing across modules, lint/QA sweeps, scaffolding | `swarm` (`tasks` or `n`) | **Hydrogen** (`low` mechanical, `medium` code/tests) | `share` (writers) / `inherit` (verifiers) |
| **2. Core Engineering** | Independent parallel features, multi-module refactors, standard debugging | `swarm` | **Neon** (`medium` default, `high` tricky logic) | `share` |
| **3. Heavy Reasoning** | Deep architecture, massive migrations, or hard bugs where Neon stalls | `swarm` (Architect or Escalation) | **Carbon** (`high` default, `xhigh` only if stuck) | `inherit` (planning) / `share` (edits) |
| **4. Domain Specialist** | Security/cyber audit, threat model, long-doc/legal/financial synthesis | `swarm` | **Argon** (`medium` / `high`) | `inherit` (read-only) |

---

## Installation

### Antigravity (Global)

```bash
git clone https://github.com/ind4x/model-router.git ~/.gemini/config/skills/model-router
```

### Antigravity (Workspace)

```bash
mkdir -p .agents/skills/model-router
cp path/to/model-router/SKILL.md .agents/skills/model-router/
```

### Optional: Reference in `GEMINI.md`

```markdown
- **Agent & Model Delegation (`model-router`)**: Run the orchestrator on **Gemini 4 Neon**. For multi-step, parallel, heavy-reasoning, or domain-specialist tasks, activate the `model-router` skill (`~/.gemini/config/skills/model-router/SKILL.md`) to route work across Hydrogen, Neon, Carbon, and Argon via `swarm`.
```

---

## License

MIT — see [LICENSE](./LICENSE).
