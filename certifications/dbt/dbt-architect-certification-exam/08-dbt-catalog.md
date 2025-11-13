# Topic 8: Configuring and Using dbt Catalog

## Overview

dbt Catalog (formerly dbt Explorer) is a comprehensive metadata interface for understanding, monitoring, and optimizing dbt projects. It provides lineage visualization, performance insights, model discovery, and cross-project navigation capabilities essential for managing production dbt environments.

## Key Concepts and Definitions

### What is dbt Catalog?

A metadata-driven interface that provides:

- **Visual lineage**: DAG visualization of model dependencies
- **Model discovery**: Search and browse all models across projects
- **Performance insights**: Execution times, resource usage, optimization opportunities
- **Documentation hub**: Centralized access to model descriptions and schemas
- **Cross-project navigation**: Explore dependencies across dbt Mesh architectures

### Core Features

#### 1. Lineage Visualization

Interactive directed acyclic graph (DAG) showing:

- Model dependencies (upstream and downstream)
- Source-to-exposure data flow
- Cross-project references
- Test coverage
- Column-level lineage (premium feature)

#### 2. Model Explorer

Browse and search capabilities:

- Filter by project, tags, materialization, owner
- Search by model name or description
- View model details (schema, tests, runs)
- Access model documentation

#### 3. Performance Analysis

Insights into execution and resource usage:

- Model execution times
- Warehouse resource consumption
- Query performance trends
- Optimization recommendations
- Cost attribution

#### 4. Public Model Registry

Discover consumable models across projects:

- Find public models for cross-project references
- View model contracts and schemas
- Understand usage and dependencies
- Identify model owners

## Practical Examples

### Example 1: Navigating Lineage to Troubleshoot Issues

**Scenario:** The `revenue_dashboard` report shows incorrect totals. Trace the issue upstream.

**Steps in dbt Catalog:**

```markdown
1. Open dbt Catalog
   Navigate to: https://cloud.getdbt.com/accounts/{account_id}/catalog

2. Search for affected model
   Search bar: "revenue_dashboard"
   Select: models/marts/revenue_dashboard

3. View lineage
   Click: "Lineage" tab
   Display: Full DAG with upstream dependencies
   
   Visual shows:
   raw.orders → stg_orders → int_order_metrics → fct_orders → revenue_dashboard
                                                              ↗
   raw.returns → stg_returns → int_returns ────────────────┘

4. Identify recent changes
   Check each upstream model for:
   - Recent run times
   - Code changes (git commits)
   - Test failures
   - Row count changes
   
   Notice: int_returns failed data quality test (row count dropped 50%)

5. Investigate int_returns
   Click: int_returns model
   View: 
   - Last run: Failed
   - Test: assert_row_count_range failed
   - Error: Expected 1000-2000 rows, got 500 rows
   
6. Root cause identified
   Check source data:
   raw.returns has missing data for recent dates
   
7. Resolution
   Fix upstream data ingestion issue
   Re-run affected models
   Verify revenue_dashboard corrected
```

### Example 2: Using Catalog for Performance Optimization

**Scenario:** Identify slow-running models and optimize them.

**Steps:**

```markdown
1. Navigate to Performance View
   dbt Catalog → "Performance" tab

2. Sort by execution time
   Columns visible:
   - Model name
   - Avg execution time (30 days)
   - Warehouse compute (credits)
   - Last run duration
   - Trend (↑ slower, ↓ faster, → stable)

3. Identify outliers
   Top slow models:
   - customer_lifetime_value: 45 min (avg)
   - daily_aggregate_metrics: 38 min
   - product_affinity_matrix: 32 min

4. Analyze customer_lifetime_value
   Click model → Performance tab
   
   Insights shown:
   - Execution time trend: Increasing over 30 days
   - Row count: 50M rows
   - Materialization: Table (full refresh nightly)
   - Warehouse: Large (32 credits/hour)
   
   Recommendations:
   ⚠️ Consider incremental materialization
   ⚠️ Model selects entire history daily
   ⚠️ No filters applied to date range

5. Review query profile
   Click: "View Query Profile" (links to warehouse)
   
   Snowflake Query Profile shows:
   - 80% time in JOIN operations
   - Cartesian product detected
   - Missing WHERE clause on date filter

6. Optimization actions
   a) Convert to incremental:
```sql
   -- Before: Full table refresh
   {{ config(materialized='table') }}
   
   -- After: Incremental with date filter
   {{
     config(
       materialized='incremental',
       unique_key='customer_id',
       on_schema_change='append_new_columns'
     )
   }}
   
   select
     customer_id,
     sum(order_amount) as lifetime_value,
     max(order_date) as last_order_date
   from {{ ref('fct_orders') }}
   {% if is_incremental() %}
     where order_date >= (select max(last_order_date) from {{ this }})
   {% endif %}
   group by 1
```

```markdown

   b) Add date filter to reduce data scanned

   c) Optimize JOIN order based on table sizes

7. Measure improvement
   After changes:
   - Execution time: 45 min → 8 min (82% improvement)
   - Credits used: 24 → 4 per run
   - ROI: $200/month savings
```

### Example 3: Discovering Public Models for Cross-Project Use

**Scenario:** Marketing team needs customer segmentation data. Find available public models.

**Steps:**

```markdown
1. Navigate to Public Models view
   dbt Catalog → "Public Models" filter
   
2. Search by domain
   Tags filter: "domain:customer"
   
   Results show:
   - core_data.dim_customers
   - core_data.fct_customer_events
   - customer_domain.customer_segments
   - customer_domain.customer_lifetime_value

3. Review customer_segments model
   Click: customer_domain.customer_segments
   
   Details shown:
   - Description: "Customer segmentation based on RFM analysis"
   - Owner: Data Platform Team
   - Project: customer_domain
   - Access: public
   - Contract: enforced
   - Update frequency: Daily at 6 AM UTC
   - SLA: Available by 8 AM UTC

4. View schema
   Columns tab:
   - customer_id (integer, PK)
   - segment_name (varchar): high_value, medium_value, low_value
   - recency_score (integer): 1-5
   - frequency_score (integer): 1-5
   - monetary_score (integer): 1-5
   - last_updated (timestamp)

5. Check usage examples
   Documentation tab shows:
```sql
   -- Example: Target high-value customers for campaign
   with customer_segments as (
     select * from {{ ref('customer_domain', 'customer_segments') }}
   ),
   
   high_value_customers as (
     select
       customer_id,
       segment_name,
       monetary_score
     from customer_segments
     where segment_name = 'high_value'
   )
   
   select * from high_value_customers
```

```markdown

6. View downstream usage
   Lineage tab → "Downstream" view
   
   Currently used by:
   - finance_analytics.revenue_forecasting
   - marketing_analytics.campaign_targeting
   - ops_analytics.churn_prediction

7. Copy reference syntax
   Click: "Copy ref()" button
   
   Copies to clipboard:
   {{ ref('customer_domain', 'customer_segments') }}

8. Implement in marketing project
   Create new model using discovered public model
```

### Example 4: Column-Level Lineage (Premium Feature)

**Scenario:** Understand which source columns flow into a specific report field.

```markdown
Column-Level Lineage View:

Report Field: revenue_dashboard.net_revenue

Trace upstream:
  revenue_dashboard.net_revenue
    ← fct_orders.order_amount
      ← int_order_totals.gross_amount
        ← stg_orders.amount
          ← raw.orders.order_total

Transformations applied at each step:
1. raw.orders.order_total → stg_orders.amount
   - Cast to decimal(18,2)
   - Converted from cents to dollars (/ 100)

2. stg_orders.amount → int_order_totals.gross_amount
   - Filtered: where order_status != 'cancelled'
   - Aggregation: sum(amount)

3. int_order_totals.gross_amount → fct_orders.order_amount
   - Joined with discounts table
   - Calculated: gross_amount - discount_amount

4. fct_orders.order_amount → revenue_dashboard.net_revenue
   - Aggregation: sum(order_amount)
   - Filtered: where order_date >= '2024-01-01'

This traces exactly how raw data transforms into final report field.
```

### Example 5: Using Catalog API for Automation

**Scenario:** Automatically generate data dictionary from Catalog metadata.

```python
# Script to extract model metadata and generate documentation
import requests
import pandas as pd
from datetime import datetime

# dbt Cloud API configuration
API_TOKEN = "your_api_token"
ACCOUNT_ID = "12345"
PROJECT_ID = "67890"

BASE_URL = f"https://cloud.getdbt.com/api/v2/accounts/{ACCOUNT_ID}"
headers = {"Authorization": f"Token {API_TOKEN}"}

def get_catalog_metadata():
    """Fetch all models from dbt Catalog"""
    url = f"{BASE_URL}/projects/{PROJECT_ID}/models/"
    response = requests.get(url, headers=headers)
    
    if response.status_code == 200:
        return response.json()['data']
    else:
        raise Exception(f"API error: {response.status_code}")

def get_model_columns(model_id):
    """Get column details for a specific model"""
    url = f"{BASE_URL}/models/{model_id}/columns/"
    response = requests.get(url, headers=headers)
    
    if response.status_code == 200:
        return response.json()['data']
    return []

def generate_data_dictionary():
    """Generate comprehensive data dictionary"""
    models = get_catalog_metadata()
    
    dictionary = []
    
    for model in models:
        model_name = model['name']
        model_desc = model.get('description', 'No description')
        model_owner = model.get('tags', {}).get('owner', 'Unknown')
        model_access = model.get('access', 'private')
        
        # Get columns
        columns = get_model_columns(model['id'])
        
        for col in columns:
            dictionary.append({
                'Model': model_name,
                'Column': col['name'],
                'Data Type': col['data_type'],
                'Description': col.get('description', ''),
                'Owner': model_owner,
                'Access': model_access,
                'PII': 'pii' in col.get('tags', [])
            })
    
    # Convert to DataFrame
    df = pd.DataFrame(dictionary)
    
    # Export to multiple formats
    df.to_csv('data_dictionary.csv', index=False)
    df.to_excel('data_dictionary.xlsx', index=False)
    
    # Generate HTML version
    html = df.to_html(index=False, classes='table table-striped')
    with open('data_dictionary.html', 'w') as f:
        f.write(f"""
        <html>
        <head>
            <title>Data Dictionary - Generated {datetime.now()}</title>
            <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5/dist/css/bootstrap.min.css">
        </head>
        <body>
            <div class="container mt-5">
                <h1>Data Dictionary</h1>
                <p>Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}</p>
                {html}
            </div>
        </body>
        </html>
        """)
    
    print(f"Data dictionary generated with {len(df)} columns across {df['Model'].nunique()} models")
    return df

def find_models_using_column(column_name):
    """Find all models that use a specific column"""
    models = get_catalog_metadata()
    results = []
    
    for model in models:
        columns = get_model_columns(model['id'])
        if any(col['name'] == column_name for col in columns):
            results.append({
                'model': model['name'],
                'project': model['project_name'],
                'access': model.get('access', 'private')
            })
    
    return results

def get_model_performance_metrics(days=30):
    """Get performance metrics for all models"""
    url = f"{BASE_URL}/projects/{PROJECT_ID}/runs/"
    params = {"limit": 100}
    
    response = requests.get(url, headers=headers, params=params)
    runs = response.json()['data']
    
    # Aggregate by model
    model_metrics = {}
    
    for run in runs:
        for model_result in run.get('run_results', []):
            model_name = model_result['unique_id']
            execution_time = model_result['execution_time']
            
            if model_name not in model_metrics:
                model_metrics[model_name] = {
                    'total_runs': 0,
                    'total_time': 0,
                    'failures': 0
                }
            
            model_metrics[model_name]['total_runs'] += 1
            model_metrics[model_name]['total_time'] += execution_time
            
            if model_result['status'] != 'success':
                model_metrics[model_name]['failures'] += 1
    
    # Calculate averages
    results = []
    for model, metrics in model_metrics.items():
        results.append({
            'model': model,
            'avg_execution_time': metrics['total_time'] / metrics['total_runs'],
            'total_runs': metrics['total_runs'],
            'failure_rate': metrics['failures'] / metrics['total_runs'] * 100
        })
    
    return pd.DataFrame(results).sort_values('avg_execution_time', ascending=False)

# Usage examples
if __name__ == '__main__':
    # Generate data dictionary
    dictionary = generate_data_dictionary()
    
    # Find models using 'customer_id'
    models_with_customer_id = find_models_using_column('customer_id')
    print(f"\nModels using 'customer_id': {len(models_with_customer_id)}")
    
    # Get performance insights
    performance = get_model_performance_metrics()
    print("\nTop 10 slowest models:")
    print(performance.head(10))
```

## Best Practices

### 1. Regular Lineage Reviews

```markdown
Establish Lineage Review Cadence:

Weekly (Team Level):
  - Review models your team owns
  - Check for new downstream dependencies
  - Verify test coverage on critical paths
  - Identify optimization opportunities

Monthly (Project Level):
  - Full lineage audit
  - Document complex dependency chains
  - Identify circular or overly complex dependencies
  - Review cross-project boundaries

Quarterly (Organization Level):
  - Audit entire mesh architecture
  - Identify deprecated models still in use
  - Review public model catalog
  - Assess overall complexity metrics

Use Catalog to:
  ✓ Visualize full data flow
  ✓ Identify orphaned models (no downstream usage)
  ✓ Find models without tests
  ✓ Discover optimization opportunities
```

### 2. Performance Monitoring Strategy

```markdown
Performance Monitoring Approach:

Daily:
  - Check execution time trends
  - Identify models with >20% increase
  - Monitor resource consumption

Weekly:
  - Review top 10 slowest models
  - Track optimization progress
  - Calculate cost per model

Monthly:
  - Full performance audit
  - Identify systemic issues
  - Update optimization backlog

Key Metrics to Track:
  ✓ Avg execution time by model
  ✓ Total warehouse compute cost
  ✓ Cost per model
  ✓ Execution time trends (increasing/decreasing)
  ✓ Models exceeding SLA thresholds
  ✓ Resource utilization patterns

Optimization Triggers:
  ⚠️ Model takes >15 min consistently
  ⚠️ Execution time increased >50% in 30 days
  ⚠️ Model costs >$10/run
  ⚠️ Model frequently causes downstream delays
```

### 3. Model Discovery and Documentation

```markdown
Make Models Discoverable:

1. Comprehensive Descriptions
   - What: Clear model purpose
   - Why: Business justification
   - Who: Owner and team
   - When: Update frequency and SLA
   - How: Key transformations

2. Consistent Tagging
   Tags to include:
   - domain: (customer, product, finance, etc.)
   - tier: (raw, staging, intermediate, marts)
   - team: (data-platform, analytics, etc.)
   - pii: (yes/no)
   - sla: (hourly, daily, weekly)

3. Usage Examples
   Include in documentation:
```sql
   -- Example: Common use case
   with base as (
     select * from {{ ref('your_model') }}
   )
   select * from base where condition
```

```markdown

4. Column Documentation
   Every column should have:
   - Description
   - Data type
   - Possible values (if enumerated)
   - Business logic
   - Related columns

5. Maintain Catalog Freshness
   - Regenerate docs after every deploy
   - Keep descriptions up to date
   - Remove deprecated models
   - Update examples when logic changes
```

### 4. Cross-Project Navigation

```markdown
Navigating dbt Mesh in Catalog:

1. Use Project Filters
   Filter by: Project name
   View: Only models from specific project
   Use case: Focus on your team's domain

2. Identify Cross-Project Dependencies
   Lineage view shows:
   - Models from other projects (different color)
   - Cross-project reference points
   - Dependency direction

3. Discover Public Models
   Filter by: access = public
   View: All consumable models across organization
   Use case: Find reusable data products

4. Understand Impact Radius
   Click model → View downstream
   See: All projects and models affected by changes
   Use case: Plan breaking changes

5. Contact Model Owners
   Model details show:
   - Owner tag
   - Team information
   - Documentation links
   Use case: Coordinate changes or ask questions
```

### 5. Cost Attribution and Optimization

```markdown
Using Catalog for Cost Management:

1. Calculate Model-Level Costs
   Formula:
   Cost = (Execution Time) × (Warehouse Rate) × (Run Frequency)
   
   Example:
   Model: customer_lifetime_value
   - Execution: 30 min = 0.5 hours
   - Warehouse: Large = $4/hour
   - Frequency: Daily = 30 runs/month
   Cost = 0.5 × $4 × 30 = $60/month

2. Identify High-Cost Models
   Sort by total monthly cost
   Focus optimization on top 20% (Pareto principle)

3. Evaluate ROI of Optimization
   Before optimization:
   - Execution time: 45 min
   - Cost: $90/month
   
   After optimization:
   - Execution time: 8 min
   - Cost: $16/month
   
   Savings: $74/month = $888/year
   
   If optimization took 8 hours @ $100/hour:
   - Investment: $800
   - Payback period: 11 months
   - ROI: Positive after year 1

4. Set Cost Budgets
   By project:
   - core-data: $500/month budget
   - finance-analytics: $200/month budget
   - marketing-analytics: $150/month budget
   
   Monitor in Catalog and alert when exceeded
```

### 6. Using Catalog for Impact Analysis

```markdown
Before Making Changes:

1. Check Downstream Impact
   Question: "If I change dim_customers, what breaks?"
   
   Steps:
   a) Open dim_customers in Catalog
   b) View lineage → Downstream
   c) Count affected models: 47 models across 5 projects
   d) Identify public consumers: 
      - marketing_analytics (12 models)
      - finance_analytics (8 models)
      - customer_success (6 models)
   e) Plan:
      - Cannot make breaking change directly
      - Use versioning (v2)
      - Notify affected teams
      - Set 90-day migration period

2. Assess Risk Level
   High Risk: >20 downstream models, crosses project boundaries
   Medium Risk: 10-20 downstream models, within project
   Low Risk: <10 downstream models, all owned by same team

3. Create Migration Plan
   Based on downstream count and complexity
```

## Anti-Patterns (What NOT to Do)

### ❌ DON'T Ignore Performance Insights

```markdown
# BAD: Never check Catalog performance metrics
Behavior:
  - Deploy models without performance review
  - Ignore increasing execution times
  - No cost monitoring

Result:
  - Warehouse costs spiral out of control
  - Jobs miss SLAs
  - Cascading delays

# GOOD: Regular performance monitoring
Behavior:
  - Weekly performance review
  - Set execution time thresholds
  - Optimize proactively

Result:
  - Predictable costs
  - Reliable SLAs
  - Efficient resource usage
```

### ❌ DON'T Make Changes Without Impact Analysis

```markdown
# BAD: Change public models without checking downstream usage
Steps:
  1. Modify dim_customers schema
  2. Deploy to production
  3. 15 downstream models break
  4. Emergency rollback

# GOOD: Impact analysis before changes
Steps:
  1. Check downstream usage in Catalog
  2. Identify all affected models
  3. Use versioning for breaking changes
  4. Coordinate with affected teams
  5. Staged rollout
```

### ❌ DON'T Let Models Become Undiscoverable

```markdown
# BAD: Minimal or no documentation
models/marts/important_model.sql:
  - No description
  - No column documentation
  - No tags
  - No usage examples

Result:
  - Models get duplicated (others can't find them)
  - No one knows what model does
  - Unused models accumulate

# GOOD: Comprehensive documentation
models/marts/important_model.sql + schema.yml:
  - Clear description
  - All columns documented
  - Proper tagging
  - Usage examples
  - Owner information

Result:
  - Easy to discover
  - Reusable
  - Maintainable
```

### ❌ DON'T Ignore Lineage Complexity

```markdown
# BAD: Allow unrestricted model proliferation
Lineage shows:
  - 500+ models in single project
  - 10+ layers deep
  - Complex circular-like patterns
  - Many orphaned models

Problems:
  - Hard to understand data flow
  - Long build times
  - Difficult troubleshooting

# GOOD: Maintain clean lineage
Best practices:
  - Keep layers to 4-5 max
  - Remove unused models
  - Simplify complex chains
  - Modularize with clear boundaries

Benefits:
  - Clear data flow
  - Faster builds
  - Easy debugging
```

### ❌ DON'T Overlook Column-Level Lineage

```markdown
# BAD: Only look at model-level lineage
Problem scenario:
  - Report field shows wrong value
  - Know which model it comes from
  - Don't know which source column
  - Trace through 8 models manually

# GOOD: Use column-level lineage (if available)
Solution:
  - Trace specific column through entire pipeline
  - See all transformations applied
  - Quickly identify where issue originates
  - Fix root cause, not symptom
```

## Real-World Scenarios and Solutions

### Scenario 1: Investigating Data Quality Issue

**Challenge:** Executive dashboard shows revenue dropped 30% overnight, but sales team reports normal activity.

**Solution using Catalog:**

```markdown
Step 1: Identify affected model
  - Dashboard model: exec_dashboard.daily_revenue
  - Open in Catalog

Step 2: Check recent changes
  - View lineage
  - Click "Recent Changes" filter
  - Shows: fct_orders modified 12 hours ago

Step 3: Analyze fct_orders changes
  - View git diff (linked from Catalog)
  - Change: Added filter `where order_status = 'completed'`
  - Previously included 'pending' status

Step 4: Verify impact
  - Column-level lineage shows:
    fct_orders.order_amount flows to daily_revenue.total_revenue
  - New filter excludes pending orders
  - Pending orders typically 30% of daily volume

Step 5: Determine if intended
  - Check commit message: "Exclude pending orders per finance request"
  - Contact finance team: Change was intentional
  - Dashboard just needs updated documentation

Step 6: Resolution
  - Update dashboard description to clarify "completed orders only"
  - No code change needed
  - Communicate to executive team

Root cause: Communication gap, not data error
Catalog helped: Quickly traced change through lineage
```

### Scenario 2: Optimizing Warehouse Costs

**Challenge:** Monthly warehouse costs increased 40% without obvious cause.

**Solution using Catalog Performance:**

```markdown
Step 1: Identify cost drivers
  - Catalog → Performance view
  - Sort by: Total cost (30 days)
  - Top 5 models account for 60% of total cost:
    1. customer_affinity_matrix: $450/month (was $180)
    2. product_recommendations: $320/month (was $220)
    3. daily_aggregates: $280/month (was $150)
    4. historical_trends: $240/month (was $140)
    5. customer_segments: $200/month (was $120)

Step 2: Analyze customer_affinity_matrix
  - Execution time: 2.5 hours (was 1 hour)
  - Trend: Increasing steadily over 60 days
  - Rows: 500M (was 200M)
  - No recent code changes

Step 3: Investigate data growth
  - Check upstream sources
  - Customer base grew 25%
  - But execution time grew 150%
  - Indicates non-linear performance degradation

Step 4: Review query execution
  - View query profile (linked from Catalog)
  - Identifies: Cross join creating cartesian product
  - Missing index on join key

Step 5: Implement optimizations
  a) Add missing join condition:
```sql
  -- Before
  from customers c
  join products p  -- Cartesian product!
  
  -- After
  from customers c
  join orders o on c.customer_id = o.customer_id
  join products p on o.product_id = p.product_id
```

```markdown

  b) Add incremental processing:
```sql
  {{
    config(
      materialized='incremental',
      unique_key=['customer_id', 'product_id']
    )
  }}
  
  {% if is_incremental() %}
    -- Only process last 7 days
    where order_date >= (select max(order_date) from {{ this }}) - interval '7 days'
  {% endif %}
```

```markdown

Step 6: Measure improvement
  After optimization:
  - Execution time: 2.5 hours → 20 minutes (92% reduction)
  - Cost: $450/month → $40/month ($410/month savings)
  
  Apply similar optimizations to other top-cost models
  Total savings: $800/month = $9,600/year

Step 7: Prevent future issues
  - Set up cost alerts (Catalog + monitoring)
  - Weekly performance review
  - Budget per project
```

### Scenario 3: Planning Breaking Change Rollout

**Challenge:** Need to restructure dim_customers to split PII into separate table for compliance.

**Solution using Catalog Impact Analysis:**

```markdown
Step 1: Assess current usage
  - Open dim_customers in Catalog
  - View downstream models: 73 models depend on this
  - Cross-project usage:
    - finance-analytics: 18 models
    - marketing-analytics: 24 models
    - customer-success: 12 models
    - product-analytics: 8 models
    - ops-analytics: 11 models

Step 2: Identify columns being restructured
  - Columns to move to new table (dim_customers_pii):
    - ssn
    - date_of_birth
    - phone_number
    - email (keeping in both for now)
  
  - Use column-level lineage to see which models use these

Step 3: Plan versioning strategy
  - Create dim_customers_v2 (without PII fields)
  - Create dim_customers_pii (PII only, restricted access)
  - Keep dim_customers_v1 (current) for migration period

Step 4: Communicate to affected teams
  Impact assessment:
  - High impact: 23 models directly use PII fields
  - Medium impact: 50 models use dim_customers but not PII fields
  
  Communication:
  - Email to all team leads
  - Slack announcement
  - Office hours for questions
  - 90-day migration timeline

Step 5: Create migration guide
```sql
  -- Before (v1)
  select
    customer_id,
    customer_name,
    email,
    phone_number,  -- Moving to separate table
    ssn              -- Moving to separate table
  from {{ ref('dim_customers') }}
  
  -- After (v2) - Non-PII use case
  select
    customer_id,
    customer_name,
    email
  from {{ ref('dim_customers', v=2) }}
  
  -- After (v2) - Need PII data
  select
    c.customer_id,
    c.customer_name,
    c.email,
    pii.phone_number,
    pii.ssn
  from {{ ref('dim_customers', v=2) }} c
  left join {{ ref('dim_customers_pii') }} pii
    on c.customer_id = pii.customer_id
```

```markdown

Step 6: Monitor migration progress
  - Use Catalog to track which models still reference v1
  - Send weekly reminders
  - Offer help to teams struggling with migration

Step 7: Execute deprecation
  - After 90 days + 2 week grace period
  - Verify zero v1 usage in Catalog
  - Remove v1 code
  - Update documentation

Result:
  - Successful migration
  - Zero breaking changes
  - Improved compliance posture
  - Clear audit trail
```

## Sample Exam Questions

### Question 1: Lineage Troubleshooting

**Question:** You need to trace why a specific column in a report has incorrect values. What feature of dbt Catalog is MOST helpful?

A) Model-level lineage  
B) Column-level lineage  
C) Performance metrics  
D) Public model registry

**Answer:** B) Column-level lineage

**Explanation:**

- Column-level lineage traces a specific column through the entire pipeline
- Shows all transformations applied to that column
- Model-level lineage (A) shows model dependencies but not column-specific flow
- Performance metrics (C) and public model registry (D) don't help trace data issues

### Question 2: Performance Optimization

**Question:** In dbt Catalog, you notice a model's execution time has increased from 10 minutes to 45 minutes over the past month, with no code changes. What should you investigate FIRST?

A) Model description accuracy  
B) Upstream source data volume growth  
C) Documentation completeness  
D) Number of downstream dependencies

**Answer:** B) Upstream source data volume growth

**Explanation:**

- Increasing execution time without code changes often indicates data volume growth
- Should check if source tables have grown significantly
- Documentation (A, C) doesn't affect performance
- Downstream dependencies (D) don't affect upstream model execution time

### Question 3: Public Model Discovery

**Question:** You need to find all public models tagged with "domain:finance" that are available for cross-project references. Where in dbt Catalog should you look?

A) Performance tab with cost filters  
B) Lineage view with project filters  
C) Model explorer with access=public and tag filters  
D) Documentation tab

**Answer:** C) Model explorer with access=public and tag filters

**Explanation:**

- Model explorer allows filtering by access level and tags
- Can combine filters: access=public AND tags include "domain:finance"
- Performance tab (A) focuses on execution metrics
- Lineage view (B) shows dependencies, not discovery
- Documentation tab (D) is for individual model docs

### Question 4: Impact Analysis

**Question:** Before making a breaking change to a public model, what should you check in dbt Catalog to assess impact?

A) Model's execution time trends  
B) Model's downstream dependencies and cross-project usage  
C) Model's git commit history  
D) Model's row count statistics

**Answer:** B) Model's downstream dependencies and cross-project usage

**Explanation:**

- Downstream dependencies show which models will be affected
- Cross-project usage indicates which teams need notification
- Execution time (A) doesn't indicate breaking change impact
- Git history (C) shows past changes, not future impact
- Row counts (D) don't relate to breaking changes

### Question 5: Cost Management

**Question:** You want to identify which models are driving the highest warehouse costs to prioritize optimization efforts. Which Catalog metric should you analyze?

A) Number of downstream dependencies  
B) Model documentation completeness  
C) Total execution time multiplied by warehouse compute rate  
D) Model test coverage percentage

**Answer:** C) Total execution time multiplied by warehouse compute rate

**Explanation:**

- Cost = execution time × warehouse rate × run frequency
- This calculation identifies highest-cost models
- Dependencies (A) don't directly indicate cost
- Documentation (B) and tests (D) don't affect compute costs

### Question 6: Catalog Usage

**Question:** What is the PRIMARY purpose of dbt Catalog (formerly dbt Explorer)?

A) To write dbt models  
B) To understand lineage, monitor performance, and discover models  
C) To execute dbt jobs  
D) To configure warehouse connections

**Answer:** B) To understand lineage, monitor performance, and discover models

**Explanation:**

- dbt Catalog is a metadata interface for understanding and optimizing projects
- Not for writing models (A) - that's done in IDE
- Not for executing jobs (C) - that's job scheduling
- Not for configuration (D) - that's in project settings

## Summary

dbt Catalog is essential for understanding, monitoring, and optimizing dbt projects at scale. Key takeaways:

1. **Lineage visualization**: Understand data flow and dependencies

2. **Performance insights**: Identify slow models and optimization opportunities

3. **Model discovery**: Find public models across projects

4. **Impact analysis**: Assess downstream effects before changes

5. **Cost management**: Calculate and optimize warehouse spend

6. **Column-level lineage**: Trace specific columns through pipelines (premium)

7. **Regular reviews**: Establish cadence for lineage and performance audits

8. **Documentation**: Keep Catalog metadata current and comprehensive

## Exam Preparation Complete

Congratulations! You have completed all 8 topics for the dbt Architect Certification Exam. Review the [README](./README.md) for study strategies and next steps.

### Recommended Study Approach

1. **Week 1-2**: Topics 1-4 (Infrastructure and orchestration)
2. **Week 3-4**: Topics 5-8 (Governance, monitoring, architecture)
3. **Week 5**: Practice questions from all topics
4. **Week 6**: Hands-on labs and final review

Good luck with your certification exam!
