# Topic 4: Creating and Maintaining Job Definitions

## Overview

Jobs in dbt Cloud automate the execution of dbt commands on a schedule or trigger. Proper job configuration ensures reliable, efficient data transformation pipelines that integrate seamlessly with your development workflow.

## Key Concepts and Definitions

### What is a dbt Job?

A **job** in dbt Cloud is a collection of dbt commands that run in a specific environment on a defined schedule or trigger. Jobs enable:

- Scheduled production runs
- Continuous integration testing
- Documentation generation
- Automated data quality checks
- Event-driven transformations

### Job Types

#### 1. Production Jobs

- Run on schedule (cron-based)
- Build and materialize production models
- Update documentation
- Execute on stable branches (main, production)

#### 2. CI Jobs (Continuous Integration)

- Triggered by pull requests
- Test changes before merging
- Run only modified models (with deferral)
- Provide fast feedback to developers

#### 3. Custom/API-Triggered Jobs

- Triggered via API or webhook
- Event-driven execution
- Integration with external orchestrators
- Manual execution for specific scenarios

### Job Components

#### Execution Steps

Ordered list of dbt commands to run:

- `dbt deps` - Install dependencies
- `dbt seed` - Load seed files
- `dbt run` - Build models
- `dbt test` - Run tests
- `dbt snapshot` - Capture snapshots
- `dbt docs generate` - Create documentation

#### Job Settings

- **Environment**: Which environment to run in
- **Commands**: What dbt commands to execute
- **Triggers**: When to run (schedule, PR, webhook, manual)
- **Threads**: Parallel execution count
- **Target**: Custom target name
- **Deferral**: Reference another job's artifacts

## Practical Examples

### Example 1: Basic Production Job

```markdown
Navigate to: Jobs → Create Job

General Settings:
  Job Name: Daily Production Run
  Environment: Production
  
Commands:
  1. dbt deps
  2. dbt seed
  3. dbt run
  4. dbt test

Triggers:
  Schedule: 
    ☑ Run on schedule
    Cron: 0 6 * * *  # 6 AM daily
    Timezone: America/New_York
    
Settings:
  Target Name: prod
  Threads: 8
  Generate docs on run: ☑ Enabled
```

### Example 2: CI Job with Deferral

```markdown
Job Configuration:

General Settings:
  Job Name: CI Check
  Environment: CI
  
Execution Settings:
  Commands:
    1. dbt deps
    2. dbt build --select state:modified+ --defer --state ./prod-artifacts
    
Triggers:
  ☑ Run on Pull Requests
  Git Provider: GitHub
  
Deferral:
  ☑ Defer to another job
  Defer to job: Daily Production Run
  
Advanced Settings:
  Run timeout: 30 minutes
  Threads: 4
  Target name: ci
```

**What happens when a PR is created:**

```bash
# 1. Webhook from GitHub triggers job
# 2. dbt compares current branch to production manifest
# 3. Identifies modified models and their downstream dependencies
# 4. Runs only those models, deferring to production for everything else
# 5. Reports status back to PR

# Example output:
# Modified models: dim_customers (and 3 downstream models)
# Deferred models: 142 models from production
# Models to run: 4
# Build time: 2 minutes (vs 30 minutes for full build)
```

### Example 3: Job Chaining

```markdown
Job 1: Data Ingestion
  Schedule: 0 4 * * *  # 4 AM
  Commands:
    - dbt run --select tag:ingestion
    
Job 2: Data Transformation
  Schedule: None
  Triggered by: Job 1 completion
  Commands:
    - dbt run --select tag:transformation
    
Job 3: Metrics & Reporting
  Schedule: None
  Triggered by: Job 2 completion
  Commands:
    - dbt run --select tag:reporting
    - dbt docs generate
```

**Configuration in UI:**

```markdown
Job 2 Settings:
  Triggers:
    ☑ Triggered by job completion
    Job: Data Ingestion
    On states: Success
    
Job 3 Settings:
  Triggers:
    ☑ Triggered by job completion
    Job: Data Transformation
    On states: Success
```

### Example 4: Advanced CI Configuration

```markdown
Advanced CI Job:

General Settings:
  Job Name: Advanced CI - Smart Build
  Environment: CI
  
Execution:
  Commands:
    1. dbt deps
    2. dbt run --select state:modified+ --defer --state ./prod-artifacts
    3. dbt test --select state:modified+ --defer --state ./prod-artifacts
    4. dbt build --select result:fail --defer --state ./prod-artifacts
    
Triggers:
  ☑ Run on Pull Requests
  ☑ Run on Draft Pull Requests: Disabled
  
Deferral:
  ☑ Defer to another job
  Defer to: Daily Production Run
  Self-defer: Disabled
  
Slim CI Settings:
  Compare changes against: Production environment
  
Advanced Settings:
  Run timeout: 45 minutes
  Threads: 4
  Run on PR events:
    ☑ opened
    ☑ synchronize
    ☑ reopened
    ☐ labeled
```

### Example 5: Documentation Generation Job

```markdown
Job Configuration:

General Settings:
  Job Name: Update Documentation
  Environment: Production
  Description: Generates and publishes project documentation
  
Commands:
  1. dbt deps
  2. dbt docs generate
  
Triggers:
  Schedule: 0 */6 * * *  # Every 6 hours
  
Settings:
  Generate docs on run: ☑ Enabled
  Run on source freshness error: Skip
  
Notification:
  ☑ Email on failure
  Recipients: data-team@company.com
```

### Example 6: Self-Deferral for Incremental Efficiency

```markdown
Production Job with Self-Deferral:

General Settings:
  Job Name: Hourly Incremental Update
  Environment: Production
  
Commands:
  1. dbt run --select config.materialized:incremental
  
Triggers:
  Schedule: 0 * * * *  # Every hour
  
Deferral:
  ☑ Self-defer
  Description: Reference artifacts from previous successful run
  
Result:
  - First run: Builds all incremental models from scratch
  - Subsequent runs: Only process new/changed data
  - Significantly reduced runtime and costs
```

### Example 7: Managing Jobs via API

```python
import requests

DBT_CLOUD_API_TOKEN = "your_token_here"
ACCOUNT_ID = "12345"
BASE_URL = "https://cloud.getdbt.com/api/v2"

headers = {
    "Authorization": f"Token {DBT_CLOUD_API_TOKEN}",
    "Content-Type": "application/json"
}

# Create a new job
job_config = {
    "id": None,
    "account_id": ACCOUNT_ID,
    "project_id": 67890,
    "environment_id": 11111,
    "name": "Nightly Production Build",
    "dbt_version": None,  # Use environment default
    "execute_steps": [
        "dbt deps",
        "dbt seed",
        "dbt run",
        "dbt test"
    ],
    "settings": {
        "threads": 8,
        "target_name": "prod"
    },
    "schedule": {
        "cron": "0 2 * * *",
        "date": {
            "type": "every_day"
        },
        "time": {
            "type": "at_exact_time",
            "interval": 1,
            "hours": 2,
            "minutes": 0
        }
    },
    "triggers": {
        "github_webhook": False,
        "schedule": True
    }
}

response = requests.post(
    f"{BASE_URL}/accounts/{ACCOUNT_ID}/jobs/",
    headers=headers,
    json=job_config
)

if response.status_code == 201:
    job_id = response.json()["data"]["id"]
    print(f"Job created successfully. Job ID: {job_id}")
else:
    print(f"Error creating job: {response.text}")

# Trigger a job run
trigger_response = requests.post(
    f"{BASE_URL}/accounts/{ACCOUNT_ID}/jobs/{job_id}/run/",
    headers=headers,
    json={"cause": "API trigger - manual execution"}
)

if trigger_response.status_code == 200:
    run_id = trigger_response.json()["data"]["id"]
    print(f"Job triggered. Run ID: {run_id}")
```

## Best Practices

### 1. Order Commands Correctly

**Correct order:**

```markdown
1. dbt deps        # Install packages first
2. dbt seed        # Load seed data before models reference them
3. dbt snapshot    # Capture state changes
4. dbt run         # Build models
5. dbt test        # Test after models are built
6. dbt docs generate  # Document last
```

**Why this matters:**

- Dependencies must be available before use
- Models must exist before testing
- Order prevents unnecessary failures

### 2. Use dbt build for Simplicity

```markdown
Instead of:
  1. dbt deps
  2. dbt seed
  3. dbt run
  4. dbt test
  
Use:
  1. dbt deps
  2. dbt build

Benefits:
  - Respects full DAG order
  - Runs seeds, models, snapshots, and tests in correct sequence
  - Stops execution on first failure
  - Simpler configuration
```

### 3. Implement Proper CI Deferral

```markdown
Good CI Configuration:
  Command: dbt build --select state:modified+ --defer
  Defer to: Production job
  
Benefits:
  - Only builds changed models
  - Fast feedback (minutes vs hours)
  - Lower compute costs
  - Encourages frequent commits
```

### 4. Configure Appropriate Timeouts

```markdown
Job Type: Timeout Setting

Development/CI: 30-60 minutes
  - Fast feedback is important
  - Long-running CI discourages testing

Production (Full Build): 4-8 hours
  - Complete builds may take longer
  - Balance reliability vs detection time

Production (Incremental): 1-2 hours
  - Incremental updates should be fast
  - Longer timeout suggests issues
```

### 5. Use Job Chaining for Complex Workflows

```markdown
Scenario: Multi-stage data pipeline

❌ Bad Approach: One massive job
  Commands:
    - dbt run (runs everything, takes 4 hours)
  Problem: 
    - No visibility into pipeline stages
    - Failures require full rerun
    - Can't optimize individual stages

✅ Good Approach: Chained jobs
  Job 1: Staging (30 min)
    - dbt run --select tag:staging
    
  Job 2: Intermediate (1 hour)
    - dbt run --select tag:intermediate
    Trigger: After Job 1 success
    
  Job 3: Marts (1 hour)
    - dbt run --select tag:marts
    Trigger: After Job 2 success
    
  Benefits:
    - Clear pipeline stages
    - Parallel execution where possible
    - Targeted reruns on failure
    - Better monitoring and alerting
```

### 6. Environment Variable Overrides

```markdown
Job Settings:
  Environment Variables:
    Override from environment:
      DBT_TARGET_SCHEMA = "ADHOC_ANALYSIS"
      ENABLE_DEBUG_LOGGING = "true"
      
Use case: 
  - Run same job with different config
  - A/B testing new logic
  - One-off analysis runs
```

### 7. Version Pinning Strategy

```markdown
dbt Version Configuration:

Production Jobs:
  ☑ Use specific version: 1.7.4
  Reason: Stability, predictability

Staging Jobs:
  ☑ Use latest 1.7.x
  Reason: Test patch updates before production

Development:
  ☑ Use versionless (latest stable)
  Reason: Stay current with new features
```

## Anti-Patterns (What NOT to Do)

### ❌ DON'T Run Everything in CI

```markdown
# BAD: Full build in CI
CI Job Commands:
  - dbt deps
  - dbt build  # Runs ALL 500 models, takes 45 minutes

Problem:
  - Slow feedback discourages PRs
  - Expensive compute costs
  - Defeats purpose of CI
```

**Solution:**

```markdown
# GOOD: Slim CI with deferral
CI Job Commands:
  - dbt deps
  - dbt build --select state:modified+ --defer

Result:
  - Only runs changed models (typically 2-10 models)
  - Fast feedback (2-5 minutes)
  - Lower costs
```

### ❌ DON'T Skip dbt deps

```markdown
# BAD: Forgetting dependencies
Job Commands:
  - dbt run
  - dbt test

Error: "Could not find package 'dbt_utils' in packages.yml"
```

**Solution:**

```markdown
# GOOD: Always install dependencies first
Job Commands:
  - dbt deps
  - dbt run
  - dbt test
```

### ❌ DON'T Use Overly Complex Schedules

```markdown
# BAD: Complex, hard-to-understand schedule
Schedule: "5,15,25,35,45,55 2-6 * * 1-5"
  What does this even mean?

# GOOD: Clear, simple schedules
Schedule: "0 6 * * *"
Description: Daily at 6 AM UTC
```

### ❌ DON'T Chain Jobs in Circle

```markdown
# BAD: Circular dependency
Job A → Triggers Job B on success
Job B → Triggers Job C on success
Job C → Triggers Job A on success

Result: Infinite loop, resource exhaustion
```

### ❌ DON'T Ignore Failed Tests in Production

```markdown
# BAD: Run even if tests fail
Job Settings:
  Run on test failure: Continue
  
# GOOD: Stop on test failure
Job Settings:
  Run on test failure: Cancel
  
Reason: 
  - Tests exist to catch data quality issues
  - Continuing builds broken data into production
  - Creates technical debt
```

### ❌ DON'T Use Same Job for Different Purposes

```markdown
# BAD: Overloaded job
Job Name: "Production Run and CI and Docs"
Triggers:
  - Schedule: Daily at 6 AM
  - Pull Requests
  - Manual
  - Webhook from external system
  
Problem:
  - Hard to maintain
  - Mixed concerns
  - Difficult to debug
```

**Solution:**

```markdown
# GOOD: Separate jobs per purpose
- Production Job: Scheduled daily runs
- CI Job: PR validation
- Docs Job: Documentation updates
- Webhook Job: Event-driven triggers
```

## Real-World Scenarios and Solutions

### Scenario 1: Implementing Zero-Downtime Deployments

**Challenge:** Deploy dbt changes to production without causing downtime or stale data.

#### Solution: Blue-Green Deployment Pattern

```markdown
Setup:
  - Environment 1 (Blue): Currently serving production
  - Environment 2 (Green): Deploy updates here

Deployment Process:

1. Pre-Deployment Job (Green Environment)
   Commands:
     - dbt deps
     - dbt build --exclude config.materialized:snapshot
     - dbt test
   Schedule: Off
   Trigger: Manual before deployment
   
2. Smoke Test Job
   Commands:
     - dbt test --select tag:smoke_test
   Trigger: After pre-deployment success
   
3. Cutover (Manual Process)
   - Update downstream systems to point to Green
   - Monitor for 24 hours
   
4. Production Job (Now Green)
   Commands:
     - dbt deps
     - dbt build
   Schedule: Normal schedule
   
5. Decommission Blue (After validation)
   - Keep as backup for 1 week
   - Archive and remove
```

### Scenario 2: Handling Long-Running Models

**Challenge:** Some models take hours to run, blocking other updates.

#### Solution: Separate Fast and Slow Jobs

```markdown
Job 1: Fast Models (Hourly)
  Commands:
    - dbt run --select tag:fast
    - dbt test --select tag:fast
  Schedule: 0 * * * *  # Every hour
  Threads: 4
  Timeout: 30 minutes
  
Job 2: Slow Models (Daily)
  Commands:
    - dbt run --select tag:slow
    - dbt test --select tag:slow
  Schedule: 0 2 * * *  # 2 AM daily
  Threads: 16  # More resources for heavy lifting
  Timeout: 6 hours
  
Job 3: Incremental Models (Every 15 minutes)
  Commands:
    - dbt run --select config.materialized:incremental
  Schedule: */15 * * * *
  Threads: 2
  Self-defer: Enabled
  Timeout: 20 minutes
```

### Scenario 3: Multi-Region Deployment

**Challenge:** Deploy same dbt project to multiple geographic regions with region-specific configurations.

**Solution:**

```markdown
Project Structure:
  - Single Git repository
  - Multiple dbt Cloud environments (one per region)

US Production Job:
  Environment: US-Production
  Environment Variables:
    REGION = "US"
    DBT_DATABASE = "ANALYTICS_US"
    DATA_RESIDENCY_ZONE = "us-east-1"
  Schedule: 0 6 * * *  # 6 AM ET
  
EU Production Job:
  Environment: EU-Production
  Environment Variables:
    REGION = "EU"
    DBT_DATABASE = "ANALYTICS_EU"
    DATA_RESIDENCY_ZONE = "eu-west-1"
  Schedule: 0 6 * * *  # 6 AM CET
  
APAC Production Job:
  Environment: APAC-Production
  Environment Variables:
    REGION = "APAC"
    DBT_DATABASE = "ANALYTICS_APAC"
    DATA_RESIDENCY_ZONE = "ap-southeast-1"
  Schedule: 0 6 * * *  # 6 AM SGT
```

### Scenario 4: Cost Optimization with Selective Runs

**Challenge:** Full dbt runs are expensive; most models don't need hourly updates.

#### Solution: Tiered Update Strategy

```markdown
Tier 1: Real-time (Every 15 minutes)
  Models: Revenue, active users, critical KPIs
  Commands:
    - dbt run --select tag:tier1
    - dbt test --select tag:tier1
  Cost: High, but justified by business value

Tier 2: Frequent (Hourly)
  Models: Dashboards, operational reports
  Commands:
    - dbt run --select tag:tier2
    - dbt test --select tag:tier2
  Cost: Medium

Tier 3: Daily (Once per day)
  Models: Historical analysis, trends
  Commands:
    - dbt run --select tag:tier3
    - dbt test --select tag:tier3
  Cost: Low

Tier 4: Weekly (Sunday nights)
  Models: Long-term analysis, archives
  Commands:
    - dbt run --select tag:tier4
    - dbt test --select tag:tier4
  Cost: Minimal
```

### Scenario 5: Debugging Failed CI Jobs

**Challenge:** CI job fails intermittently, but runs succeed in development.

**Troubleshooting Process:**

```markdown
Step 1: Check Recent Runs
  - Look for patterns (time of day, specific models)
  - Review error messages
  - Compare successful vs failed runs

Step 2: Review CI Job Configuration
  - Verify deferral setup
  - Check timeout settings
  - Confirm environment variables
  - Validate credentials

Step 3: Test Locally with CI Conditions
  dbt run --select state:modified+ --defer --state ./prod-artifacts
  
Step 4: Common Issues & Solutions

Issue: "Could not find relation"
  Cause: Deferral not configured properly
  Solution: Ensure "Defer to" job is set and has artifacts

Issue: Timeout after 30 minutes
  Cause: Too many models selected
  Solution: Increase timeout or optimize selection criteria

Issue: "Permission denied"
  Cause: CI environment lacks write permissions
  Solution: Grant appropriate permissions to CI service account

Step 5: Add Debugging
  Enable debug logging:
    dbt --debug run --select state:modified+
  
  Add explicit logging to models:
    {{ log("Processing model: " ~ this, info=true) }}
```

## Sample Exam Questions

### Question 1: Command Order

**Question:** In what order should the following dbt commands be executed in a production job?

1. dbt test  
2. dbt run  
3. dbt seed  
4. dbt deps  

A) 4, 3, 2, 1  
B) 4, 2, 3, 1  
C) 1, 2, 3, 4  
D) 3, 4, 2, 1

**Answer:** A) 4, 3, 2, 1

**Explanation:**

- `dbt deps` (4) must run first to install packages
- `dbt seed` (3) loads CSVs that models may reference
- `dbt run` (2) builds models
- `dbt test` (1) runs tests on built models

### Question 2: CI Job Configuration

**Question:** What is the PRIMARY benefit of configuring a CI job to defer to a production job?

A) It ensures the CI job uses production credentials  
B) It reduces CI runtime by referencing existing production models instead of rebuilding them  
C) It automatically promotes code to production after CI passes  
D) It prevents CI jobs from running if production jobs are running

**Answer:** B) It reduces CI runtime by referencing existing production models instead of rebuilding them

**Explanation:**

- Deferral allows CI to reference production artifacts for unchanged models
- Only modified models and their dependents are built in CI
- This dramatically reduces CI runtime and cost
- CI and production use separate credentials (A is incorrect)

### Question 3: Job Chaining

**Question:** You have three jobs that must run in sequence: Job A, Job B, and Job C. Job B should only run if Job A succeeds. How should you configure this?

A) Set Job B's schedule to run 1 hour after Job A  
B) Configure Job B's trigger to "Run after job completion" with Job A specified and "On success" selected  
C) Use the same schedule for all jobs  
D) Configure all jobs to run in the same environment

**Answer:** B) Configure Job B's trigger to "Run after job completion" with Job A specified and "On success" selected

**Explanation:**

- Job chaining feature explicitly handles sequential execution
- "On success" ensures Job B only runs if Job A completes successfully
- Scheduling with time delays (A) is fragile and doesn't handle variable job durations
- Same schedule (C) would cause parallel execution, not sequential

### Question 4: Self-Deferral

**Question:** When would you use self-deferral in a job configuration?

A) When you want the job to skip execution if it's already running  
B) When you want incremental models to reference the previous successful run's artifacts  
C) When you want the job to automatically retry on failure  
D) When you want to defer to the same job in a different environment

**Answer:** B) When you want incremental models to reference the previous successful run's artifacts

**Explanation:**

- Self-deferral allows a job to reference its own previous artifacts
- Particularly useful for incremental models that need to know what data was processed last time
- Improves efficiency by avoiding unnecessary reprocessing
- Not related to concurrent execution prevention (A) or retries (C)

### Question 5: Environment Variable Override

**Question:** You have a job that normally writes to `PROD` schema, but you want to run it once to write to `TEST` schema without changing the job configuration permanently. What should you use?

A) Change the environment's default schema setting  
B) Use a job-level environment variable override for `DBT_TARGET_SCHEMA`  
C) Create a new environment with the TEST schema  
D) Modify the dbt_project.yml file in Git

**Answer:** B) Use a job-level environment variable override for `DBT_TARGET_SCHEMA`

**Explanation:**

- Job-level environment variable overrides allow one-time configuration changes
- Doesn't affect other jobs or future runs
- No need to modify environment settings (A) or create new environments (C)
- Doesn't require code changes in Git (D)

### Question 6: Documentation Generation

**Question:** You want your project documentation to update automatically every time your production job runs. What setting should you enable?

A) Run dbt docs generate in the job commands  
B) Enable "Generate docs on run" in job settings  
C) Schedule a separate documentation job  
D) Enable documentation in the environment settings

**Answer:** B) Enable "Generate docs on run" in job settings

**Explanation:**

- "Generate docs on run" automatically generates docs after successful job completion
- Simpler than manually adding `dbt docs generate` command (A)
- No need for separate job (C)
- Documentation is a job setting, not environment setting (D)

## Summary

Proper job configuration is essential for automated, reliable dbt workflows. Key takeaways:

1. **Order commands correctly:**
   - Dependencies first (dbt deps)
   - Seeds before models
   - Tests after models

2. **Use appropriate job types:**
   - Production jobs for scheduled builds
   - CI jobs with deferral for fast PR validation
   - Custom jobs for special workflows

3. **Leverage deferral:**
   - Regular deferral for CI jobs
   - Self-deferral for incremental efficiency

4. **Chain jobs** for complex multi-stage pipelines

5. **Configure appropriate timeouts** based on job type and expectations

6. **Use `dbt build`** for simplified execution

7. **Monitor and optimize** job performance regularly

## Next Steps

Continue to [Topic 5: Configuring dbt Security and Licenses](./05-security-licenses.md) to learn about access control and license management.
