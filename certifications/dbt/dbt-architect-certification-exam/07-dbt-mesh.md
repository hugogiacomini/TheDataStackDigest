# Topic 7: Setting up a dbt Mesh and Leveraging Cross-Project References

## Overview

dbt Mesh is an architectural pattern that enables multiple teams to develop data products independently while maintaining dependencies and governance across projects. This approach supports scalable, modular data platforms with clear ownership boundaries.

## Key Concepts and Definitions

### What is dbt Mesh?

A distributed architecture where:

- Multiple dbt projects exist within an organization
- Projects reference models from other projects
- Each project maintains its own repository, schedules, and ownership
- Central governance ensures consistency and discoverability

### Core Components

#### 1. Projects

Independent dbt codebases with their own:

- Git repository
- Development and deployment environments
- Job schedules
- Access controls
- Dependencies

#### 2. Cross-Project References

Mechanism to reference models from other projects:

- Use `ref()` function with project syntax
- Creates dependency graphs across projects
- Enables modular development
- Maintains lineage

#### 3. Public Models

Models explicitly marked as consumable by other projects:

- Defined in `models/contracts`
- Have stable interfaces (contracts)
- Follow versioning practices
- Include comprehensive documentation

#### 4. Model Governance

Controls and standards for cross-project dependencies:

- Access policies
- Versioning strategies
- Breaking change management
- Documentation requirements

### Environment Relationships

Cross-project references consider environment types:

- **Development**: References development state of upstream projects
- **Staging**: References staging deployment of upstream projects
- **Production**: References production deployment of upstream projects

## Practical Examples

### Example 1: Basic Multi-Project Setup

**Organizational Structure:**

```
Organization: Acme Corp
├── Project: core-data (foundational models)
│   ├── Repository: github.com/acme/dbt-core-data
│   ├── Environments:
│   │   ├── Development
│   │   ├── Staging
│   │   └── Production
│   └── Public Models:
│       ├── dim_customers
│       ├── dim_products
│       └── fct_orders
│
├── Project: finance-analytics (finance team)
│   ├── Repository: github.com/acme/dbt-finance
│   ├── Depends on: core-data
│   └── Models:
│       ├── revenue_by_customer (uses core-data.fct_orders)
│       └── product_margins (uses core-data.dim_products)
│
└── Project: marketing-analytics (marketing team)
    ├── Repository: github.com/acme/dbt-marketing
    ├── Depends on: core-data
    └── Models:
        ├── customer_segments (uses core-data.dim_customers)
        └── campaign_performance (uses core-data.fct_orders)
```

**Project Configuration:**

```yaml
# core-data/dbt_project.yml
name: core_data
version: '1.0.0'
profile: 'snowflake'

models:
  core_data:
    marts:
      +access: public  # Mark all marts as public
      +contract:
        enforced: true
```

```yaml
# finance-analytics/dbt_project.yml
name: finance_analytics
version: '1.0.0'
profile: 'snowflake'

dependencies:
  - project: core_data
```

```yaml
# finance-analytics/dependencies.yml
projects:
  - name: core_data
    git: https://github.com/acme/dbt-core-data.git
```

### Example 2: Cross-Project Reference Usage

**In Upstream Project (core-data):**

```sql
-- models/marts/dim_customers.sql
{{
  config(
    materialized='table',
    access='public',
    contract={'enforced': true}
  )
}}

-- Define contract
{{ config(
  contract={
    'enforced': true,
    'schema': [
      {'name': 'customer_id', 'data_type': 'integer'},
      {'name': 'customer_name', 'data_type': 'varchar'},
      {'name': 'email', 'data_type': 'varchar'},
      {'name': 'created_at', 'data_type': 'timestamp'}
    ]
  }
)}}

select
  customer_id,
  customer_name,
  email,
  created_at
from {{ source('raw', 'customers') }}
```

```yaml
# models/marts/dim_customers.yml
version: 2

models:
  - name: dim_customers
    description: "Core customer dimension - stable interface for all downstream projects"
    access: public
    config:
      contract:
        enforced: true
    columns:
      - name: customer_id
        description: "Unique customer identifier"
        data_type: integer
      - name: customer_name
        description: "Full customer name"
        data_type: varchar
      - name: email
        description: "Customer email address"
        data_type: varchar
      - name: created_at
        description: "Customer account creation timestamp"
        data_type: timestamp
```

**In Downstream Project (finance-analytics):**

```sql
-- models/revenue_by_customer.sql
with customers as (
  -- Cross-project reference
  select * from {{ ref('core_data', 'dim_customers') }}
),

orders as (
  -- Another cross-project reference
  select * from {{ ref('core_data', 'fct_orders') }}
),

aggregated as (
  select
    c.customer_id,
    c.customer_name,
    c.email,
    sum(o.order_amount) as total_revenue,
    count(o.order_id) as order_count
  from customers c
  left join orders o on c.customer_id = o.customer_id
  group by 1, 2, 3
)

select * from aggregated
```

### Example 3: Setting Up Project Dependencies in dbt Cloud

**Step 1: Create Upstream Project (core-data)**

```markdown
dbt Cloud Console:
1. Account Settings → Projects → New Project
2. Project Name: Core Data
3. Connect Repository: github.com/acme/dbt-core-data
4. Configure warehouse connection
5. Create environments:
   - Development (branch: main)
   - Production (branch: main)
```

**Step 2: Create Downstream Project (finance-analytics)**

```markdown
dbt Cloud Console:
1. Account Settings → Projects → New Project
2. Project Name: Finance Analytics
3. Connect Repository: github.com/acme/dbt-finance
4. Configure warehouse connection (same or different warehouse)
5. Add project dependencies:
   - Navigate to: Project Settings → Dependencies
   - Click "Add Dependency"
   - Select "Core Data" project
   - Save configuration
```

**Step 3: Configure Environment-Level Dependencies**

```markdown
Environment Configuration (Finance Analytics → Production):
1. Navigate to: Environments → Production → Settings
2. Cross-Project Dependencies:
   ✓ Core Data Project
     Environment: Production  # Production references Production
     Enabled: Yes
3. Save changes

Environment Configuration (Finance Analytics → Development):
1. Navigate to: Environments → Development → Settings
2. Cross-Project Dependencies:
   ✓ Core Data Project
     Environment: Development  # Dev references Dev
     Enabled: Yes
3. Save changes
```

### Example 4: Versioned Public Models

**In core-data project:**

```sql
-- models/marts/dim_customers_v1.sql
-- Version 1: Original schema
{{ config(
  materialized='table',
  access='public',
  version=1
) }}

select
  customer_id,
  customer_name,
  email
from {{ source('raw', 'customers') }}
```

```sql
-- models/marts/dim_customers_v2.sql
-- Version 2: Added fields
{{ config(
  materialized='table',
  access='public',
  version=2
) }}

select
  customer_id,
  customer_name,
  email,
  phone_number,     -- New field
  customer_segment  -- New field
from {{ source('raw', 'customers') }}
```

```yaml
# models/marts/dim_customers.yml
version: 2

models:
  - name: dim_customers
    latest_version: 2
    description: "Core customer dimension"
    access: public
    
    versions:
      - v: 1
        deprecation_date: 2024-06-30
        description: "Original version - deprecated"
        
      - v: 2
        description: "Enhanced with phone and segment"
        columns:
          - name: customer_id
          - name: customer_name
          - name: email
          - name: phone_number
          - name: customer_segment
```

**In downstream project (gradual migration):**

```sql
-- Legacy model still using v1
select * from {{ ref('core_data', 'dim_customers', v=1) }}

-- New model using v2
select * from {{ ref('core_data', 'dim_customers', v=2) }}

-- Default to latest version (v2)
select * from {{ ref('core_data', 'dim_customers') }}
```

### Example 5: Model Access Controls

```yaml
# models/marts/restricted/schema.yml
version: 2

models:
  - name: sensitive_customer_data
    access: protected  # Only this project can reference
    config:
      contract:
        enforced: true
    columns:
      - name: customer_id
      - name: ssn
      - name: credit_score

  - name: internal_metrics
    access: private  # Not referenceable outside this project
    columns:
      - name: metric_date
      - name: internal_kpi
```

Access levels:

- **private**: Not referenceable outside the model's package
- **protected**: Referenceable within the project only
- **public**: Referenceable by other projects

## Best Practices

### 1. Project Organization Strategy

```markdown
Recommended Project Structure:

Tier 1: Foundation Projects
  - Purpose: Ingest and clean raw data
  - Examples: source-systems, raw-data-ingestion
  - Public models: Clean, typed source data
  - Ownership: Data platform team

Tier 2: Core Domain Projects
  - Purpose: Create canonical business entities
  - Examples: core-data, customer-domain, product-domain
  - Public models: Dimensions, facts, core metrics
  - Ownership: Data engineering team

Tier 3: Analytics Projects
  - Purpose: Team-specific analytics and reporting
  - Examples: finance-analytics, marketing-analytics
  - Public models: Team-specific aggregations
  - Ownership: Domain teams (finance, marketing, etc.)

Tier 4: Presentation Projects
  - Purpose: Final reporting layers, BI tool interfaces
  - Examples: tableau-datasets, powerbi-models
  - Public models: Report-ready datasets
  - Ownership: Analytics/BI teams
```

### 2. Public Model Design

```markdown
Public Model Checklist:

✓ Contract Definition
  - Define explicit schema with data types
  - Enable contract enforcement
  - Document all columns

✓ Stability Guarantees
  - Avoid breaking changes to existing columns
  - Add new columns, don't modify existing
  - Use versioning for major changes

✓ Documentation
  - Clear model description
  - Business definitions for all fields
  - Usage examples
  - SLA expectations

✓ Testing
  - Unique and not_null tests on primary keys
  - Referential integrity tests
  - Data quality thresholds
  - Freshness checks

✓ Performance
  - Appropriate materialization strategy
  - Indexed on common join keys
  - Partitioned if large

Example:
```sql
-- ✓ Good public model
{{ config(
  materialized='incremental',
  unique_key='order_id',
  access='public',
  contract={'enforced': true},
  on_schema_change='fail'  -- Prevents accidental breaks
) }}

select
  order_id,
  customer_id,
  order_date,
  order_amount,
  order_status
from {{ source('raw', 'orders') }}
```

```markdown

### 3. Dependency Management

```markdown
Minimize Cross-Project Dependencies:

❌ Bad: Many fine-grained dependencies
finance-analytics depends on:
  - core-data.dim_customers
  - core-data.dim_products
  - core-data.fct_orders
  - core-data.dim_dates
  - core-data.dim_stores
  - customer-domain.customer_lifetime_value
  - product-domain.product_categories
  (7 dependencies across 3 projects)

✓ Good: Consolidated dependencies
finance-analytics depends on:
  - core-data (single project)
    └─ Use: dim_customers, fct_orders, dim_products
    
Benefits:
  - Simpler dependency graph
  - Easier to reason about
  - Fewer breaking change risks
```

### 4. Environment Isolation

```markdown
Environment Configuration Strategy:

Development Environment:
  Purpose: Individual developer work
  Cross-project refs: Point to upstream DEV environments
  Isolation: Each developer has own schemas
  Data: Sample or recent production subset
  
Staging Environment:
  Purpose: Pre-production validation
  Cross-project refs: Point to upstream STAGING
  Isolation: Shared staging schemas
  Data: Recent production copy
  
Production Environment:
  Purpose: Live data products
  Cross-project refs: Point to upstream PRODUCTION
  Isolation: Dedicated production schemas
  Data: Full production data
  
Key Rule: Environment types always reference matching upstream types
  ✓ Production → Production
  ✓ Staging → Staging
  ✓ Development → Development
  
  ❌ Production → Development (inconsistent data)
  ❌ Development → Production (security risk)
```

### 5. Change Management

```markdown
Managing Breaking Changes:

Step 1: Introduce new version alongside old
  - Create dim_customers_v2 with new schema
  - Keep dim_customers_v1 operational
  - Mark v1 as deprecated with date

Step 2: Communicate to downstream teams
  - Document changes in CHANGELOG
  - Notify via Slack/email
  - Provide migration guide
  - Set deprecation timeline (e.g., 90 days)

Step 3: Monitor adoption
  - Track which projects use which versions
  - Use dbt Cloud metadata API
  - Send reminders as deprecation approaches

Step 4: Remove old version
  - After deprecation date + grace period
  - Verify no downstream dependencies
  - Remove old model code
  - Update documentation

Example migration communication:
```

# BREAKING CHANGE: dim_customers v2

## What's changing?

`dim_customers` is being updated with additional fields and improved
data quality. Version 1 will be deprecated on June 30, 2024.

## New fields in v2

- phone_number
- customer_segment
- lifetime_value

## Migration guide

```sql
-- Old (v1) - works until June 30
select * from {{ ref('core_data', 'dim_customers', v=1) }}

-- New (v2) - recommended now
select * from {{ ref('core_data', 'dim_customers', v=2) }}
-- or
select * from {{ ref('core_data', 'dim_customers') }}  -- defaults to v2
```

## Action required

Update your models to use v2 by June 30, 2024.
Contact #data-platform with questions.

```
```markdown

### 6. Governance and Discovery

```markdown
Model Governance Framework:

1. Naming Conventions
   - Public models: Descriptive, stable names
   - Format: <entity_type>_<entity_name>
   - Examples: dim_customers, fct_orders, agg_revenue_daily

2. Documentation Standards
   - Every public model must have:
     ✓ Description (what, why, who owns)
     ✓ Column descriptions
     ✓ Business logic explanation
     ✓ Example queries
     ✓ Known limitations
     ✓ Update frequency

3. Access Policies
   - Default: protected (project-only)
   - Public: Requires approval
   - Process: Submit PR to governance team
   - Review criteria: Stability, quality, documentation

4. Discoverability
   - Use dbt Catalog (Explorer) to find models
   - Tag models by domain, team, use case
   - Maintain central documentation wiki
   - Regular "show and tell" sessions

Example tags:
```yaml
models:
  - name: dim_customers
    access: public
    tags:
      - domain:customer
      - tier:core
      - team:data-platform
      - sla:daily
      - pii:yes
```

```markdown

## Anti-Patterns (What NOT to Do)

### ❌ DON'T Create Circular Dependencies

```markdown
# BAD: Projects reference each other
Project A depends on Project B
Project B depends on Project A

Result: Impossible to determine build order

Example:
  core-data.dim_customers uses marketing.customer_segments
  marketing.customer_analysis uses core-data.dim_customers
  ❌ Circular dependency!

# GOOD: Unidirectional dependencies
Tier 1 (Foundation) → Tier 2 (Core) → Tier 3 (Analytics)

All dependencies flow in one direction.
```

### ❌ DON'T Make All Models Public

```markdown
# BAD: Everything is public
models:
  staging:
    +access: public     # ❌ Staging models shouldn't be public
  intermediate:
    +access: public     # ❌ Intermediate models are internal
  marts:
    +access: public     # ✓ Marts can be public (if stable)

# GOOD: Intentional public surface
models:
  staging:
    +access: private    # Internal to this project
  intermediate:
    +access: private    # Internal to this project
  marts:
    +access: public     # Stable, contracted interfaces
    +contract:
      enforced: true
```

### ❌ DON'T Skip Model Contracts

```markdown
# BAD: Public model without contract
{{ config(
  access='public'
  # No contract definition
) }}

select * from source('raw', 'customers')

Risk: Upstream changes break downstream without warning

# GOOD: Public model with enforced contract
{{ config(
  access='public',
  contract={'enforced': true}
) }}

-- Explicit schema definition
-- Changes that break contract will fail at build time
```

### ❌ DON'T Mix Environment Types in References

```markdown
# BAD: Production job references development data
Production Job (finance-analytics):
  Cross-project ref: core-data DEVELOPMENT environment
  
Result: 
  - Production reports show inconsistent data
  - Development changes appear in production
  - Data quality issues

# GOOD: Matching environment types
Production Job (finance-analytics):
  Cross-project ref: core-data PRODUCTION environment
  
Development Environment (finance-analytics):
  Cross-project ref: core-data DEVELOPMENT environment
```

### ❌ DON'T Create Deep Dependency Chains

```markdown
# BAD: Long chain of project dependencies
raw-ingestion → clean-data → core-data → customer-domain 
  → customer-analytics → customer-reporting → tableau-exports

Problems:
  - Long build times (sequential)
  - Fragile (one failure blocks all downstream)
  - Hard to debug
  - Difficult to understand lineage

# GOOD: Flatter structure
Foundation tier (parallel):
  - raw-ingestion
  - clean-data
  
Core tier (parallel):
  - core-data
  - customer-domain
  
Analytics tier (parallel):
  - customer-analytics
  - customer-reporting
  - tableau-exports

Each tier builds in parallel, clearer dependencies.
```

## Real-World Scenarios and Solutions

### Scenario 1: Migrating Monolithic Project to Mesh

**Challenge:** Large single dbt project with 500+ models becoming hard to manage.

**Solution - Phased Migration:**

```markdown
Phase 1: Assess and Plan (Week 1-2)
  Tasks:
    - Analyze model dependencies (dbt docs, lineage)
    - Identify logical domains (customer, product, finance, marketing)
    - Map team ownership
    - Define project boundaries
  
  Output: Migration plan document

Phase 2: Create Foundation Projects (Week 3-4)
  Tasks:
    - Extract staging/source models to "sources" project
    - Keep original project running
    - Set up new project in dbt Cloud
    - Configure cross-project references
  
  Testing:
    - Run both projects in parallel
    - Compare outputs for consistency

Phase 3: Extract Domain Projects (Week 5-8)
  Tasks:
    - Create customer-domain project
    - Move customer-related models
    - Update dependencies in main project
    - Repeat for product-domain, finance-domain
  
  Testing:
    - Validate each extraction independently
    - Ensure no broken references

Phase 4: Migrate Consuming Projects (Week 9-12)
  Tasks:
    - Create team-specific projects (marketing-analytics, etc.)
    - Move team models
    - Configure cross-project references to domains
  
  Testing:
    - Validate end-to-end pipelines
    - Performance testing

Phase 5: Decommission Monolith (Week 13-14)
  Tasks:
    - Verify all models migrated
    - Run parallel for 1 week
    - Switch production jobs to new projects
    - Archive old project

Example extraction script:
```python
# Script to extract models and update references
import os
import re

def extract_models(source_dir, target_project, model_list):
    """Move models to new project and update refs"""
    for model in model_list:
        source_path = f"{source_dir}/models/{model}.sql"
        target_path = f"{target_project}/models/{model}.sql"
        
        # Copy model file
        shutil.copy(source_path, target_path)
        
        # Update references in remaining models
        update_references(source_dir, model, target_project)

def update_references(project_dir, model_name, new_project):
    """Update ref() calls to use cross-project syntax"""
    for sql_file in glob.glob(f"{project_dir}/models/**/*.sql"):
        content = open(sql_file).read()
        
        # Replace: ref('model_name')
        # With: ref('new_project', 'model_name')
        pattern = f"ref\\(['\"]}{model_name}['\"]\\)"
        replacement = f"ref('{new_project}', '{model_name}')"
        
        updated = re.sub(pattern, replacement, content)
        
        if updated != content:
            with open(sql_file, 'w') as f:
                f.write(updated)
            print(f"Updated references in {sql_file}")

# Usage
extract_models(
    source_dir="/projects/monolith",
    target_project="/projects/customer-domain",
    model_list=["dim_customers", "fct_customer_events"]
)
```

```markdown

### Scenario 2: Implementing Model Versioning Strategy

**Challenge:** Need to add columns to widely-used public model without breaking downstream consumers.

**Solution:**

```sql
-- Step 1: Create new version with additional fields
-- models/marts/dim_customers_v2.sql
{{
  config(
    materialized='table',
    access='public',
    version=2,
    contract={'enforced': true}
  )
}}

select
  customer_id,
  customer_name,
  email,
  created_at,
  -- New fields in v2
  phone_number,
  customer_segment,
  lifetime_value,
  last_purchase_date
from {{ source('raw', 'customers') }}
left join {{ ref('customer_metrics') }} using (customer_id)
```

```yaml
# models/marts/dim_customers.yml
version: 2

models:
  - name: dim_customers
    latest_version: 2
    access: public
    description: |
      Core customer dimension table.
      
      **Version History:**
      - v2 (current): Added phone, segment, LTV, last purchase date
      - v1 (deprecated 2024-06-30): Original schema
    
    versions:
      - v: 1
        deprecation_date: 2024-06-30
        description: "Original version - basic customer info only"
        columns:
          - name: customer_id
          - name: customer_name
          - name: email
          - name: created_at
        
      - v: 2
        description: "Enhanced with engagement metrics"
        columns:
          - name: customer_id
            description: "Unique customer identifier"
            tests:
              - unique
              - not_null
          - name: customer_name
          - name: email
          - name: created_at
          - name: phone_number
            description: "Customer phone (added in v2)"
          - name: customer_segment
            description: "Segmentation: high_value, medium_value, low_value"
          - name: lifetime_value
            description: "Total customer LTV in USD"
          - name: last_purchase_date
            description: "Date of most recent purchase"
```

```markdown
Communication Plan:
1. Announce v2 in team channels
2. Update documentation
3. Provide migration examples
4. Set 90-day migration period
5. Send weekly reminders
6. Remove v1 after grace period
```

### Scenario 3: Cross-Team Collaboration Workflow

**Challenge:** Marketing team needs customer segmentation from data platform team's model.

**Solution - Collaboration Process:**

```markdown
Step 1: Marketing Team Request
  - Opens issue in core-data repository
  - Title: "Request: Add customer_segment to dim_customers"
  - Justification: Need for campaign targeting
  - Expected timeline: 2 weeks

Step 2: Data Platform Review
  - Evaluate feasibility
  - Check if fits in existing model or needs new model
  - Estimate effort
  - Approve or suggest alternatives

Step 3: Implementation (Data Platform)
  - Create feature branch
  - Add customer_segment logic
  - Update model contract
  - Add tests
  - Document new field
  
Step 4: Review and Merge
  - Marketing reviews changes in dev environment
  - Data platform merges to main
  - Deploy to staging
  - Marketing validates in staging
  
Step 5: Production Rollout
  - Deploy to production
  - Marketing updates downstream models
  - Monitor for issues
```

**Implementation:**

```sql
-- core-data: models/marts/dim_customers.sql
-- Adding requested customer_segment field

with customer_metrics as (
  select
    customer_id,
    sum(order_amount) as total_spend,
    count(order_id) as order_count,
    max(order_date) as last_order_date
  from {{ ref('fct_orders') }}
  group by 1
),

customer_segments as (
  select
    customer_id,
    case
      when total_spend >= 10000 then 'high_value'
      when total_spend >= 1000 then 'medium_value'
      when total_spend > 0 then 'low_value'
      else 'no_purchase'
    end as customer_segment
  from customer_metrics
)

select
  c.customer_id,
  c.customer_name,
  c.email,
  c.created_at,
  cs.customer_segment  -- New field for marketing
from {{ source('raw', 'customers') }} c
left join customer_segments cs on c.customer_id = cs.customer_id
```

```sql
-- marketing-analytics: models/campaign_targeting.sql
-- Consuming new customer_segment field

with customers as (
  select * from {{ ref('core_data', 'dim_customers') }}
),

high_value_campaign as (
  select
    customer_id,
    customer_name,
    email,
    'premium_offer' as campaign_type
  from customers
  where customer_segment = 'high_value'
)

select * from high_value_campaign
```

## Sample Exam Questions

### Question 1: Cross-Project References

**Question:** You're creating a model in the `marketing-analytics` project that needs to reference `dim_customers` from the `core-data` project. What is the correct syntax?

A) `select * from ref('dim_customers')`  
B) `select * from {{ ref('core-data.dim_customers') }}`  
C) `select * from {{ ref('core_data', 'dim_customers') }}`  
D) `select * from {{ source('core_data', 'dim_customers') }}`

**Answer:** C) `select * from {{ ref('core_data', 'dim_customers') }}`

**Explanation:**

- Cross-project ref uses two-argument syntax: `ref('project_name', 'model_name')`
- Project name uses underscore (not hyphen) matching dbt_project.yml name
- `source()` is for raw data sources, not dbt models
- Single-argument `ref()` only works within the same project

### Question 2: Model Access Levels

**Question:** You want a model to be usable by other models in the same project, but NOT by other projects. Which access level should you use?

A) private  
B) protected  
C) public  
D) internal

**Answer:** B) protected

**Explanation:**

- `private`: Not referenceable outside the model's package
- `protected`: Referenceable within the project only (correct answer)
- `public`: Referenceable by other projects
- `internal`: Not a valid access level in dbt

### Question 3: Environment Types

**Question:** Your production job in `finance-analytics` project should reference models from which environment in the upstream `core-data` project?

A) Development  
B) Staging  
C) Production  
D) Latest

**Answer:** C) Production

**Explanation:**

- Environment types should always match across projects
- Production jobs should reference production data for consistency
- Using development or staging in production would cause data inconsistencies
- "Latest" is not a valid environment type

### Question 4: Model Contracts

**Question:** What is the PRIMARY purpose of enforcing model contracts on public models?

A) Improve query performance  
B) Prevent breaking changes to downstream consumers  
C) Reduce storage costs  
D) Enable incremental materializations

**Answer:** B) Prevent breaking changes to downstream consumers

**Explanation:**

- Contracts define explicit schemas that must be maintained
- Changes that break the contract fail at build time
- Protects downstream projects from unexpected schema changes
- Performance, cost, and materializations are separate concerns

### Question 5: Project Organization

**Question:** Which project structure BEST supports scalable dbt Mesh architecture?

A) Single project with all models  
B) One project per team with no dependencies  
C) Tiered projects (foundation → core → analytics) with unidirectional dependencies  
D) Each model in its own project

**Answer:** C) Tiered projects (foundation → core → analytics) with unidirectional dependencies

**Explanation:**

- Tiered structure enables clear ownership and dependencies
- Unidirectional flow prevents circular dependencies
- Single project doesn't scale well
- One project per team with no dependencies doesn't enable reuse
- One project per model is too granular and unmaintainable

### Question 6: Versioning

**Question:** You need to add new columns to a public model that's widely used. What is the BEST approach?

A) Add columns directly to the existing model  
B) Create a new versioned model (v2), deprecate v1 after migration period  
C) Create a completely new model with a different name  
D) Add columns but mark them as optional in documentation

**Answer:** B) Create a new versioned model (v2), deprecate v1 after migration period

**Explanation:**

- Versioning allows graceful migration without breaking existing consumers
- Downstream teams can migrate on their own schedule
- Direct changes (A) could break downstream models
- New model with different name (C) loses continuity
- Documentation (D) doesn't prevent breaking changes

## Summary

dbt Mesh enables scalable, modular data platforms through project independence and cross-project collaboration. Key takeaways:

1. **Multi-project architecture**: Separate codebases with clear ownership

2. **Cross-project references**: Use `ref('project_name', 'model_name')` syntax

3. **Model access levels**: private, protected, public

4. **Environment matching**: Prod references prod, dev references dev

5. **Model contracts**: Enforce schemas for public models

6. **Versioning strategy**: Manage breaking changes gracefully

7. **Governance**: Clear policies for public model creation and maintenance

8. **Unidirectional dependencies**: Avoid circular references

## Next Steps

Continue to [Topic 8: Configuring and Using dbt Catalog](./08-dbt-catalog.md) to learn about lineage visualization and model discovery.
