---
name: aid-discover
description: >
  Brownfield codebase discovery with built-in quality gate. Generates KB, reviews it,
  and fixes issues — one step per run. State-machine: GENERATE → REVIEW → FIX → DONE.
  Sequential processing (no subagents). Checkpoint/resume logic for resilient analysis.
allowed-tools: Read, Glob, Grep, Bash, Write, Edit
argument-hint: "[--grade A] minimum acceptable grade (default: A)  [--reset] clear KB and restart"
---

# Brownfield Codebase Discovery

Analyze an existing codebase and produce a structured `knowledge/` directory.
Includes a built-in quality gate that reviews, grades, and fixes KB documents.

> **Note:** Codex does not support subagents. All document generation is handled directly
> in this session, one document at a time. Checkpoint/resume logic skips existing files,
> so the skill is safe to re-run after interruption.

**State machine — each `/aid-discover` run does ONE step and exits.**

## Arguments

| Argument | Effect |
|----------|--------|
| `--grade X` | Set minimum acceptable grade. Format: `[A-F][-+]?` (e.g., A, A-, B+). Default: `A`. Persists in DISCOVERY-GRADE.md — user doesn't need to repeat it. |
| `--reset` | Clear entire `knowledge/` directory and restart from scratch. |

**Grade persistence:**
- When DISCOVERY-GRADE.md doesn't exist yet: the `--grade` value is saved into the file as "Minimum Grade"
- When DISCOVERY-GRADE.md already exists: if `--grade` is provided, it UPDATES the minimum in the file. If not provided, the minimum is READ from the file.

---

## State Detection

Read the filesystem to determine which mode to enter:

```
State 1: Missing KB docs           → GENERATE mode
State 2: All docs, no GRADE file   → REVIEW mode
State 3: GRADE file, grade < min   → FIX mode
State 4: GRADE file, grade >= min  → DONE
```

**Detection logic:**

1. Check `knowledge/` for the 13 expected documents:
   ```
   architecture.md, technology-stack.md, module-map.md, coding-standards.md, data-model.md,
   api-contracts.md, integration-map.md, domain-glossary.md, test-landscape.md,
   security-model.md, tech-debt.md, infrastructure.md, open-questions.md
   ```
2. If any are missing → **GENERATE**
3. If all 13 exist but `knowledge/DISCOVERY-GRADE.md` does not exist → **REVIEW**
4. If `knowledge/DISCOVERY-GRADE.md` exists:
   - Read the current overall grade and minimum grade
   - If `--grade` was provided, update the minimum grade in the file
   - Compare current grade against minimum (use grade ordering below)
   - If current grade < minimum → **FIX**
   - If current grade >= minimum → **DONE**

**Grade ordering** (highest to lowest):
`A+, A, A-, B+, B, B-, C+, C, C-, D+, D, D-, F`

Print the detected state: `[State: {GENERATE|REVIEW|FIX|DONE}]`

---

## Mode: GENERATE

Generate missing KB documents sequentially. **Skip any document that already exists.**

### Step 0: Check Existing KB

Scan `knowledge/` for existing files. Print: `[0/13] Checking existing KB...`

If ALL 13 documents exist and no `--reset` was requested, report KB is complete and skip
directly to Step 12 (README.md and INDEX.md regeneration).

---

### Step 1-2: Architecture Analysis → architecture.md

Print: `[1/13] Analyzing project structure and architecture...`
Skip if `knowledge/architecture.md` exists.

Detect project type, map folder structure, count by language. Identify entry points,
patterns (MVVM, CQRS, Clean Architecture, etc.), module boundaries, data flow,
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

**knowledge/README.md** — completeness tracking table and revision history.

**knowledge/INDEX.md** — 2-3 line summary of every KB document for agent self-service.

Regenerate INDEX.md on every discovery run (full or targeted).

---

### Step 13: Update Project Config Files

Print: `[13/13] Updating project config files...`

Scan the project root for `AGENTS.md` and `CLAUDE.md`. Replace any `<!-- AID:DISCOVER ... -->`
placeholders with real data from the analysis.

Keep the `<!-- AID:DISCOVER ... -->` comment above each section so future re-discoveries can update them.
Replace only the `(pending discovery)` placeholder lines with real content.
If a field cannot be determined, leave it as `(not found — check open-questions.md)`.

Print: `Generation complete — Knowledge Base ready. Run /aid-discover again to review.`

---

## Mode: REVIEW

All 13 KB documents exist. Grade them.

### Step 1: Review All Documents

Read every document in `knowledge/`, plus `AGENTS.md` and `CLAUDE.md`. For each document:

1. **Completeness** — Does it cover what it should?
2. **Accuracy** — Cross-reference 3-5 claims per doc against actual source code
3. **Depth** — Lists things vs explains patterns and relationships
4. **Usefulness** — Would an agent building a feature find this helpful?
5. **Grade** — A+ through F (see grading criteria below)

**Minimum 15 total spot-checks** where you verify a claim from a KB doc against actual code.
At least 5 must be version verifications (check actual pom.xml, package.json, jar filenames, manifests).
All issues MUST have severity: [CRITICAL], [HIGH], [MEDIUM], or [MINOR].

**Grade caps (absolute — no exceptions):**
Multiple [CRITICAL] → max D. One [CRITICAL] → max D+.
Multiple [HIGH] → max C. One [HIGH] → max C+.
Multiple [MEDIUM] → max B. One [MEDIUM] → max B+.
Multiple [MINOR] → max A-. One [MINOR] → max A. Only zero issues = A+.

**Do NOT trust what the document says.** Verify claims against actual source files.

Print: `[Review 1/2] Reviewing Knowledge Base quality...`

---

### Step 2: Write DISCOVERY-GRADE.md

Write `knowledge/DISCOVERY-GRADE.md` using the template format (see `templates/reports/discovery-grade-template.md`).

Set the Minimum Grade:
- If `--grade` was provided, use that value
- If not, use the default: `A`

Add the first Review History entry.

Print: `[Review 2/2] Review complete. Grade: {overall}. Minimum: {min}. Run /aid-discover again to {fix issues|proceed}.`

---

## Mode: FIX

DISCOVERY-GRADE.md exists but the overall grade is below the minimum.

### Step 1: Identify Documents Below Threshold

Read DISCOVERY-GRADE.md. List all documents graded below the minimum grade.
Prioritize: [CRITICAL] issues first, then [HIGH], then [MEDIUM].

Print: `[Fix] {N} documents below {minimum}. Fixing...`

---

### Step 2: Fix Each Document

For each document below the minimum grade, in priority order:

1. Read the specific issues from the Issues Found section of DISCOVERY-GRADE.md
2. Read the relevant source code to gather missing information
3. Edit the KB document to address the issues — be specific, add evidence
4. **REMOVE the fixed issue lines** from the Issues Found section of DISCOVERY-GRADE.md
5. Re-grade the document

Print: `[Fix] Improving {document}... {old grade} → {new grade}`

**IMPORTANT:** When an issue is fixed, its line MUST be removed from the Issues Found section.
DISCOVERY-GRADE.md always reflects the CURRENT state, not history. History is tracked in the
Review History table.

---

### Step 3: Re-Review (MANDATORY — Do NOT Self-Evaluate)

**After fixing all documents, you MUST review them again from scratch.**
The agent that wrote the fix CANNOT evaluate its own work. Re-read every fixed document
and re-verify claims against source code. Apply the same grade caps.

Print: `[Fix] Re-reviewing after fixes...`

Re-run the full review process from REVIEW Step 1 on all documents.
Overwrite DISCOVERY-GRADE.md with the fresh assessment. Preserve Review History entries.

### Step 4: Update DISCOVERY-GRADE.md

After re-review:
1. The reviewer has recalculated grades based on fresh assessment
2. Update the Documents table with new grades and statuses
3. Update the Last Run timestamp
4. Add a new entry to the Review History table
5. Update the Recommendation field

Print: `[Fix] Complete. Grade: {old} → {new}. Run /aid-discover again to {re-review|proceed}.`

---

## Mode: DONE

Print: `✅ Discovery complete. Grade: {grade}. Minimum: {minimum}. KB is ready for the Interview phase.`

---

## Targeted Discovery (Re-entry)

When a GAP.md or IMPEDIMENT.md triggers re-discovery of a specific area:

1. Read the GAP/IMPEDIMENT to understand exactly what's missing
2. Delete (or overwrite) only the specific KB documents that need updating
3. Re-run only the relevant steps for the missing/outdated documents
4. Regenerate README.md and INDEX.md
5. Update `knowledge/README.md` revision history with the targeted update
6. Delete `knowledge/DISCOVERY-GRADE.md` so the next run re-reviews
7. Report completion to the calling phase

---

## Quality Checklist

- [ ] No overlap between KB documents
- [ ] Claims grounded in code evidence (file paths, line numbers)
- [ ] Inferred info marked with ⚠️
- [ ] open-questions.md captures everything needing human input
- [ ] README.md reflects completeness status and revision history
- [ ] INDEX.md generated with 2-3 line summaries of every KB document
- [ ] AGENTS.md and CLAUDE.md placeholders filled with discovered data
- [ ] All issues in DISCOVERY-GRADE.md have severity: [CRITICAL], [HIGH], or [MEDIUM]
- [ ] Minimum 10 spot-checks in DISCOVERY-GRADE.md

---

## Grading Criteria

| Grade | Meaning |
|-------|---------|
| A+ | Exceptional — comprehensive, accurate, evidence-rich, immediately useful |
| A | Thorough — covers expected scope with solid evidence |
| B+ | Good — minor gaps or missing details that don't block work |
| B | Adequate — covers basics but lacks depth in important areas |
| B- | Shallow — lists things without explaining patterns or relationships |
| C+ | Significant gaps — missing important sections or inaccurate in places |
| C | Barely useful — an agent would need to re-discover most information |
| D | Misleading — contains wrong information that could cause bad decisions |
| F | Missing or empty |

**Overall grade** = weighted average where architecture, module-map, and coding-standards
count double (they're referenced most by downstream phases).

---

## Document Expectations

- **architecture.md**: folder structure, patterns with evidence, data flow, entry points
- **technology-stack.md**: versions from actual config (not "TBD")
- **module-map.md**: every module with purpose AND dependencies
- **coding-standards.md**: conventions from actual code, not generic advice
- **data-model.md**: entities with relationships, not just lists
- **api-contracts.md**: actual URLs/paths, not just class names
- **integration-map.md**: connection details, NOT same content as module-map
- **domain-glossary.md**: project-specific terms
- **test-landscape.md**: which modules have real tests vs placeholders
- **security-model.md**: project-specific assessment, not generic OWASP
- **tech-debt.md**: severity-categorized with file locations
- **infrastructure.md**: CI/CD flow, deployment, monitoring details
- **open-questions.md**: specific, answerable, comprehensive
- **AGENTS.md/CLAUDE.md**: no placeholders, real working commands
