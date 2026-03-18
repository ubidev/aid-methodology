# REQUIREMENTS.md Template

```markdown
# {Project Name} — Requirements

**Date:** {date}
**Interviewer:** {who conducted the interview}
**Stakeholder:** {who was interviewed}

## Client
- **Name:** {company or individual}
- **Domain:** {industry / business type}
- **Contact:** {primary contact and channel}

## Problem Statement

{In the stakeholder's words, not technical language. This should read like they wrote it.}

## Users

| Role | Description | Primary Needs |
|------|-------------|---------------|
| {role name} | {who they are, what they do} | {what they need from this system} |

## Features (Priority Ordered)

| # | Feature | Priority | Description | Notes |
|---|---------|----------|-------------|-------|
| 1 | {feature name} | Must | {what it does} | {constraints, details} |
| 2 | {feature name} | Must | {what it does} | |
| 3 | {feature name} | Should | {what it does} | |
| 4 | {feature name} | Could | {what it does} | {nice to have} |

**Priority key:**
- **Must** — The system is not viable without this.
- **Should** — Important but the system works without it.
- **Could** — Nice to have. Build if time permits.

## Technical Context

### Existing Systems
{What they already have. Current tools, platforms, databases.}

### Integrations
| System | Type | Direction | Notes |
|--------|------|-----------|-------|
| {name} | {API/DB/Queue/File} | {In/Out/Both} | {auth, rate limits, etc.} |

### Platform
- **Target:** {web, desktop, mobile, API, CLI}
- **OS:** {if relevant}
- **Browser:** {if web}

### Data
- **Volume:** {approximate data size/throughput}
- **Sensitivity:** {public, internal, PII, regulated}
- **Existing data:** {migration needed? from where?}

## Constraints

### Timeline
{Hard deadline? Preferred pace? Phased delivery acceptable?}

### Budget
{Fixed price, hourly, range. What's the expected investment?}

### Team
{Who's available on the client side? Technical skills?}

### Compliance
{HIPAA, GDPR, SOC2, PCI, industry-specific regulations.}

## Assumptions

{Things we assumed during the interview that need explicit verification.
Each assumption should be stated clearly so the stakeholder can confirm or deny.}

- {Assumption 1 — stated because of {reason}}
- {Assumption 2 — inferred from {answer}}

## Out of Scope

{Explicitly excluded from this project. Prevents scope creep.
Be specific — "mobile app" not "anything else."}

- {Item 1}
- {Item 2}

## Interview Notes

{Optional. Raw notes, quotes, context that doesn't fit above but may be useful later.}

## Revision History

| Rev | Date | Source | Description |
|-----|------|--------|-------------|
| 1.0 | {date} | wf-interview | Initial requirements from stakeholder interview |
```
