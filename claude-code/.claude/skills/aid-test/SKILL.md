---
name: aid-test
description: >
  Staging validation — E2E, integration, and acceptance testing per deliverable.
  The gate between implement and deploy. Use when ALL tasks in a deliverable
  have been implemented (grade A- or above from built-in review).
allowed-tools: Read, Glob, Grep, Bash
context: fork
agent: critic
argument-hint: "delivery-001 (required)  [work-001 if multiple works]"
---

# Staging Validation

Deploy deliverable to staging. Run integration + E2E tests. Validate acceptance
criteria. Produce TEST-REPORT.md.

## Core Principle

Implement catches what code **looks like** (built-in review grades it).
Test catches what code **does**. Unit tests run during implement.
Integration and E2E run here, in staging, where real services connect
and real data flows.

## Workspace

```
.aid/
  knowledge/                ← shared KB (read)
  work-NNN-{name}/
    PLAN.md                 ← deliverable definitions
    tasks/
      task-NNN.md           ← acceptance criteria per task
    features/
      feature-NNN-{name}/
        SPEC.md             ← expected behavior, NFRs
```

## Arguments

| Argument | Effect |
|----------|--------|
| `delivery-NNN` | Required. Which deliverable to test. |
| `work-NNN` | Required if multiple works exist. |

## Pre-flight

### Check 1: Locate Work

1. If work arg provided → use that work directory
2. If single work exists → auto-select
3. If multiple works → list them, ask user to choose

### Check 2: Verify Deliverable

1. Read `PLAN.md` → find `delivery-NNN`
2. Deliverable not found → **STOP.** List available deliverables.
3. Identify which features are in this deliverable
4. Identify which tasks belong to those features

### Check 3: Verify All Tasks Implemented

For each task in this deliverable:
- Must have passed `/aid-implement` with grade A- or above (built-in review)
- Any task not implemented or below A- → **STOP.** List incomplete tasks.

### Check 4: Verify On Delivery Branch

- Must be on `aid/delivery-NNN` branch (matching the deliverable being tested)
- If not → `git checkout aid/delivery-NNN`
- If branch doesn't exist → **STOP.** Tasks haven't been implemented yet.

### Check 5: Verify No Open Impediments

- Check for IMPEDIMENT-task-NNN.md files in the work directory
- If any exist and relate to this deliverable → **STOP.** Resolve first.

---

## Inputs

- **PLAN.md** — deliverable definition, success criteria
- **Feature SPECs** — acceptance criteria, NFRs for features in this deliverable
- **Task files** — `tasks/task-NNN.md` for tasks in this deliverable
- **KB via INDEX.md** — Read `.aid/knowledge/INDEX.md`, use summaries to pull
  relevant docs (typically test-landscape, infrastructure — but let the INDEX guide you)

---

## Process

### Step 1: Deploy to Staging

Deploy the deliverable's code to staging environment.

Verify:
- [ ] Application starts successfully
- [ ] Health checks pass
- [ ] Database migrations applied (if any)
- [ ] External services connected or properly mocked

**If deployment fails → STOP.** Document failure, return to `/aid-implement`.

### Step 2: Run Automated Test Suite

Run the FULL test battery in staging:

| Category | What | Source |
|----------|------|--------|
| Unit (regression) | All existing unit tests | Catch regressions in staging env |
| Integration | Service-to-service, DB queries | test-landscape.md patterns |
| E2E | User flows end-to-end | Feature SPEC acceptance criteria |

Record per category:
- Total / Passed / Failed / Skipped
- Execution time
- Failure details with reproduction steps

### Step 3: Validate Acceptance Criteria

For each task in this deliverable, check its acceptance criteria in staging:

| Task | Criterion | Status | Method |
|------|-----------|--------|--------|
| task-001 | {criterion} | ✅ Pass / ❌ Fail | Automated / Manual |

**Every criterion must be validated.** If a criterion can't be tested in staging,
document why and note what would be needed.

### Step 4: Non-Functional Validation

If feature SPECs define NFRs (performance targets, concurrency, data volume):

| NFR | Target | Actual | Status |
|-----|--------|--------|--------|
| Response time | < 200ms | 150ms | ✅ Pass |
| Concurrent users | 100 | 95 | ⚠️ Close |

**Only test NFRs that are specified.** Don't invent performance requirements.

### Step 5: Manual Testing (if applicable)

When automated tests can't cover everything:
- Visual/UI review (layout, responsiveness, design system compliance)
- Workflow coherence (multi-step flows make sense end-to-end)
- Accessibility (keyboard nav, screen reader, contrast)
- Cross-platform/browser (if specified in requirements)

Document observations — not just pass/fail.

### Step 6: Produce TEST-REPORT.md

Write to `.aid/{work}/TEST-REPORT-{delivery-NNN}.md`:

```markdown
# Test Report — delivery-NNN: {Name}

**Work:** work-NNN-{name}
**Date:** {YYYY-MM-DD}
**Environment:** {staging details}

## Deployment
- Status: ✅ Success / ❌ Failed
- Notes: {any deployment observations}

## Automated Tests

| Category | Total | Passed | Failed | Skipped | Time |
|----------|-------|--------|--------|---------|------|
| Unit | | | | | |
| Integration | | | | | |
| E2E | | | | | |

{If failures: details with reproduction steps}

## Acceptance Criteria

| Task | Criterion | Status | Method | Notes |
|------|-----------|--------|--------|-------|
| task-001 | {criterion} | ✅/❌ | Auto/Manual | |

## Non-Functional (if applicable)

| NFR | Target | Actual | Status |
|-----|--------|--------|--------|

## Manual Testing (if applicable)

{Observations}

## Verdict: {PASS / PASS WITH NOTES / FAIL}

{Justification}
```

---

## Verdicts

| Verdict | Meaning | Next Step |
|---------|---------|-----------|
| **PASS** | All green, no issues | `/aid-deploy` |
| **PASS WITH NOTES** | All critical pass, minor observations | `/aid-deploy` (notes tracked) |
| **FAIL** | Critical or high-priority failures | CORRECTION.md → `/aid-implement` → re-test |

### On FAIL

Produce `.aid/{work}/CORRECTION.md` with:
- Failed test details
- Failed acceptance criteria
- Reproduction steps
- Severity classification

Route back to `/aid-implement` → `/aid-test` cycle.

---

## Feedback Loops

- **→ Implement:** Test failure from code bug → CORRECTION.md → `/aid-implement` → re-test
- **→ Specify:** Spec gap discovered in staging → Q&A to feature STATE.md → re-specify

---

## Output

`.aid/{work}/TEST-REPORT-{delivery-NNN}.md` with clear verdict.

## Quality Checklist

- [ ] Staging deployment successful
- [ ] Full automated suite ran (unit + integration + E2E)
- [ ] Every acceptance criterion from every task validated
- [ ] NFRs tested (if specified in SPECs)
- [ ] Manual testing completed (if applicable)
- [ ] All failures documented with reproduction steps
- [ ] TEST-REPORT.md produced with clear verdict
- [ ] CORRECTION.md produced on FAIL
