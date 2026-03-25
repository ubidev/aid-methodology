# Implement Task

Code + built-in review. Same universal loop as every design phase:
**code → review → fix → re-review → done when grade ≥ A-.**

Review is NOT a separate skill. It's built into implement.

## How It Works

1. **Branch per delivery.** First task creates `aid/delivery-NNN`. All tasks
   in that delivery accumulate on the same branch.
2. **Code.** Spawn coding agent with task + feature SPEC + KB context.
   Agent writes code + unit tests, runs build, runs all tests.
3. **Review.** Spawn separate reviewer agent (clean context). Grades A+ to F.
   Tags issues by source (CODE/TASK/SPEC/KB). Auto-fixes P1/P2 CODE issues.
4. **Fix.** If grade < A-, CORRECTION file produced, developer fixes,
   back to review. Circuit breaker after 3 cycles.
5. **Done.** Grade ≥ A-. Next task or `/aid-test` when deliverable complete.

## Inputs

- `task-NNN.md` — primary prompt (title, source, scope, acceptance criteria)
- Feature SPEC — Technical Specification sections
- KB docs — always coding-standards + architecture; conditionally data-model,
  api-contracts, integration-map, test-landscape, ui-architecture
- known-issues.md — issues in code the task touches

## Isolation

One branch per delivery, not per task. Tasks within a delivery are sequential.
Branch merges to main only after `/aid-test` passes.

```
git checkout -b aid/delivery-001
  → /aid-implement task-001  (code → review → fix → ✅)
  → /aid-implement task-002  (code → review → fix → ✅)
  → /aid-test delivery-001
  → PR / merge to main
```

## Impediments

When agent hits a contradiction between SPEC and codebase, it reports
an IMPEDIMENT (not a silent workaround):
- **kb-gap** → `/aid-discover`
- **architecture-conflict** → `/aid-specify`
- **missing-dependency** → `/aid-detail`
- **wrong-assumption** → update task or SPEC

## Loopbacks

- CODE issues → fix this cycle
- TASK issues → update task, re-implement
- SPEC issues → Q&A to feature STATE.md → `/aid-specify`
- KB issues → Q&A to DISCOVERY-STATE.md → `/aid-discover`

## Related Phases

- **Previous:** [Detail](../aid-detail/) — produces task files
- **Next:** [Test](../aid-test/) — validates deliverable in staging
