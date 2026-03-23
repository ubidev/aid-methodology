---
name: aid-specify
description: >
  Transform REQUIREMENTS.md into a formal SPEC.md grounded in the Knowledge Base.
  Orchestrates 4 sequential spec subagents + 1 reviewer with built-in quality gate.
  State-machine: GENERATE → REVIEW → Q&A → FIX → APPROVAL → DONE.
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, Agent
argument-hint: "[--grade A] minimum acceptable grade (default: A)  [--reset] clear SPEC.md and restart"
---

# Generate Specification

Transform REQUIREMENTS.md into a formal SPEC.md grounded in the Knowledge Base by orchestrating
4 sequential specification subagents. Includes a built-in quality gate that reviews, grades,
collects user input, fixes issues, and gets user approval — one step per run.

**State machine — each `/aid-specify` run does ONE step and exits.**

## ⚠️ Pre-flight Check

**Before starting specify, verify you are NOT in Plan Mode.**

Plan Mode restricts all operations to read-only — subagents will NOT be able to write SPEC.md.

**How to check:** Look at the permission indicator in your Claude Code interface (bottom of screen).
- ✅ `Default` or `Auto-accept edits` → Proceed with specify.
- ❌ `Plan mode` → **STOP.** Tell the user: "Specify needs to write files. Please press `Shift+Tab` to switch out of Plan Mode, then re-run `/aid-specify`."

**Do NOT proceed with specify while in Plan Mode.** The subagents will analyze the requirements but silently fail to write any files.

## ⚠️ Pre-requisites

Before running specify, verify:

1. **REQUIREMENTS.md exists** — If missing, tell the user to run `/aid-interview` first.
2. **knowledge/ directory exists with INDEX.md** — If missing, tell the user to run `/aid-discover` first.

If either is missing, print the error and exit. Do not attempt to generate a spec without inputs.

## Arguments

| Argument | Effect |
|----------|--------|
| `--grade X` | Set minimum acceptable grade. Format: `[A-F][-+]?` (e.g., A, A-, B+). Default: `A`. Persists in SPEC-STATE.md — user doesn't need to repeat it. |
| `--reset` | Delete SPEC.md and SPEC-STATE.md and restart from scratch. |

**Grade persistence:**
- When SPEC-STATE.md doesn't exist yet: the `--grade` value is saved into the file as "Minimum Grade"
- When SPEC-STATE.md already exists: if `--grade` is provided, it UPDATES the minimum in the file. If not provided, the minimum is READ from the file.

---

## State Detection

⚠️ **FILESYSTEM IS THE ONLY SOURCE OF TRUTH.**
Do NOT rely on memory from previous runs in this session. Do NOT assume state based on
what happened earlier. ALWAYS read the actual files on disk right now.

Read the filesystem to determine which mode to enter:

```plaintext
State 1: No SPEC.md                                    → GENERATE mode
State 2: SPEC.md exists, no SPEC-STATE.md              → REVIEW mode
State 3: SPEC-STATE.md, grade < min, has Pending Q&A   → Q&A mode
State 4: SPEC-STATE.md, grade < min, no Pending Q&A    → FIX mode
State 5: SPEC-STATE.md, grade >= min, not user-approved → APPROVAL mode
State 6: SPEC-STATE.md, grade >= min, user-approved    → DONE
```

**Detection logic:**

1. Check for `SPEC.md` in the project root
2. If missing → **GENERATE**
3. If SPEC.md exists but `SPEC-STATE.md` does not exist → **REVIEW**
4. If `SPEC-STATE.md` exists:
   - Read the current overall grade and minimum grade
   - If `--grade` was provided, update the minimum grade in the file
   - Compare current grade against minimum (use grade ordering below)
   - If current grade < minimum:
     - Read SPEC-STATE.md for Q&A entries with `**Status:** Pending`
     - If Pending entries exist → **Q&A**
     - If no Pending entries → **FIX**
   - If current grade >= minimum:
     - Check SPEC-STATE.md for `**User Approved:** yes`
     - If approved → **DONE**
     - If not approved → **APPROVAL**

**Grade ordering** (highest to lowest):
`A+, A, A-, B+, B, B-, C+, C, C-, D+, D, D-, F`

Print the detected state: `[State: {GENERATE|REVIEW|Q&A|FIX|APPROVAL|DONE}]`

---

## Mode: GENERATE

Create SPEC.md by dispatching 4 subagents sequentially. Each subagent reads the full
REQUIREMENTS.md and KB (via INDEX.md), and builds on what previous subagents wrote.

### Step 0: Check for Existing SPEC.md

If SPEC.md already exists and `--reset` was not requested, report it exists and skip
directly to REVIEW mode detection.

If `--reset` was requested:
- Delete SPEC.md and SPEC-STATE.md
- Print: `[Reset] Cleared SPEC.md and SPEC-STATE.md. Starting fresh.`

---

### Step 1: Initialize SPEC.md

Before dispatching any subagent, create the SPEC.md skeleton from the template at
`templates/specs/spec-template.md`. Replace `{date}` with today's date.

This gives subagents a document to write into rather than starting from nothing.

Print: `[0/4] SPEC.md initialized from template. Dispatching spec agents...`

---

### Steps 2-5: Dispatch 4 Subagents Sequentially

**⚠️ SEQUENTIAL, NOT PARALLEL.** Each subagent reads the current state of SPEC.md before
writing. This ensures coherence — later agents can reference and build on earlier sections.

**⚠️ ALL subagents read REQUIREMENTS.md + KB (via INDEX.md).** Each subagent has full access
to the source material, not just what previous agents wrote.

**⚠️ Q&A ACCUMULATION.** Every subagent may add questions to the Q&A section of SPEC-STATE.md
if they encounter ambiguities that cannot be resolved from REQUIREMENTS.md or KB. Use the
structured format: unique ID (SQ{N}), category, impact level, status, context.

---

#### [1/4] Vision & Constraints (spec-vision)

Print: `[1/4] Dispatching spec-vision...`

Prompt:
> You are building a formal specification document (SPEC.md) from REQUIREMENTS.md and the
> Knowledge Base. You are the FIRST of 4 sequential agents — the document is currently a
> skeleton template.
>
> **Your responsibility:** Write the **Vision** and **Constraints** sections of SPEC.md.
>
> **Inputs (read ALL of these before writing):**
> - REQUIREMENTS.md — the source of truth for what needs to be built
> - knowledge/INDEX.md — read this first, then read the specific KB documents it points to
>   that are relevant to vision and constraints (especially technology-stack.md, architecture.md,
>   infrastructure.md, security-model.md)
>
> **Vision section:**
> - One clear paragraph explaining what this is, why it exists, what problem it solves
> - Written for a developer who knows nothing about this project
> - Grounded in REQUIREMENTS.md objectives, not generic language
>
> **Constraints section:**
> - Stack: from KB technology-stack.md — don't change unless REQUIREMENTS.md explicitly demands it
> - Platform: target environments, OS, browser support from REQUIREMENTS.md
> - Performance: P95 targets, throughput, data volume — from REQUIREMENTS.md or inferred from KB
> - Security: auth, authorization, data classification — from KB security-model.md + REQUIREMENTS.md
> - Non-negotiable: any hard constraint from REQUIREMENTS.md
>
> **Rules:**
> - Every claim must reference its source: REQUIREMENTS.md section or KB document
> - If a constraint comes from KB (existing system), say so: "Following existing pattern in knowledge/technology-stack.md"
> - If a constraint comes from REQUIREMENTS.md, say so: "Per REQUIREMENTS.md §{section}"
> - If something is unclear or contradictory between REQUIREMENTS.md and KB, do NOT guess.
>   Instead, create a Q&A entry in SPEC-STATE.md (create the file if it doesn't exist) with:
>   `SQ{N}`, category, impact level (High/Medium/Low), `**Status:** Pending`, and context.
> - Do NOT modify sections after Constraints — leave them for subsequent agents.
> - Write directly into SPEC.md, replacing the template placeholders for your sections.

Wait for completion. Verify SPEC.md has Vision and Constraints filled in.

---

#### [2/4] Architecture & Domain Model (spec-architect)

Print: `[2/4] Dispatching spec-architect...`

Prompt:
> You are building a formal specification document (SPEC.md) from REQUIREMENTS.md and the
> Knowledge Base. You are the SECOND of 4 sequential agents — Vision and Constraints have
> already been written by the previous agent.
>
> **Your responsibility:** Write the **Architecture** and **Domain Model** sections of SPEC.md.
>
> **Inputs (read ALL of these before writing):**
> - REQUIREMENTS.md — the source of truth for what needs to be built
> - knowledge/INDEX.md — read this first, then read the specific KB documents it points to
>   (especially architecture.md, module-map.md, data-model.md, domain-glossary.md,
>   coding-standards.md, integration-map.md)
> - SPEC.md — read the existing content (Vision, Constraints) to maintain consistency
>
> **Architecture section:**
> - Pattern: reference KB architecture.md — "Following the {pattern} established in
>   knowledge/architecture.md, this feature..."
> - New components: what modules/services/classes this adds, WHERE they live in existing structure
> - Modified components: existing code that changes, reference KB module-map.md module names
> - Data: new tables/schema changes, reference KB data-model.md for conventions
> - External integrations: new APIs/services, reference KB integration-map.md
>
> **Domain Model section:**
> - Key entities with properties, types, descriptions — using vocabulary from KB domain-glossary.md
> - Relationships between entities (1:N, M:N) with descriptions in domain language
> - Business rules that apply to each entity — in domain language, not implementation language
>
> **Rules:**
> - Read the existing SPEC.md content first. Your sections must be consistent with what's there.
> - If Vision or Constraints contain something that affects your sections, reference it.
> - If you find an issue in Vision/Constraints while writing, you MAY fix it — but only if it's
>   clearly wrong (e.g., wrong framework name). Otherwise, add a Q&A entry noting the concern.
> - Every architectural decision must reference KB architecture.md or coding-standards.md
> - Use domain terms from KB domain-glossary.md consistently
> - If REQUIREMENTS.md implies an architectural change that contradicts KB, flag it as Q&A.
> - Write directly into SPEC.md, replacing the template placeholders for your sections.
> - Add Q&A entries to SPEC-STATE.md if you encounter ambiguities. Use next sequential SQ{N} ID.

Wait for completion. Verify SPEC.md has Architecture and Domain Model filled in.

---

#### [3/4] Feature Specifications (spec-features)

Print: `[3/4] Dispatching spec-features...`

Prompt:
> You are building a formal specification document (SPEC.md) from REQUIREMENTS.md and the
> Knowledge Base. You are the THIRD of 4 sequential agents — Vision, Constraints, Architecture,
> and Domain Model have already been written by previous agents.
>
> **Your responsibility:** Write the **Feature Specifications** section of SPEC.md.
>
> **Inputs (read ALL of these before writing):**
> - REQUIREMENTS.md — the source of truth for features. Every feature in REQUIREMENTS.md must
>   appear in your Feature Specifications section (unless explicitly marked as out of scope)
> - knowledge/INDEX.md — read this first, then read relevant KB documents (especially
>   coding-standards.md, api-contracts.md, data-model.md, module-map.md, test-landscape.md)
> - SPEC.md — read the existing content (Vision, Constraints, Architecture, Domain Model)
>   to maintain consistency and reference decisions already made
>
> **For each feature from REQUIREMENTS.md:**
>
> - **Priority:** Must / Should / Could (from REQUIREMENTS.md)
> - **Behavior:** What happens from user's perspective. Happy path first, then edge cases.
>   Ground in the Architecture section already written — reference the components defined there.
> - **Interface:** Language-specific interface definition following patterns from KB
>   coding-standards.md. Use the conventions actually in the repository, not generic patterns.
> - **Acceptance criteria:** Concrete, testable criteria. Include happy path, edge cases,
>   error cases. These become test cases in the Test phase.
> - **Error handling:** Table of conditions, responses, and notes. Follow error patterns
>   from KB coding-standards.md.
>
> **Rules:**
> - Read the existing SPEC.md content first. Reference Architecture and Domain Model sections.
> - Use the entities defined in Domain Model — don't introduce new entities without updating it.
> - If a feature requires a new entity not in Domain Model, ADD it to Domain Model (you have
>   edit rights to earlier sections when the change is additive and clearly needed).
> - Interface definitions must follow KB coding-standards.md conventions (naming, patterns, DI).
> - If REQUIREMENTS.md is vague about a feature, do NOT invent requirements. Add Q&A entry.
> - Write directly into SPEC.md, replacing the template placeholders for your section.
> - Add Q&A entries to SPEC-STATE.md if you encounter ambiguities. Use next sequential SQ{N} ID.

Wait for completion. Verify SPEC.md has Feature Specifications filled in.

---

#### [4/4] Non-Functional Requirements & Scope (spec-nfr)

Print: `[4/4] Dispatching spec-nfr...`

Prompt:
> You are building a formal specification document (SPEC.md) from REQUIREMENTS.md and the
> Knowledge Base. You are the FOURTH and FINAL agent — Vision, Constraints, Architecture,
> Domain Model, and Feature Specifications have been written by previous agents.
>
> **Your responsibility:** Write the **Non-Functional Requirements**, **Out of Scope**, and
> **Revision History** sections of SPEC.md. Also perform a coherence pass over the full document.
>
> **Inputs (read ALL of these before writing):**
> - REQUIREMENTS.md — NFR requirements, out-of-scope items, priorities
> - knowledge/INDEX.md — read this first, then read relevant KB documents (especially
>   test-landscape.md, infrastructure.md, coding-standards.md, security-model.md)
> - SPEC.md — read the FULL existing content to ensure your sections are consistent and
>   to perform the coherence pass
>
> **Non-Functional Requirements section:**
> - Observability: logging events, levels, metrics — follow patterns from KB coding-standards.md
> - Testing: coverage targets, integration test scope, E2E flows — reference KB test-landscape.md
> - Accessibility: WCAG level if web — from REQUIREMENTS.md
> - i18n: localization if applicable — from REQUIREMENTS.md
> - Any other NFR from REQUIREMENTS.md (performance targets not already in Constraints, etc.)
>
> **Out of Scope section:**
> - Explicitly list what's excluded, from REQUIREMENTS.md
> - Include future-version items and won't-fix items
> - Be explicit — this prevents scope creep during implementation
>
> **Revision History:**
> - Add the initial entry: `| 1.0 | {date} | aid-specify | Initial specification |`
>
> **Coherence pass (IMPORTANT):**
> After writing your sections, read the FULL SPEC.md and check:
> - Do Feature Specifications reference entities from Domain Model consistently?
> - Do interfaces follow the conventions stated in Constraints?
> - Are there contradictions between sections?
> - Are domain terms used consistently throughout?
> If you find issues, FIX them (you have edit rights to all sections for coherence fixes).
> If a fix would be controversial (changing a design decision), add Q&A entry instead.
>
> **Rules:**
> - Write directly into SPEC.md, replacing the template placeholders for your sections.
> - For coherence fixes to earlier sections, use targeted edits — don't rewrite large sections.
> - Add Q&A entries to SPEC-STATE.md if you encounter ambiguities. Use next sequential SQ{N} ID.

Wait for completion. Verify SPEC.md is complete (all template sections filled in).

---

### Step 6: Post-Generate Verification

After all 4 subagents have completed, verify SPEC.md:

```bash
echo "=== SPEC.md Section Check ==="
grep -n "^## " SPEC.md
echo ""
echo "=== Template Placeholders Remaining ==="
grep -n '{.*}' SPEC.md | head -20
echo ""
echo "=== File Size ==="
wc -l SPEC.md
```

Check for:
- [ ] All template sections replaced with real content
- [ ] No `{placeholder}` text remaining (except in code examples where braces are literal)
- [ ] Vision section present and non-empty
- [ ] Constraints section present with stack details
- [ ] Architecture section present with components
- [ ] Domain Model section present with entities
- [ ] Feature Specifications section present with at least one feature
- [ ] Non-Functional Requirements section present
- [ ] Out of Scope section present
- [ ] Revision History section present

If any section is still a template placeholder, re-dispatch the responsible subagent:
| Subagent | Sections |
|----------|----------|
| spec-vision | Vision, Constraints |
| spec-architect | Architecture, Domain Model |
| spec-features | Feature Specifications |
| spec-nfr | Non-Functional Requirements, Out of Scope, Revision History |

Print: `[4/4] SPEC.md generation complete. Run /aid-specify again to review.`

If SPEC-STATE.md was created (Q&A entries from subagents), note it:
`[4/4] SPEC.md generation complete. {N} questions accumulated for Q&A. Run /aid-specify again to review.`

---

## Mode: REVIEW

SPEC.md exists. Grade it.

### Step 1: Dispatch the Reviewer

Dispatch **spec-reviewer** subagent.

Print: `[Review 1/2] Reviewing specification quality...`

**⚠️ CLEAN CONTEXT:** Do NOT include any information about the generation process,
which agents ran, what was easy or hard, or any prior state. The reviewer must
evaluate the spec purely on what's on disk — as if a stranger wrote it.

Prompt to pass to the subagent:
> Review SPEC.md for quality against REQUIREMENTS.md and the Knowledge Base. Be AGGRESSIVE —
> a lenient review is worse than useless because it lets bad specs through to implementation.
>
> **Inputs:**
> - SPEC.md — the specification to review
> - REQUIREMENTS.md — the source requirements (does the spec cover everything?)
> - knowledge/INDEX.md — read this, then relevant KB documents (does the spec respect existing architecture?)
>
> For each section of SPEC.md, assess:
>
> 1. **Requirements Coverage** (MOST IMPORTANT) — Does the spec address every requirement
>    from REQUIREMENTS.md?
>    - Map each requirement to its spec coverage. If a requirement has no corresponding
>      feature specification, that's [CRITICAL].
>    - If acceptance criteria don't cover a requirement's edge cases, that's [HIGH].
>    - Requirements marked "Must" in REQUIREMENTS.md that lack detailed spec → [CRITICAL].
>    - Requirements marked "Should" that are completely missing → [HIGH].
>
> 2. **KB Grounding** — Is the spec consistent with the existing project?
>    - Architecture decisions that contradict KB architecture.md → [CRITICAL]
>    - New components placed in wrong layers (per KB module-map.md) → [HIGH]
>    - Interface definitions that don't follow KB coding-standards.md → [MEDIUM]
>    - Domain terms inconsistent with KB domain-glossary.md → [MEDIUM]
>    - Generic patterns used where KB documents project-specific ones → [MEDIUM]
>
> 3. **Internal Consistency** — Does the spec contradict itself?
>    - Domain Model entity used in Features but not defined → [HIGH]
>    - Constraints say one thing, Architecture says another → [CRITICAL]
>    - Feature interfaces reference non-existent entities or services → [HIGH]
>
> 4. **Implementability** — Could a developer implement from this spec?
>    - Vague acceptance criteria ("should work well") → [HIGH]
>    - Missing error handling for obvious failure modes → [MEDIUM]
>    - Interface definitions without enough detail to implement → [MEDIUM]
>    - Ambiguous behavior for edge cases → [MEDIUM]
>
> 5. **Completeness** — Are all expected sections present and substantive?
>    - Empty or placeholder sections → [CRITICAL]
>    - Sections that just repeat REQUIREMENTS.md without adding spec detail → [HIGH]
>    - Missing NFR coverage for areas the project clearly needs → [MEDIUM]
>
> 6. **Traceability** — Can you trace each spec element back to its source?
>    - Spec decisions without source reference (no "Per REQUIREMENTS.md §X" or
>      "Following KB architecture.md") → [MINOR]
>    - Decisions that appear invented (not in REQUIREMENTS.md or KB) → [HIGH]
>
> Grade each section: A+ (exceptional), A (thorough), B+ (good with minor gaps), B (adequate),
> B- (shallow), C+ (significant gaps), C (barely useful), D (misleading or wrong), F (missing/empty).
>
> **Grade caps (absolute — no exceptions):**
> Multiple [CRITICAL] → max D. One [CRITICAL] → max D+.
> Multiple [HIGH] → max C. One [HIGH] → max C+.
> Multiple [MEDIUM] → max B. One [MEDIUM] → max B+.
> Multiple [MINOR] → max A-. One [MINOR] → max A. Only zero issues = A+.
>
> All issues MUST have severity: [CRITICAL], [HIGH], [MEDIUM], or [MINOR].
>
> **Requirements Coverage Matrix:** Include a table mapping REQUIREMENTS.md items to SPEC.md
> coverage. Every requirement must appear. Mark as: ✅ Covered, ⚠️ Partial, ❌ Missing.
>
> **Minimum 10 spot-checks:**
> - At least 3 verifying KB consistency (check spec claims against actual KB documents)
> - At least 3 verifying requirements coverage (check spec features against REQUIREMENTS.md)
> - At least 2 verifying internal consistency (check cross-references within SPEC.md)
>
> **After grading, add Q&A entries to SPEC-STATE.md** for any ambiguities found during review
> that cannot be resolved from REQUIREMENTS.md or KB. These become Q&A items for the user.
> Use sequential SQ{N} IDs, categorize by area, and assign impact levels. Only add questions
> for things genuinely needing human input.
>
> Write the full review to SPEC-STATE.md using this structure:
>
> ```markdown
> # SPEC-STATE.md — Specification Review
>
> **Overall Grade:** {grade}
> **Minimum Grade:** {min}
>
> ## Requirements Coverage Matrix
>
> | Requirement | Section | SPEC Coverage | Status |
> |-------------|---------|---------------|--------|
> | {req} | {REQUIREMENTS.md section} | {SPEC.md section/feature} | ✅/⚠️/❌ |
>
> ## Section Grades
>
> | Section | Grade | Issues |
> |---------|-------|--------|
> | Vision | {grade} | {count} |
> | Constraints | {grade} | {count} |
> | ... | ... | ... |
>
> ## Issues Found
>
> ### {Section Name}
>
> - [{SEVERITY}] {description} — {evidence}
>
> ## Spot-Checks
>
> | # | Type | Check | Result |
> |---|------|-------|--------|
> | 1 | KB Consistency | {what was checked} | ✅/❌ {finding} |
>
> ## Cross-Cutting Concerns
>
> {observations that span multiple sections}
>
> ## Q&A — Pending Questions
>
> ### SQ{N}: [{Category}: {Impact}]
>
> **Question:** {question text}
> **Context:** {why this matters}
> **Suggested:** {answer if inferrable}
> **Status:** Pending
>
> ## Review History
>
> | # | Date | Grade | Type | Notes |
> |---|------|-------|------|-------|
> | 1 | {date} | {grade} | Initial Review | {summary} |
> ```

Wait for completion.

---

### Step 2: Post-Process SPEC-STATE.md

Read `SPEC-STATE.md`. Verify it contains:
- [ ] Overall grade
- [ ] Requirements Coverage Matrix
- [ ] Section grades for every SPEC.md section
- [ ] Specific issues with severity levels ([CRITICAL], [HIGH], [MEDIUM], [MINOR])
- [ ] Verification spot-checks (minimum 10)
- [ ] Review History entry

Set the Minimum Grade in the file:
- If `--grade` was provided, use that value
- If not, use the default: `A`

Print: `[Review 2/2] Review complete. Grade: {overall}. Minimum: {min}. Run /aid-specify again to {fix issues|proceed}.`

**Grade comparison:**
- If overall grade >= minimum → Next run will enter APPROVAL mode (user sign-off)
- If overall grade < minimum and Pending Q&A exists → Next run will enter Q&A mode
- If overall grade < minimum and no Pending Q&A → Next run will enter FIX mode

---

## Mode: Q&A

SPEC-STATE.md exists, the grade is below minimum, and SPEC-STATE.md contains Q&A entries
with `**Status:** Pending`.

The Q&A mode presents questions to the user **one at a time**, collects answers, and updates
SPEC-STATE.md.

### Step 1: Load and Filter Questions

Read SPEC-STATE.md. Collect all Q&A entries with `**Status:** Pending`.

**Before presenting each question, apply the filter:**

1. **Check REQUIREMENTS.md** — Is the answer already there? → Change status to `Answered`,
   fill in the answer from REQUIREMENTS.md, cite the section.
2. **Check KB** — Is the answer in a KB document? → Change status to `Answered`,
   fill in the answer from the KB document.
3. **Inferrable from context?** — Can the answer be reasonably deduced from existing material?
   → Keep the question but ensure it has a `**Suggested:**` answer. The user confirms or corrects.

After filtering, sort remaining Pending questions by impact: **High → Medium → Low**.

Print: `[Q&A] {N} questions for user input. Asking one at a time...`

If zero questions remain after filtering, print: `[Q&A] All questions resolved from existing material. Proceeding to Fix.`
and skip to the end of Q&A mode.

### Step 2: Ask One Question at a Time

For each Pending question, present it to the user in this format:

```
SQ{N}: [{Category}: {Impact}] {question text}

Context: {context from the Q&A entry}

Suggested: {suggested answer, if present}

[1] Skip / Not applicable
[2] Accept suggestion (only shown if Suggested exists)
[3] Your answer: ___
```

**Wait for the user's response before asking the next question.**

### Step 3: Record the Answer

Based on the user's response, update the entry in SPEC-STATE.md:

- **User chose [1] (Skip):** Set `**Status:** Skipped`
- **User chose [2] (Accept suggestion):** Set `**Status:** Answered`, copy the suggested text to `**Answer:**`
- **User typed an answer [3]:** Set `**Status:** Answered`, record their text in `**Answer:**`

**Important:** Write the update to the file immediately after each answer. Do not batch.

### Step 4: Continue or Exit

After recording, move to the next Pending question. Repeat Steps 2-3.

When all Pending questions have been addressed (answered or skipped):

Print: `[Q&A] Complete. {answered} answered, {skipped} skipped. Run /aid-specify again to fix.`

The next run will detect no Pending Q&A and enter FIX mode. The FIX will use the answers
from SPEC-STATE.md to improve SPEC.md.

---

## Mode: FIX

SPEC-STATE.md exists but the overall grade is below the minimum.

### Step 1: Identify Issues

Read SPEC-STATE.md. List all issues by severity, and all Answered Q&A entries not yet applied.
Prioritize: [CRITICAL] issues first, then [HIGH], then [MEDIUM], then Answered Q&A.

Print: `[Fix] {N} issues + {M} answered questions to apply. Fixing...`

---

### Step 2: Fix SPEC.md

Work through issues in priority order:

1. Read the specific issues from SPEC-STATE.md
2. Read Answered Q&A entries that apply to the area being fixed
3. Read REQUIREMENTS.md sections relevant to the issue
4. Read KB documents relevant to the issue (via INDEX.md)
5. Edit SPEC.md to address the issues — combining review findings WITH user answers.
   A review finding ("features section doesn't cover authentication flow") + a user answer
   ("OAuth2 with PKCE, redirect to /callback") together produce a precise, evidence-backed fix.
6. **REMOVE the fixed issue lines** from the Issues Found section of SPEC-STATE.md
7. For each Answered Q&A entry that was incorporated, update its `**Status:**` to `Applied`
   and add `**Applied to:** SPEC.md §{section}, cycle {N}`

Print progress: `[Fix] Fixing {section}... {description of fix}`

**IMPORTANT:** When an issue is fixed, its line MUST be removed from the Issues Found section.
SPEC-STATE.md always reflects the CURRENT state, not history. History is tracked in the
Review History table.

**IMPORTANT:** Answered Q&A entries are fix items too. Treat them with the same priority as
review findings. Cross-reference findings and answers — they often complement each other.

---

### Step 2b: Coherence Check (MANDATORY after every fix pass)

After all fixes, perform a coherence check on SPEC.md:

1. **Internal consistency** — Do Features reference entities that exist in Domain Model?
   Do interfaces follow conventions stated in Constraints? Are domain terms consistent?
2. **Requirements coverage** — Re-check the Coverage Matrix. Did fixes close any gaps?
   Did fixes introduce new gaps?
3. **KB alignment** — Do any fixes now contradict KB? (e.g., changing an interface pattern
   away from what KB coding-standards.md documents)

Fix any coherence issues found. If a coherence fix would be controversial, add a Q&A entry.

Print: `[Fix] Coherence check complete.`

---

### Step 3: Re-Review (MANDATORY — Do NOT Self-Evaluate)

**After fixing all issues, you MUST dispatch the `spec-reviewer` subagent again.**
The agent that wrote the fix CANNOT evaluate its own work. This is a hard rule.

Print: `[Fix 2/3] Re-reviewing after fixes...`

Dispatch **spec-reviewer** with the prompt from REVIEW mode Step 1.
The reviewer will overwrite SPEC-STATE.md with a fresh assessment.

**⚠️ CONTAMINATION PREVENTION:**
- Do NOT include previous review results in the prompt to the reviewer.
- Do NOT tell the reviewer what was fixed or what the previous grade was.
- Do NOT say "re-review" or "verify the fixes" — the reviewer must approach
  SPEC.md as if seeing it for the first time.
- The reviewer's `background: true` gives it an isolated context window — but
  only if you don't inject prior results into its prompt.
- If the reviewer asks about previous grades, respond: "Evaluate fresh. No prior context."

Wait for completion.

---

### Step 4: Post-Fix Update

Read the new SPEC-STATE.md produced by the reviewer.

1. Verify the Review History was preserved (reviewer should append, not replace)
2. If Review History is missing entries from before the re-review, add them back
3. Ensure the new entry reflects this was a Fix + Re-review cycle

Print: `[Fix 3/3] Complete. Grade: {old} → {new}. Run /aid-specify again to {fix remaining issues|proceed}.`

**If the grade is still below minimum:** The next run will check for Pending Q&A entries first (→ Q&A mode) or proceed to FIX mode if none. This is expected — some fixes may introduce new issues or the reviewer may catch things the fixer missed. The cycle continues until the grade meets the minimum.

**If the grade meets the minimum:** The next run will enter APPROVAL mode for user sign-off.

---

## Mode: APPROVAL

SPEC-STATE.md exists, the grade meets or exceeds the minimum, but the user has not
yet approved the specification.

### Step 1: Present Summary

Print a brief summary of the spec state:
- Overall grade and minimum
- Requirements Coverage Matrix summary (how many ✅/⚠️/❌)
- Total Q&A items (answered/skipped/pending)
- Number of fix cycles completed
- Any remaining [MINOR] issues

### Step 2: Ask for User Approval

```
The specification has reached the minimum grade of {minimum} (current: {grade}).

Requirements coverage: {covered}/{total} fully covered, {partial} partial, {missing} missing.

Please review SPEC.md and let us know if there is anything else we should consider.

[1] Approved — SPEC.md is ready for the next phase
[2] Additional consideration: ___
```

**Wait for the user's response.**

### Step 3: Process Response

- **User chose [1] (Approved):**
  - Add `**User Approved:** yes` to the top of SPEC-STATE.md (after the Minimum Grade line)
  - Add a Review History entry: `| {N} | {date} | {grade} | User Approval | User approved spec for next phase |`
  - Print: `✅ Specify complete. Grade: {grade}. SPEC.md approved and ready for the Plan phase.`

- **User provided additional consideration [2]:**
  - Add the consideration as a new Q&A entry in SPEC-STATE.md with:
    - Next sequential SQ{N} ID
    - `[User Feedback: High]` category and impact
    - `**Status:** Pending`
    - The user's text as the context
  - Print: `[Approval] Consideration recorded as SQ{N}. Run /aid-specify again to address it.`
  - The next run will detect Pending Q&A and re-enter the cycle.

---

## Mode: DONE

SPEC-STATE.md exists, the grade meets or exceeds the minimum, and the user has approved.

Print: `✅ Specify complete. Grade: {grade}. Minimum: {minimum}. SPEC.md approved and ready for the Plan phase.`

No action needed. The user can proceed to `/aid-plan`.

---

## Targeted Specify (Re-entry)

When a GAP.md or IMPEDIMENT.md triggers spec revision for a specific area:

1. Read the GAP/IMPEDIMENT to understand exactly what needs revision
2. Identify which subagent owns the affected sections:
   - Vision, Constraints → dispatch spec-vision
   - Architecture, Domain Model → dispatch spec-architect
   - Feature Specifications → dispatch spec-features
   - Non-Functional Requirements, Out of Scope → dispatch spec-nfr
3. Dispatch ONLY the relevant subagent with a targeted prompt that includes the GAP context
4. The subagent reads the current SPEC.md + REQUIREMENTS.md + KB and makes targeted edits
5. Delete SPEC-STATE.md so the next run re-reviews
6. Report completion to the calling phase

---

## Quality Checklist

- [ ] Every requirement from REQUIREMENTS.md has corresponding spec coverage
- [ ] Architecture decisions reference KB documents (architecture.md, coding-standards.md)
- [ ] Domain terms match KB domain-glossary.md
- [ ] Interface definitions follow KB coding-standards.md conventions
- [ ] Acceptance criteria are concrete and testable (no "should work well")
- [ ] Error handling covers obvious failure modes
- [ ] No contradictions between SPEC.md sections
- [ ] No placeholder text remaining from template
- [ ] Constraints are sourced (KB or REQUIREMENTS.md reference for each)
- [ ] Out of Scope explicitly lists exclusions from REQUIREMENTS.md

---

## Grading Criteria

| Grade | Meaning |
|-------|---------|
| A+ | Exceptional — every requirement covered, KB-grounded, immediately implementable |
| A | Thorough — comprehensive coverage with solid traceability |
| B+ | Good — minor gaps or vague criteria that don't block implementation |
| B | Adequate — covers basics but some features need more detail |
| B- | Shallow — lists features without enough detail to implement confidently |
| C+ | Significant gaps — missing features or contradicts KB |
| C | Barely useful — a developer would need to re-interview to implement |
| D | Misleading — contradicts requirements or KB, would cause wrong implementation |
| F | Missing or empty |

**Overall grade** = weighted average where Feature Specifications and Architecture
count double (they're referenced most by downstream phases).

---

## Section Expectations

These define what the reviewer should look for in each SPEC.md section.

### Vision
Must have: clear one-paragraph explanation of purpose, grounded in REQUIREMENTS.md.
**Red flags**: Generic vision that could apply to any project. No reference to actual problem.

### Constraints
Must have: stack (from KB), platform, performance targets, security requirements, non-negotiables.
Each constraint sourced (KB or REQUIREMENTS.md reference).
**Red flags**: Generic constraints. Missing stack details extractable from KB. Performance
targets without numbers. Security section that's just "TBD."

### Architecture
Must have: pattern reference to KB architecture.md, new components with layer placement,
modified components referencing KB module-map.md, data changes referencing KB data-model.md.
**Red flags**: Architecture decisions that ignore existing patterns. New components without
layer placement. "We should use microservices" when KB shows monolith.

### Domain Model
Must have: all entities needed by Feature Specifications, with properties, types,
descriptions in domain language (from KB domain-glossary.md), relationships, business rules.
**Red flags**: Entities mentioned in Features but not defined here. Generic property names
("data", "value") instead of domain terms. Missing relationships.

### Feature Specifications
Must have: every "Must" requirement from REQUIREMENTS.md with behavior, interface, acceptance
criteria, and error handling. "Should" requirements at minimum listed with behavior.
**Red flags**: Missing "Must" requirements. Acceptance criteria that are untestable.
Interface definitions that don't match KB conventions. Features that introduce entities
not in Domain Model.

### Non-Functional Requirements
Must have: observability (logging, metrics), testing strategy (coverage, test types),
accessibility and i18n if applicable.
**Red flags**: Copy-paste from REQUIREMENTS.md without spec-level detail. Missing testing
strategy. No reference to KB test-landscape.md patterns.

### Out of Scope
Must have: explicit exclusions from REQUIREMENTS.md. Future items clearly marked.
**Red flags**: Empty section. Missing items that REQUIREMENTS.md explicitly excluded.

### Revision History
Must have: at least the initial entry with date and source.
**Red flags**: Missing or empty.