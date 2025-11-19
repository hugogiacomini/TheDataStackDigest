# CLAUDE.md - AI Assistant Guide for TheDataStackDigest

This document provides essential context for AI assistants working on this repository. For detailed content authoring guidelines, see `.github/copilot-instructions.md`.

## Repository Overview

**TheDataStackDigest** is a community-driven knowledge hub for Data Engineering professionals. It contains:

- **Technical articles** with hands-on examples for practitioners
- **Certification study guides** for dbt and Databricks exams
- **Production-focused content** emphasizing real-world implementation

**Target audience**: Senior data engineers, certification candidates, and data platform professionals.

## Quick Reference

| Path | Purpose |
|------|---------|
| `articles/` | Technical deep-dives organized by category (8 subdirs) |
| `certifications/` | Exam prep materials (dbt, Databricks) |
| `ARTICLE_TEMPLATE.md` | Template for new articles |
| `CONTRIBUTING.md` | Contribution guidelines |
| `.github/copilot-instructions.md` | Detailed AI content authoring rules |

## Directory Structure

```
TheDataStackDigest/
├── articles/
│   ├── analytics-bi/
│   ├── apache-spark/           # 6-part series
│   ├── career-skills/
│   ├── data-architecture/
│   ├── data-engineering-tools/
│   ├── data-pipelines/
│   ├── data-quality/
│   └── data-storage/
└── certifications/
    ├── dbt/
    │   ├── analytics-engineer-certification-exam/
    │   └── dbt-architect-certification-exam/
    └── databricks/
        ├── associate-developer-for-apache-spark/
        ├── data-engineer-professional/
        └── generative-ai-engineer-associate/
```

## Content Patterns

### Articles

- **Location**: `articles/<category>/<filename>.md`
- **Naming**: Lowercase, hyphen-separated (e.g., `apache-airflow-getting-started.md`)
- **Series**: Use `NN-` numeric prefixes (e.g., `01-spark-architecture...`, `02-advanced-dataframe...`)

### Certifications

- **Location**: `certifications/<vendor>/<certification>/<topic>.md`
- **Naming**: `NN-<topic-name>.md` or `NN-<topic-name>/` directories
- **Each certification has**: `README.md` (index), topic modules with sample questions

## Key Conventions

### File Structure

**Articles must include**:

1. Title with overview
2. Prerequisites and environment assumptions
3. Core concepts (concise)
4. Practical walkthrough with code
5. Production considerations (security, cost, reliability)
6. Troubleshooting section
7. References/Further reading

**Certification modules must include**:

1. Key concepts and definitions
2. Practical examples with code
3. Best practices and anti-patterns
4. Real-world scenarios
5. 5-8 sample exam questions with explanations
6. Navigation links to adjacent topics

### Code Examples

- **Always specify language** in fenced blocks: `sql`, `python`, `yaml`, `bash`, `scala`, `json`
- **Make examples runnable** with environment notes (versions, setup)
- **Include both DataFrame and SQL** for Spark when instructive
- **Show production context**: partitioning, shuffle, cost implications

### Markdown Standards

Follow these rules to avoid lint errors:

- Blank lines around headings and lists
- Blank lines around fenced code blocks
- Single H1 per document
- Incremental heading levels (no H1 → H3 jumps)
- Use `### Subheading` not `**Subheading**`
- No trailing spaces
- Files end with single newline
- Use `-` for unordered lists consistently
- 2 spaces per indentation level for nested lists

### Navigation

- Use **relative links** within the repo: `../README.md`, `./02-next-topic.md`
- Maintain **ordered index** in each certification README
- Add **previous/next links** at end of series articles

## Development Workflow

### Creating New Content

1. Place in appropriate directory following naming conventions
2. Use `ARTICLE_TEMPLATE.md` for articles
3. Follow certification module structure for exam content
4. Update relevant `README.md` index when adding certification topics
5. Add previous/next navigation links for series content

### Commit Messages

Use conventional format:

```
docs(articles/spark): add memory/performance tuning guide
docs(certs/dbt-architect): add 06 monitoring & alerting module
```

### Pull Request Workflow

1. Validate markdown formatting (headings, code fences, spacing)
2. Verify all code examples are syntactically correct
3. Check all relative links resolve
4. Provide concise PR description with scope and notable choices

## Common Tasks

### Adding an Article

```bash
# 1. Create file in appropriate category
articles/<category>/<topic-name>.md

# 2. Use ARTICLE_TEMPLATE.md structure
# 3. Include code examples with language tags
# 4. Add production considerations
```

### Adding Certification Content

```bash
# 1. Create topic file with numeric prefix
certifications/<vendor>/<cert>/NN-<topic>.md

# 2. Follow 5-part module structure
# 3. Include 5-8 sample questions
# 4. Update certification README.md index
# 5. Add navigation links
```

### Extending a Series

1. Follow existing `NN-` numbering pattern
2. Update series outline file (e.g., `00-series-outline-*.md`)
3. Add previous/next links to adjacent files

## Technology Coverage

### Primary Focus Areas

- **Apache Spark**: Architecture, DataFrames, optimization, memory/perf, deployment
- **dbt**: Models, tests, contracts, environments, jobs, security, monitoring
- **Databricks**: Python/SQL dev, streaming, LakeFlow, jobs, governance, MLOps
- **Data Engineering Tools**: Airflow, orchestration, ingestion, observability

### Code Languages Used

- Python (PySpark)
- SQL
- Scala (Spark)
- YAML (dbt, configs)
- Bash (CLI commands)

## Content Quality Standards

### Do

- Provide realistic, runnable examples
- Include environment/version context
- Show production considerations (cost, security, reliability)
- Use vendor-correct terminology
- Add troubleshooting for common failure modes

### Don't

- Write generic "what is X" without actionable steps
- Omit environment context making examples non-runnable
- Skip partitioning/shuffle notes for Spark content
- Forget contracts/tests for dbt public models
- Use emphasis (`**text**`) as headings

## Agent Execution Guidelines

When working on this repository:

1. **Use small, focused patches** - don't reformat unrelated sections
2. **Preserve existing structure** - follow established naming/numbering
3. **Update indexes** when adding content - certification READMEs, series outlines
4. **Prefer incremental PRs** - one topic/section per PR
5. **Validate formatting** before committing - check markdown lint rules

## Useful Commands

```bash
# Find all certification topics
ls certifications/*/*/

# Find all articles
ls articles/*/

# Check for markdown files missing code language tags
grep -r '```$' articles/ certifications/
```

## References

- **Article template**: `ARTICLE_TEMPLATE.md`
- **Contribution guidelines**: `CONTRIBUTING.md`
- **Detailed AI instructions**: `.github/copilot-instructions.md`
- **License**: MIT (2024)
