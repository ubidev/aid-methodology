# Contributing to AID

AID is an open methodology — it improves through use. If you've run these phases in production and found something that works better, we want it here.

## What We Accept

### New Skills
- A skill for a phase variant not yet covered
- A specialized version of an existing skill (e.g., a wf-discover variant for monorepos, or for Python projects specifically)
- Skills that extend the methodology to edge cases (multi-repo, microservices, data science workflows)

### Improved Templates
- Better KB document templates with more concrete guidance
- New template variants (e.g., a task template for data pipeline tasks vs. API tasks)
- Example content that makes templates immediately usable

### Examples
- Anonymized real-world case studies — discovery outputs, task specs, review reports
- Examples from domains not yet covered (mobile, data science, infrastructure-as-code)
- Anti-pattern examples showing what NOT to do and why

### Methodology Feedback
- Phase descriptions that don't match production reality
- Missing feedback loops you've encountered
- Anti-patterns that deserve documenting
- Adoption challenges and how you solved them

## What We Don't Accept

- Tool-specific implementations (this repo is tool-agnostic)
- Skills that require specific proprietary services
- Examples with real client data, company names, or identifiable personal information
- Changes to the core methodology without discussion first (open an issue)

## How to Contribute

1. **Fork the repo** and create a branch: `git checkout -b your-contribution`

2. **For skill improvements:** Edit the relevant `skills/wf-*/SKILL.md`. Keep the same structure (Core Principle, Inputs, Process, Output, Feedback Loops). Don't add tool-specific CLI commands that only work in one environment.

3. **For new templates:** Add to the appropriate `templates/` subdirectory. Templates must include guidance comments explaining *why* each section exists, not just *what* to fill in.

4. **For examples:** Add to `examples/` with a `README.md` explaining the context (project type, team size, what was interesting about this case). **Anonymize everything** — no company names, no real URLs, no personal information.

5. **For methodology changes:** Open an issue first. Core methodology changes need discussion. Implementation improvements (templates, examples, skill clarity) don't.

6. **Submit a PR** with a clear description:
   - What changed and why
   - What phase(s) this affects
   - Whether this was tested in production (and roughly how)

## Style Guide

- **Tone:** Professional and practical. Opinionated. Methodology from someone who ships.
- **Language:** Active voice. Concrete over abstract. "Do X" not "X should be done."
- **Templates:** Include example content in `{curly braces}` for fill-in fields. Include guidance as `> blockquotes` or HTML comments for multi-line guidance.
- **Skills:** Start with the Core Principle (one paragraph, sharp). Then process steps. Then feedback loops. Then output definition.
- **No vendor lock-in:** Any specific tool (Claude Code, Codex, etc.) should be mentioned as an example, not a requirement.

## Anonymization Rules

If you're contributing examples from real projects:
- Replace company names with generic descriptions: "Enterprise Logistics Platform", "SaaS Analytics App"
- Replace team member names with roles: "the architect", "the product owner"
- Replace real URLs with example.com
- Replace real data with representative fake data
- If the client is identifiable from the description, change the context enough to break the link

We enforce this strictly. Real client data in a public repo creates legal and ethical problems.

## Questions?

Open an issue. We respond.
