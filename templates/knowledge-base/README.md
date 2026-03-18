# Knowledge Base Templates

The Knowledge Base (`knowledge/`) is the gravitational center of every AID project. Every phase reads from it. Any phase can trigger updates to it. It outlives the project.

## Documents

| Template | Purpose | Source |
|----------|---------|--------|
| [architecture.md](architecture.md) | Patterns, layers, module boundaries, data flow | wf-discover |
| [module-map.md](module-map.md) | Every module: purpose, dependencies, size, coverage | wf-discover |
| [technology-stack.md](technology-stack.md) | Languages, frameworks, versions, runtime | wf-discover |
| [coding-standards.md](coding-standards.md) | Naming conventions, error handling, patterns | wf-discover |
| [data-model.md](data-model.md) | Schema, entities, relationships, migrations | wf-discover |
| [api-contracts.md](api-contracts.md) | APIs consumed/exposed, auth models, rate limits | wf-discover + wf-interview |
| [integration-map.md](integration-map.md) | Message queues, caches, third-party services | wf-discover + wf-interview |
| [domain-glossary.md](domain-glossary.md) | Business terms, domain language, entity definitions | wf-interview |
| [test-landscape.md](test-landscape.md) | Test frameworks, coverage, CI/CD pipeline | wf-discover |
| [security-model.md](security-model.md) | Auth/authz, secrets, compliance | wf-interview + wf-discover |
| [tech-debt.md](tech-debt.md) | Known debt items with file refs and risk ratings | wf-discover |
| [infrastructure.md](infrastructure.md) | Hosting, networking, environments, deployment | wf-discover + wf-interview |
| [open-questions.md](open-questions.md) | What code alone can't tell us | wf-discover |

## Top-Level README Template

The KB root `README.md` tracks completeness across all documents:

```markdown
# Knowledge Base — {Project Name}

**Created:** {date}
**Last Updated:** {date}
**Source:** wf-discover

## Documents

| Document | Status | Last Updated | Source |
|----------|--------|-------------|--------|
| architecture.md | ✅ Complete | {date} | wf-discover |
| module-map.md | ✅ Complete | {date} | wf-discover |
| technology-stack.md | ✅ Complete | {date} | wf-discover |
| coding-standards.md | ⚠️ Partial | {date} | wf-discover (inferred) |
| data-model.md | ✅ Complete | {date} | wf-discover |
| api-contracts.md | ❌ Missing | — | Needs interview |
| integration-map.md | ❌ Missing | — | Needs interview |
| domain-glossary.md | ❌ Missing | — | Needs interview |
| test-landscape.md | ✅ Complete | {date} | wf-discover |
| security-model.md | ❌ Missing | — | Needs interview |
| tech-debt.md | ✅ Complete | {date} | wf-discover |
| infrastructure.md | ❌ Missing | — | Needs interview |
| open-questions.md | ✅ Complete | {date} | wf-discover |

**Status key:** ✅ Complete | ⚠️ Partial | ❌ Missing
```

## Not Every Document Is Required

- **Simple CLI tool:** 4-5 documents (architecture, tech-stack, coding-standards, open-questions)
- **Greenfield project:** Start with technology-stack, coding-standards, domain-glossary — populated from interview
- **Enterprise monorepo:** All 13 documents, possibly more
- **Data pipeline:** Focus on data-model, integration-map, api-contracts, domain-glossary

The Discovery phase assesses the project and generates what's relevant. Don't create documents for things that don't exist.

## Revision Log

Every KB update should be logged in a `revision-log.md` file:

```markdown
# Revision Log

| Date | Source Phase | Document | Change Description |
|------|------------|----------|-------------------|
| {date} | wf-discover | All | Initial Knowledge Base creation |
| {date} | wf-plan (GAP-001) | module-map.md | Added 8 missing service consumers |
| {date} | wf-implement (IMP-003) | architecture.md | Corrected async model for RecordingService |
```
