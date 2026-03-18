# Open Questions

> **Source:** wf-discover (Phase 1)
> **Status:** Active — feeds directly into wf-interview
> **Last Updated:** {date}

> Questions that code analysis alone could not answer. These drive the interview phase. Every item here is something an AI agent would have to guess about — guessing creates specs that don't match reality.

---

## Open Questions

| # | Question | Context | Impact | Status |
|---|----------|---------|--------|--------|
| OQ-001 | {question} | {why we need to know this} | {what it blocks or affects} | {Open / Answered / No Action} |

---

## Detailed Items

### OQ-001: {Question}

**Why we're asking:**
{Specific evidence from the codebase that raises this question. Don't say "unknown" — say "SearchService has three implementations with different timeout values (5s, 15s, 30s) with no comments. We don't know which is correct."}

**What we found in code:**
- {concrete evidence: file path, pattern, or behavior observed}

**Impact if wrong:**
{What happens if we spec or implement against the wrong assumption}

**Likely answer options:**
- Option A: {description} — {implication}
- Option B: {description} — {implication}

**Who can answer:** {architect / product owner / technical lead / the person who wrote {module}}

**Resolution:** {leave blank until answered}

---

### OQ-002: {Question}

**Why we're asking:** {evidence from code}

**What we found:** {specifics}

**Impact if wrong:** {consequence}

**Who can answer:** {role}

**Resolution:** {leave blank}

---

## Answered Questions

> Moved here once resolved. Keep as record of what was discovered and when.

| # | Question | Answer | Source | Date |
|---|----------|--------|--------|------|
| OQ-00n | {question} | {answer} | {wf-interview Q7 / stakeholder email / code comment} | {date} |

---

## Assumptions Made Without Answers

> Items where we made an assumption to proceed. These are risks — flag for early interview confirmation.

| Assumption | What We Assumed | Risk if Wrong |
|------------|-----------------|---------------|
| {e.g., Auth model} | {JWT, 24h expiry} | {Security model and session handling will need rework} |
| {e.g., Multi-tenancy} | {Single tenant} | {Data isolation, schema, and API design may all be wrong} |

---

## Revision History

| Rev | Date | Source | Description |
|-----|------|--------|-------------|
| 1.0 | {date} | wf-discover | Initial open questions from code analysis |
