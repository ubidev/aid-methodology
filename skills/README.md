# AID Skills

Each skill is a self-contained `SKILL.md` file — a complete set of instructions for executing one phase of the AID pipeline. Load it as system context or an initial prompt for your AI agent of choice.

## The 12 Skills

### Group 1: Problem Mapping

| Skill | Phase | When to Use |
|-------|-------|-------------|
| [aid-discover](aid-discover/SKILL.md) | Phase 1: Discover | Analyzing an existing codebase to build the Knowledge Base |
| [aid-interview](aid-interview/SKILL.md) | Phase 2: Interview | Gathering requirements from a human stakeholder |

### Group 2: Planning

| Skill | Phase | When to Use |
|-------|-------|-------------|
| [aid-specify](aid-specify/SKILL.md) | Phase 3: Specify | Transforming requirements into a grounded SPEC.md |
| [aid-plan](aid-plan/SKILL.md) | Phase 4: Plan | Defining MVP scope, modules, and deliverables |
| [aid-detail](aid-detail/SKILL.md) | Phase 5: Detail | Decomposing the plan into executable tasks |

### Group 3: Implementation

| Skill | Phase | When to Use |
|-------|-------|-------------|
| [aid-implement](aid-implement/SKILL.md) | Phase 6: Implement | Executing a task with an AI coding agent |
| [aid-review](aid-review/SKILL.md) | Phase 7: Review | Spec-anchored code review with grading |
| [aid-test](aid-test/SKILL.md) | Phase 8: Test | Staging validation before deploy |

### Group 4: Production

| Skill | Phase | When to Use |
|-------|-------|-------------|
| [aid-deploy](aid-deploy/SKILL.md) | Phase 9: Deploy | Packaging and shipping a completed delivery |
| [aid-track](aid-track/SKILL.md) | Phase 10: Track | Monitoring production telemetry |

### Group 5: Maintenance

| Skill | Phase | When to Use |
|-------|-------|-------------|
| [aid-triage](aid-triage/SKILL.md) | Phase 11: Triage | Classifying production findings and routing them |
| [aid-correct](aid-correct/SKILL.md) | Phase 12: Correct | Root cause analysis and patch scope definition |

---

## How to Use a Skill

Each `SKILL.md` contains:
1. **Core Principle** — the mental model for this phase
2. **When to Use** — when this skill applies (and when it doesn't)
3. **Inputs** — what this phase requires
4. **Process** — step-by-step instructions for the agent
5. **Output** — what this phase produces
6. **Feedback Loops** — what can trigger this phase from downstream

**Basic usage pattern:**
```
Load SKILL.md as system prompt or initial context
Provide the inputs described in the skill
Run the process
Review the outputs before advancing to the next phase
```

The human approves every phase transition. The pipeline never auto-advances.

---

## Starting Point Decision

```
Is there existing code?
  YES → Start with aid-discover (Phase 1)
  NO  → Start with aid-interview (Phase 2)
```

**Brownfield path:** Discover → Interview → Specify → Plan → Detail → Implement → Review → Test → Deploy → Track → Triage

**Greenfield path:** Interview → Specify → Plan → Detail → Implement → Review → Test → Deploy → Track → Triage

---

## Incremental Adoption

You don't need all 12 phases from day one. Recommended ramp:

1. **Start:** Detail + Implement (formalize task decomposition and agent execution)
2. **Add:** Review (introduce grading and spec-anchored review)
3. **Add:** Test (staging validation gate)
4. **Add:** Plan (separate strategy from tactics)
5. **Add:** Discover (for next brownfield project)
6. **Add:** Interview (for next client engagement)
7. **Add:** Track + Triage (once shipping regularly)
8. **Add:** Correct (close the bug loop)
9. **Full pipeline:** All 12 phases with feedback loops
