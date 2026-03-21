# AID for Cursor

Use the `setup.sh` (or `setup.ps1` on Windows) script at the repo root to install AID into your project, or copy manually:

## Setup

```bash
# Automated (recommended)
path/to/aid-methodology/setup.sh /path/to/your/project

# Manual
cp -r path/to/aid-methodology/cursor/.cursor  .cursor/
cp path/to/aid-methodology/cursor/AGENTS.md   AGENTS.md
```

This gives you:
- `.cursor/rules/aid-methodology.mdc` — Always-on rule: KB integration and phase workflow
- `.cursor/rules/aid-review.mdc` — Code review standards (applied to source files)
- 11 phase-specific rules (invoked on demand)
- `AGENTS.md` — Project context for AI agents (edit with your project details)

## Rules

### Always-On Rules

#### `aid-methodology.mdc` (always applied)

Tells Cursor to:
- Read `knowledge/INDEX.md` before making changes
- Treat the Knowledge Base as the single source of truth
- Follow AID phases and produce artifacts at each gate

#### `aid-review.mdc` (applied to source files)

When Cursor reviews code it will:
- Check against task acceptance criteria
- Verify against `knowledge/coding-standards.md` and `knowledge/architecture.md`
- Grade A+ to F and tag issues by category

### Phase Rules (invoked on demand)

These rules contain the full AID phase instructions. Invoke them by referencing the rule name in your prompt (e.g., "use the aid-discover rule").

| Rule | Phase | Description |
|------|-------|-------------|
| `aid-discover.mdc` | Discovery | Brownfield codebase analysis → `knowledge/` directory |
| `aid-interview.mdc` | Interview | Adaptive requirements gathering → `REQUIREMENTS.md` |
| `aid-specify.mdc` | Specify | Requirements → formal `SPEC.md` grounded in KB |
| `aid-plan.mdc` | Plan | High-level roadmap → `PLAN.md` (MVP, modules, deliverables) |
| `aid-detail.mdc` | Detail | Decompose plan → user stories, `TASK-{id}.md` files, execution waves |
| `aid-implement.mdc` | Implement | Execute tasks with KB context, mandatory build verification |
| `aid-review-skill.mdc` | Review | Spec-anchored code review, A+ to F grading, auto-fix P1/P2 |
| `aid-test.mdc` | Test | Staging validation — E2E, integration, manual testing |
| `aid-deploy.mdc` | Deploy | Final verification, PR creation, delivery summary, KB updates |
| `aid-track.mdc` | Track | Production telemetry interpretation → `TRACK-REPORT.md` |
| `aid-triage.mdc` | Triage | Classify findings (BUG/CR/Infra), root cause analysis, routing |

### Phase Flow

```
Discovery → Interview → Specify → Plan → Detail → Implement → Review → Test → Deploy → Track → Triage
    ↑                                                                                          │
    └──────────────────────── feedback loops (GAP.md, IMPEDIMENT.md) ──────────────────────────┘
```

## Usage

1. Run `setup.sh` to install into your project.
2. Edit `AGENTS.md` with your project description, build commands, and conventions.
3. Run the Discovery phase (invoke `aid-discover`) to generate `knowledge/INDEX.md`.
4. Cursor will automatically apply the always-on rules on every edit.
5. Invoke phase rules as needed throughout the development lifecycle.

## Notes

- Cursor uses `.mdc` files in `.cursor/rules/` — these are Markdown with YAML frontmatter
- `alwaysApply: true` rules are injected into every conversation
- `alwaysApply: false` rules are available on demand (invoke by name)
- `globs:` rules are injected when matching files are open
- Human-readable phase documentation lives in the repo's `skills/` directory
- Templates for all artifacts live in `templates/`
- `aid-review.mdc` is the always-on lightweight review rule; `aid-review-skill.mdc` is the full review phase with grading
