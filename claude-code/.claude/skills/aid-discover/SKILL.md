---
name: aid-discover
description: >
  Brownfield codebase analysis. Produces a structured Knowledge Base (knowledge/ directory).
  Orchestrates 5 specialized discovery subagents for resilient, resumable analysis.
  Use for new brownfield projects (full discovery) or when a downstream phase generates a
  GAP.md with discovery-needed (targeted discovery).
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, Agent
---

# Brownfield Codebase Discovery

Analyze an existing codebase and produce a structured `knowledge/` directory by orchestrating
5 specialized discovery subagents. Each subagent owns a focused area of analysis, keeping
individual context windows manageable even on large codebases (200K+ lines).

## ⚠️ Pre-flight Check

**Before starting discovery, verify you are NOT in Plan Mode.**

Plan Mode restricts all operations to read-only — subagents will NOT be able to write KB files.

**How to check:** Look at the permission indicator in your Claude Code interface (bottom of screen).
- ✅ `Default` or `Auto-accept edits` → Proceed with discovery.
- ❌ `Plan mode` → **STOP.** Tell the user: "Discovery needs to write files. Please press `Shift+Tab` to switch out of Plan Mode, then re-run `/aid-discover`."

**Do NOT proceed with discovery while in Plan Mode.** The subagents will analyze the codebase but silently fail to write any files.

## Inputs

- Codebase access (local path, git URL, or archive)
- For targeted discovery: GAP.md or IMPEDIMENT.md that triggered re-entry

## Orchestration Flow

### Step 0: Check Existing KB

Scan `knowledge/` for existing files. Build a list of what exists vs. what's missing from the
full set of 13 expected documents:

```
architecture.md, technology-stack.md, module-map.md, coding-standards.md, data-model.md,
api-contracts.md, integration-map.md, domain-glossary.md, test-landscape.md,
security-model.md, tech-debt.md, infrastructure.md, open-questions.md
```

Print: `[0/13] Checking existing KB...`

If ALL 13 documents exist and no `--reset` was requested, report KB is complete and skip
directly to Step 6 (README.md and INDEX.md regeneration).

---

### Step 1: Architecture Analysis

Dispatch **discovery-architect** to produce `architecture.md` and `technology-stack.md`.
Only dispatch if either file is missing.

Print: `[1/5] Dispatching architecture analysis...`

Prompt to pass to the subagent:
> Analyze this codebase and produce knowledge/architecture.md and knowledge/technology-stack.md.
> Cover: project type, folder structure, architectural patterns, module boundaries, data flow,
> DI registration, entry points, tech stack (languages, frameworks, versions, package managers,
> runtime, build tools, dev tooling). Write only to the knowledge/ directory.

Wait for completion. Verify both files exist before proceeding.

---

### Step 2: Module and Convention Analysis

Dispatch **discovery-analyst** to produce `module-map.md`, `coding-standards.md`, and `data-model.md`.
Only dispatch if any of those files are missing.

Print: `[2/5] Dispatching module and convention analysis...`

Prompt to pass to the subagent:
> Analyze this codebase and produce knowledge/module-map.md, knowledge/coding-standards.md,
> and knowledge/data-model.md. Map every module (purpose, size, dependencies, test coverage).
> Mine coding conventions from actual code — naming, error handling, logging, config, file
> organization. Extract data models: schemas, relationships, migrations, indexes, validation.
> Write only to the knowledge/ directory.

Wait for completion. Verify files before proceeding.

---

### Step 3: Integration Mapping

Dispatch **discovery-integrator** to produce `api-contracts.md`, `integration-map.md`, and `domain-glossary.md`.
Only dispatch if any of those files are missing.

Print: `[3/5] Dispatching integration mapping...`

Prompt to pass to the subagent:
> Analyze this codebase and produce knowledge/api-contracts.md, knowledge/integration-map.md,
> and knowledge/domain-glossary.md. Map APIs exposed and consumed, message queues, caches,
> webhooks, and third-party services. Build a domain glossary from class names, method names,
> constants, and comments that encode business concepts. Write only to the knowledge/ directory.

Wait for completion. Verify files before proceeding.

---

### Step 4: Quality Assessment

Dispatch **discovery-quality** to produce `test-landscape.md`, `security-model.md`, and `tech-debt.md`.
Only dispatch if any of those files are missing.

Print: `[4/5] Dispatching quality assessment...`

Prompt to pass to the subagent:
> Analyze this codebase and produce knowledge/test-landscape.md, knowledge/security-model.md,
> and knowledge/tech-debt.md. Assess test frameworks, test types, coverage, CI/CD integration.
> Evaluate security: auth, authorization, secrets management, OWASP concerns. Audit tech debt:
> large files, TODO/FIXME density, missing tests, outdated packages, dead code. Classify all
> debt items with risk ratings (Critical/High/Medium/Low). Write only to the knowledge/ directory.

Wait for completion. Verify files before proceeding.

---

### Step 5: Infrastructure and Gap Analysis

Dispatch **discovery-scout** to produce `infrastructure.md` and `open-questions.md`.
Only dispatch if either file is missing.

Print: `[5/5] Dispatching infrastructure and gap analysis...`

Prompt to pass to the subagent:
> Analyze this codebase and produce knowledge/infrastructure.md and knowledge/open-questions.md.
> Map CI/CD pipelines, Docker/container config, IaC (Terraform, Pulumi, CDK), environments,
> and monitoring. For open-questions.md: capture EVERYTHING that cannot be determined from code
> alone — every uncertainty, assumption, and gap needing human input. Be comprehensive.
> Write only to the knowledge/ directory.

Wait for completion. Verify files before proceeding.

---

### Step 6: Generate README.md and INDEX.md

The orchestrator (you) generates these two documents directly — they are small and require
reading across all previously produced KB documents.

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
| architecture.md | {2-3 line summary of patterns found and structure} |
| technology-stack.md | {2-3 line summary of stack} |
| module-map.md | {2-3 line summary of modules} |
| coding-standards.md | {2-3 line summary of key conventions} |
| data-model.md | {2-3 line summary of data model} |
| api-contracts.md | {2-3 line summary of API surface} |
| integration-map.md | {2-3 line summary of integrations} |
| domain-glossary.md | {2-3 line summary of key terms} |
| test-landscape.md | {2-3 line summary of test coverage} |
| security-model.md | {2-3 line summary of security posture} |
| tech-debt.md | {2-3 line summary of debt and risk} |
| infrastructure.md | {2-3 line summary of infrastructure} |
| open-questions.md | {2-3 line summary of gaps needing human input} |
```

Regenerate INDEX.md on every discovery run (full or targeted).

---

### Step 7: Update Project Config Files

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

---

### Step 8: Final Verification

List all 13 expected KB documents. Check each exists. Report any missing.

Print: `[13/13] Discovery complete — Knowledge Base ready`

If any documents are missing, report them and offer to re-dispatch the responsible subagent.

---

## Targeted Discovery (Re-entry)

When a GAP.md or IMPEDIMENT.md triggers re-discovery of a specific area:

1. Read the GAP/IMPEDIMENT to understand exactly what's missing
2. Identify which subagent owns the missing documents:
   - `architecture.md`, `technology-stack.md` → dispatch discovery-architect
   - `module-map.md`, `coding-standards.md`, `data-model.md` → dispatch discovery-analyst
   - `api-contracts.md`, `integration-map.md`, `domain-glossary.md` → dispatch discovery-integrator
   - `test-landscape.md`, `security-model.md`, `tech-debt.md` → dispatch discovery-quality
   - `infrastructure.md`, `open-questions.md` → dispatch discovery-scout
3. Dispatch ONLY the relevant subagent for the area that needs updating
4. Regenerate README.md and INDEX.md (orchestrator does this directly)
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
