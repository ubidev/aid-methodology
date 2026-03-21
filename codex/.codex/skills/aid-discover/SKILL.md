---
name: aid-discover
description: >
  Brownfield codebase analysis. Produces a structured Knowledge Base (knowledge/ directory).
  Processes documents sequentially with checkpoint/resume logic for resilient analysis.
  Use for new brownfield projects (full discovery) or when a downstream phase generates a
  GAP.md with discovery-needed (targeted discovery).
allowed-tools: Read, Glob, Grep, Bash, Write, Edit
---

# Brownfield Codebase Discovery

Analyze an existing codebase and produce a structured `knowledge/` directory.

> **Note:** Codex does not support subagents. All document generation is handled directly
> in this session, one document at a time. Checkpoint/resume logic skips existing files,
> so the skill is safe to re-run after interruption.

## Inputs

- Codebase access (local path, git URL, or archive)
- For targeted discovery: GAP.md or IMPEDIMENT.md that triggered re-entry

## Process

### Step 0: Check Existing KB

Scan `knowledge/` for existing files from the full expected set of 13 documents:

```
architecture.md, technology-stack.md, module-map.md, coding-standards.md, data-model.md,
api-contracts.md, integration-map.md, domain-glossary.md, test-landscape.md,
security-model.md, tech-debt.md, infrastructure.md, open-questions.md
```

Print: `[0/13] Checking existing KB...`

If ALL 13 documents exist and no `--reset` was requested, report KB is complete and skip
directly to Step 12 (README.md and INDEX.md regeneration).

**Skip any document that already exists.** Only generate missing ones.

---

### Step 1: Structure Scan → architecture.md

Print: `[1/13] Analyzing project structure...`
Skip if `knowledge/architecture.md` exists.

Detect project type, map folder structure, count by language. Identify entry points.

---

### Step 2: Architecture Analysis → architecture.md (continued)

Print: `[2/13] Analyzing architecture patterns...`
Skip if `knowledge/architecture.md` exists.

Identify patterns (MVVM, CQRS, Clean Architecture, etc.), module boundaries, data flow,
DI registration. Record all findings in `knowledge/architecture.md`.

---

### Step 3: Stack Inventory → technology-stack.md

Print: `[3/13] Cataloging technology stack...`
Skip if `knowledge/technology-stack.md` exists.

Catalog languages, frameworks, versions, package managers, runtime, build system, dev tools.
Record in `knowledge/technology-stack.md`.

---

### Step 4: Module Mapping → module-map.md

Print: `[4/13] Mapping modules...`
Skip if `knowledge/module-map.md` exists.

For each module: purpose, dependencies, size (lines/files), test coverage estimate.
Record in `knowledge/module-map.md`.

---

### Step 5: Convention Mining → coding-standards.md

Print: `[5/13] Mining coding conventions...`
Skip if `knowledge/coding-standards.md` exists.

Infer coding standards from actual code (not docs): naming, error handling, logging, config,
file organization. Mark inferred conventions with ⚠️ Inferred from code — needs confirmation.
Record in `knowledge/coding-standards.md`.

---

### Step 6: Data Model Extraction → data-model.md

Print: `[6/13] Extracting data model...`
Skip if `knowledge/data-model.md` exists.

Schema, relationships, migrations, indexes, validation rules.
Record in `knowledge/data-model.md`.

---

### Step 7: API Contracts → api-contracts.md

Print: `[7/13] Mapping API contracts...`
Skip if `knowledge/api-contracts.md` exists.

APIs exposed by this codebase (endpoints, methods, auth, request/response shapes).
APIs consumed (external services, SDKs, HTTP clients, credentials).
Record in `knowledge/api-contracts.md`.

---

### Step 8: Integration Map → integration-map.md

Print: `[8/13] Mapping integrations...`
Skip if `knowledge/integration-map.md` exists.

Message queues, event buses, caches, webhooks, third-party services, feature flags.
Record in `knowledge/integration-map.md`.

---

### Step 9: Domain Glossary → domain-glossary.md

Print: `[9/13] Building domain glossary...`
Skip if `knowledge/domain-glossary.md` exists.

Mine terminology from class names, method names, constants, and comments that encode
business concepts. Record in `knowledge/domain-glossary.md`.

---

### Step 10: Test Landscape → test-landscape.md

Print: `[10/13] Assessing test landscape...`
Skip if `knowledge/test-landscape.md` exists.

Test frameworks, test types (unit/integration/E2E), coverage tooling, CI/CD integration,
testing patterns, coverage gaps. Record in `knowledge/test-landscape.md`.

---

### Step 10b: Security Model → security-model.md

Print: `[10b/13] Evaluating security model...`
Skip if `knowledge/security-model.md` exists.

Authentication, authorization, secrets management, input validation, OWASP concerns.
Note: static analysis only — dynamic testing required for full security assessment.
Record in `knowledge/security-model.md`.

---

### Step 11: Tech Debt Audit → tech-debt.md

Print: `[11/13] Auditing tech debt...`
Skip if `knowledge/tech-debt.md` exists.

Large files, circular dependencies, missing tests, outdated packages, TODO/FIXME density,
dead code. Classify each item: Critical / High / Medium / Low.
Record in `knowledge/tech-debt.md`.

---

### Step 11b: Infrastructure Map → infrastructure.md

Print: `[11b/13] Mapping infrastructure...`
Skip if `knowledge/infrastructure.md` exists.

CI/CD pipelines, Docker/container config, IaC (Terraform, Pulumi, CDK, CloudFormation),
environments, monitoring/alerting, deployment mechanism.
Record in `knowledge/infrastructure.md`.

---

### Step 11c: Open Questions → open-questions.md

Print: `[11c/13] Identifying gaps and open questions...`
Skip if `knowledge/open-questions.md` exists.

What CANNOT be determined from code alone. Every uncertainty, assumption, and gap that needs
human input. Be comprehensive — it is better to over-document uncertainty.
Record in `knowledge/open-questions.md`.

---

### Step 12: KB Index → README.md and INDEX.md

Print: `[12/13] Generating KB index...`

**knowledge/README.md** — completeness tracking table and revision history:
```markdown
# Knowledge Base

## Completeness

| Document | Status | Notes |
|----------|--------|-------|
| architecture.md | ✅ Complete | |
| technology-stack.md | ✅ Complete | |
| module-map.md | ✅ Complete | |
| coding-standards.md | ✅ Complete | |
| data-model.md | ✅ Complete | |
| api-contracts.md | ✅ Complete | |
| integration-map.md | ✅ Complete | |
| domain-glossary.md | ✅ Complete | |
| test-landscape.md | ✅ Complete | |
| security-model.md | ✅ Complete | |
| tech-debt.md | ✅ Complete | |
| infrastructure.md | ✅ Complete | |
| open-questions.md | ✅ Complete | |

## Revision History

| Date | Documents Updated | Notes |
|------|-------------------|-------|
| {date} | All (initial discovery) | |
```

**knowledge/INDEX.md** — 2-3 line summary of every KB document for agent self-service:
```markdown
# Knowledge Base Index — {Project Name}

Use this index to find the right document before making assumptions.
If your task touches an area covered here, read the relevant document first.

| Document | Summary |
|----------|---------|
| architecture.md | {2-3 line summary} |
| technology-stack.md | {2-3 line summary} |
| ... | ... |
```

Regenerate INDEX.md on every discovery run (full or targeted).

---

### Step 13: Update Project Config Files

Print: `[13/13] Updating project config files...`

Scan the project root for `AGENTS.md` and `CLAUDE.md`. Replace any `<!-- AID:DISCOVER ... -->`
placeholders with real data from the analysis:

- **Project Overview** — project name, purpose, tech stack, target platform
- **Build & Test** — actual build, test, and lint commands (from build scripts, CI config, package manager)
- **Code Conventions** — key naming patterns, formatting rules, idioms found in code
- **Architecture** — high-level summary (layers, modules, entry points)
- **Project description** (CLAUDE.md) — project name and one-line description

Keep the `<!-- AID:DISCOVER ... -->` comment above each section so future re-discoveries can update them.
Replace only the `(pending discovery)` placeholder lines with real content.
If a field cannot be determined, leave it as `(not found — check open-questions.md)`.

Print: `Discovery complete — Knowledge Base ready`

---

## Targeted Discovery (Re-entry)

When a GAP.md or IMPEDIMENT.md triggers re-discovery of a specific area:

1. Read the GAP/IMPEDIMENT to understand exactly what's missing
2. Delete (or overwrite) only the specific KB documents that need updating
3. Re-run only the relevant steps above for the missing/outdated documents
4. Regenerate README.md and INDEX.md
5. Update `knowledge/README.md` revision history with the targeted update
6. Report completion to the calling phase

---

## Quality Checklist

- [ ] No overlap between KB documents
- [ ] Claims grounded in code evidence (file paths, line numbers)
- [ ] Inferred info marked with ⚠️
- [ ] open-questions.md captures everything needing human input
- [ ] README.md reflects completeness status and revision history
- [ ] INDEX.md generated with 2-3 line summaries of every KB document
- [ ] AGENTS.md and CLAUDE.md placeholders filled with discovered data
