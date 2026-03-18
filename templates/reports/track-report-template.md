# Track Report — {Date or Deployment ID}

## Context
**Trigger:** Post-deployment | Scheduled | Alert | On-demand
**Period:** {start date} to {end date}
**Deployment:** {deployment ID, if applicable}

## Sources Checked
| Source | Type | Period | Status |
|--------|------|--------|--------|
| {error tracker} | Error logs | {date range} | ✅ Checked |
| {CI/CD} | Build/test results | {last N deployments} | ✅ Checked |
| {monitoring} | Performance metrics | {date range} | ✅ Checked |
| {issue tracker} | User-reported issues | {date range} | ✅ Checked |

## Findings

### Finding 1: {Title}
**Severity:** Critical | High | Medium | Low
**Source:** {where this was detected}
**Evidence:** {concrete data — numbers, log snippets, metric values}
**Impact:** {users affected, functionality impaired, revenue/data risk}
**Correlation:** {related events, deployments, changes — or "No clear correlation"}
**Recommendation:** Trigger triage | Monitor (below threshold) | No action

### Finding 2: {Title}
...

## Clean Areas
{Explicitly state what's working fine — especially areas touched by recent deployments.}

## Summary

| Findings | Critical | High | Medium | Low |
|----------|----------|------|--------|-----|
| {total} | {count} | {count} | {count} | {count} |

**Overall status:** Healthy | Degraded | Critical
**Action:** Trigger triage for {N} findings | No action needed
