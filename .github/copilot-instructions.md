# AI Content Authoring Instructions — TheDataStackDigest

These instructions guide AI Agents to create accurate, consistent, and high-quality content for this repository across technology articles and certification study guides.

## Repository Scope

- Content lives under:
  - `articles/` — technology deep-dives, tutorials, and series (e.g., `analytics-bi/apache-spark/`, `data-engineering-tools/`)
  - `certifications/` — exam guides and topic series (e.g., `dbt/dbt-architect-certification-exam/`, `databricks/data-engineer-professional/`)
- A general `ARTICLE_TEMPLATE.md` exists at the repo root. Use it when creating new articles unless a section-specific template overrides it.

## General Writing Principles

- Audience: Senior data engineers and practitioners.
- Voice: Concise, direct, actionable; avoid filler.
- Accuracy: Prefer official docs as sources, rewrite in your own words, and cite with inline links when appropriate.
- Hands-on first: Show realistic examples (SQL, Python, PySpark, YAML, Bash) before theory.
- Production-minded: Include deployment/operations, cost/perf trade-offs, and security considerations.

## File Placement and Naming

- Articles:
  - Place under the appropriate topic folder in `articles/`.
  - Use lowercase, hyphen-separated filenames, e.g., `apache-airflow-getting-started.md`.
  - For series, use `NN-` numeric prefixes to define reading order, e.g., `01-...`, `02-...`.

- Certifications:
  - Place under the correct vendor and certification path in `certifications/`.
  - Use `NN-<topic>.md` naming for topic modules, e.g., `06-monitoring-alerting.md`.
  - Include a `README.md` as the entry point with a navigable index.

## Article Authoring Checklist (use `ARTICLE_TEMPLATE.md`)

Each article should include:

1. Title and Overview
2. Prerequisites and Environment assumptions (cloud, tools, versions)
3. Core Concepts (what, why) — keep concise
4. Practical Walkthrough (copy-paste-ready steps and code)
5. Production Considerations (security, cost, reliability)
6. Troubleshooting (common failure modes, diagnostics)
7. Further Reading (official docs, high-signal references)

Example structure excerpt:

```markdown
# <Title>

## Overview

## Prerequisites

## Concepts

## Hands-on Walkthrough

## Production Considerations

## Troubleshooting

## References
```

## Certification Guide Authoring Checklist

For certification modules (topics), use this consistent structure:

1. Key concepts and definitions
2. Practical examples with code (SQL/YAML/Python/Bash as appropriate)
3. Best practices and anti-patterns
4. Real-world scenarios and solutions
5. Sample exam-style questions with explanations (5–8 questions)

Additional rules:
- Topic files must link back to the certification `README.md` and to adjacent topics.
- Use two-digit prefixes for ordering (e.g., `01-...`, `02-...`).
- Favor vendor-correct terminology; reconcile differences between platforms (e.g., Snowflake vs BigQuery vs Databricks).

## Code and Example Standards

- Always specify a language for fenced code blocks:
  - `sql`, `yaml`, `python`, `bash`, `json`, `scala` (for Spark), `markdown` (only when needed)
- When there is no specific language, use `shell`.
- Prefer runnable, minimal examples; include environment notes (e.g., Python version, dbt version).
- For Spark:
  - Provide both DataFrame and SQL examples when instructive.
  - Clarify cluster/runtime assumptions and partitioning/shuffle notes.
- For dbt:
  - Use `ref()`/`source()` correctly; demonstrate contracts, tests, and environment strategies.
  - Show `dbt_project.yml`, `schema.yml`, and job configuration examples where relevant.

## Markdown Quality and Linting (important)

Follow these rules to avoid common markdown issues:

- Blank lines around headings and lists:
  - Ensure one blank line before and after headings (MD022) and lists (MD032).
- Fenced code blocks:
  - Surround with blank lines (MD031) and include a language (MD040).
- Do not use emphasis as a heading (MD036): prefer `### Subheading` over `**Subheading**`.
- Keep a single H1 per document (MD025). Use H2+ for sections.
- When creating questions with alternatives, make sure to format them clearly with two spaces after each option, and a blank line after the question.
- Line endings and spacing:
  - No trailing spaces at the end of lines (MD009).
  - No multiple consecutive blank lines (MD012).
  - Files should end with a single newline (MD047).
- Links and references:
  - Use descriptive link text, avoid "click here" or bare URLs (MD034).
  - Reference-style links should be defined at the end of the document (MD052).
- Lists formatting:
  - Use consistent marker style within a list (MD004) - prefer `-` for unordered lists.
  - Avoid mixing ordered and unordered list markers (MD005).
  - Use proper indentation for nested lists (MD007) - 2 spaces per level.
- Emphasis and strong formatting:
  - Use consistent markers for emphasis: `*italic*` and `**bold**` (MD049, MD050).
  - Avoid using emphasis for entire sentences unless necessary.
- Table formatting:
  - Tables should have headers (MD043).
  - Table rows should have consistent column counts.
  - Use proper table alignment with pipes `|`.
- Heading structure:
  - Use incremental heading levels, don't skip (MD001) - no H1 → H3 jumps.
  - Avoid duplicate heading content in the same document (MD024).
  - Headings should not end with punctuation (MD026).

Quick example (good formatting):

```markdown
## Section Title

Intro paragraph.

- Item one
- Item two

```python
print("code with language and spacing")
```
```

## Cross-linking and Navigation

- Use relative links within the repo, e.g., `../README.md` or `./02-next-topic.md`.
- In each certification `README.md`, maintain an ordered index linking all topic files.
- In article series, add previous/next links at the end of each file if applicable.

## Topic Coverage Guidance (non-exhaustive)

- `articles/analytics-bi/apache-spark/` — architecture, DataFrames, optimization, memory/perf, deployment/ops.
- `articles/data-engineering-tools/` — orchestrators (e.g., Airflow), ingestion, testing, observability.
- `certifications/dbt/dbt-architect-certification-exam/` — connections, git, environments, jobs, security, monitoring, mesh, catalog.
- `certifications/databricks/data-engineer-professional/` — Python/SQL development, UDFs, LakeFlow, streaming, jobs, performance, security, governance, CI/CD.

When extending a topic:
- Keep naming/numbering consistent with existing files.
- Update the corresponding `README.md` index.

## Review and PR Workflow for Agents

1. Validate formatting:
   - Headings/lists spacing, code fence languages, and single H1 rule.
2. Validate examples:
   - Commands and code are syntactically correct; clearly marked assumptions.
3. Validate navigation:
   - All relative links resolve locally within the repo.
4. Summarize changes:
   - Provide a concise PR description with scope, sections added, and notable choices.

Suggested commit messages:

```text
docs(articles/spark): add memory/performance tuning guide with examples
docs(certs/dbt-architect): add 06 monitoring & alerting with webhooks
```

## Common Content Pitfalls to Avoid

- Overly generic “what is X” without actionable steps.
- Examples that omit environment/context, making them non-runnable.
- Public model advice without contracts, tests, or versioning in dbt contexts.
- Spark guidance without partitioning/shuffle or cost/perf notes.
- Markdown lint errors (spacing around headings/lists; missing code languages).

## Agent Execution Notes (for this repo)

- Use small, focused patches. Do not reformat unrelated sections.
- Preserve existing structure and naming conventions.
- When creating multiple related files (e.g., a new series topic), also update the relevant `README.md` index and add previous/next links.
- Prefer incremental PRs per topic/section over mega-PRs.

## Quick References

- Article template: `ARTICLE_TEMPLATE.md`
- Articles root: `articles/`
- Certifications root: `certifications/`
- dbt Architect guide: `certifications/dbt/dbt-architect-certification-exam/`
- Databricks DE Professional: `certifications/databricks/data-engineer-professional/`

By following these guidelines, agents will produce consistent, production-grade content that aligns with the structure and quality bar of TheDataStackDigest.
