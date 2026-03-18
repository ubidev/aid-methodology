# TEST-REPORT.md Template

```markdown
# Test Report — {Delivery ID}: {Name}

## Environment
- **Staging URL:** {url or environment name}
- **Branch:** {feature branch}
- **Commit:** {short hash}
- **Date:** {YYYY-MM-DD}
- **Deployed by:** {who/how}

---

## Automated Tests

### Unit Tests
- **Run:** {count} | **Pass:** {count} | **Fail:** {count} | **Skip:** {count}
- **Time:** {duration}
- **Failures:** {none, or list below}

### Integration Tests
- **Run:** {count} | **Pass:** {count} | **Fail:** {count} | **Skip:** {count}
- **Time:** {duration}
- **Failures:** {none, or list below}

### E2E Tests
- **Run:** {count} | **Pass:** {count} | **Fail:** {count} | **Skip:** {count}
- **Time:** {duration}
- **Failures:** {none, or list below}

---

## User Story Validation

### US-{id}: {Title}
| # | Acceptance Criterion | Method | Result | Notes |
|---|---------------------|--------|--------|-------|
| AC1 | {criterion} | E2E test | ✅ Pass | — |
| AC2 | {criterion} | Manual | ✅ Pass | — |
| AC3 | {criterion} | E2E test | ❌ Fail | {details} |

### US-{id}: {Title}
| # | Acceptance Criterion | Method | Result | Notes |
|---|---------------------|--------|--------|-------|
| ... | ... | ... | ... | ... |

---

## Non-Functional Validation

| Requirement | Target | Actual | Result |
|-------------|--------|--------|--------|
| Response time | <200ms | 142ms avg | ✅ Pass |
| Concurrent users | 50 | 50 stable | ✅ Pass |
| Error recovery | Graceful | 500 on edge case | ❌ Fail |

---

## Manual Testing

| Area | Tester | Result | Observations |
|------|--------|--------|-------------|
| UI review | {name} | ✅ Pass | {notes} |
| Accessibility | {name} | ⚠️ Minor | {notes} |
| Cross-browser | {name} | ✅ Pass | {notes} |

---

## Issues Found

| # | Severity | Category | Description | Reproduction | Status |
|---|----------|----------|-------------|-------------|--------|
| 1 | Critical | E2E | {description} | {steps} | Open |
| 2 | Medium | NFR | {description} | {steps} | Deferred |

---

## Coverage

- **Line coverage:** {percentage}
- **Branch coverage:** {percentage}
- **New code coverage:** {percentage}
- **Delta from previous delivery:** {+/- percentage}

---

## Verdict: {PASS | PASS WITH NOTES | FAIL}

**Summary:** {One paragraph explaining the verdict.}

**Blocking issues:** {List if FAIL, "None" if PASS.}

**Notes for deploy:** {Anything wf-deploy needs to know.}

---

## Revision History

| Rev | Date | Source | Description |
|-----|------|--------|-------------|
| 1.0 | {date} | wf-test | Initial test report |
```
