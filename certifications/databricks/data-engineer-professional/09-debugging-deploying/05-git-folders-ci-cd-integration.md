# Git Folders CI/CD Integration for Notebooks and Code

## Overview

Git Folders let you connect a workspace directory to a Git repository to develop notebooks and code with standard Git workflows. Combined with Bundles or Jobs APIs, this enables true Git-based CI/CD.

## Setup

- Connect Git (User Settings → Git integration) with a personal access token or enterprise app.
- In Workspace, create a Git Folder and link to your repository/branch.
- Use pull requests to review changes; use CI to validate and deploy.

## Typical Workflow

1. Create feature branch in Git Folder.  
2. Develop notebooks/code; commit and push.  
3. Open PR; CI runs tests (`pytest`, SQL unit tests, pipeline dry-runs).  
4. Merge to main; CD deploys via Bundles or Jobs API.

## Commands and Automation

```bash
# Databricks CLI can pull latest content in Git Folders (via workspace import/export)
# Export for CI validation
databricks workspace export_dir \
  --overwrite \
  "/Repos/org/project" \
  ./repo_export

# Run tests (example)
pytest -q

# Deploy updated resources through Bundles
databricks bundle deploy --target prod
```

## Best Practices

- Keep notebooks deterministic; parameterize via widgets or job params.
- Separate dev vs prod repos or branches; protect prod.
- Enforce code owners and PR checks for sensitive areas.

## Troubleshooting

- Desync between workspace and remote → use Pull Latest or re-link Git Folder.
- Credential errors → refresh token/app; verify repository permissions.
- Merge conflicts in notebooks → prefer source control–friendly formats or modular Python files.

## Mock Questions

1. **What is the primary benefit of Git Folders in Databricks?**  
    a. Auto-scaling clusters  
    b. Git-based development of notebooks/code with PR workflows  
    c. Cheaper storage  
    d. Automatic job retries

2. **How should production deployments be triggered after merging to main?**  
    a. Manual notebook runs  
    b. Workspace export to DBFS  
    c. CI pipeline invoking Bundles or Jobs API  
    d. Cluster restart

3. **Which practice reduces merge conflicts in notebooks?**  
    a. Commit checkpoints  
    b. Use large monolithic notebooks  
    c. Prefer modular Python files and smaller notebooks  
    d. Disable PR reviews

## Answers

1. b
2. c
3. c

## References

- Databricks Git Folders documentation
- Databricks Repos and CI/CD guides
