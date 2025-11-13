# Topic 6: Setting up Monitoring and Alerting for Jobs

## Overview

Effective monitoring and alerting ensures data pipelines run reliably and stakeholders are informed of both successes and failures. This topic covers notification strategies, webhook integrations, and proactive monitoring approaches.

## Key Concepts and Definitions

### Job Monitoring

Tracking job execution status, performance metrics, and data quality:

- **Run status**: Success, failure, cancelled, skipped
- **Execution time**: Duration and trends
- **Model-level results**: Which models succeeded/failed
- **Test results**: Data quality checks
- **Resource utilization**: Warehouse compute usage

### Notification Types

#### Email Notifications

Built-in feature sending alerts to specified addresses:

- Job completion (success/failure)
- Configurable per job
- Supports multiple recipients
- Simple setup, no code required

#### Webhooks

HTTP callbacks for event-driven integrations:

- POST requests to external endpoints
- Custom payloads with run metadata
- Enables sophisticated workflows
- Integrates with external systems

### Webhook Events

Types of events that can trigger webhooks:

- `job.run.started`
- `job.run.completed`
- `job.run.error`
- `job.run.cancelled`

## Practical Examples

### Example 1: Basic Email Notifications

```markdown
Navigate to: Job Settings → Notifications

Email Configuration:
  ☑ Email notifications enabled
  
  Notify on:
    ☑ Job failure
    ☑ Job success (for critical jobs only)
    ☐ Job cancelled
  
  Recipients:
    - data-team@company.com
    - ops-alerts@company.com
    - specific-owner@company.com
  
  Include in email:
    ☑ Error messages
    ☑ Run duration
    ☑ Link to run details
```

**Email Template:**

```
Subject: [dbt Cloud] Daily Production Run - Failed

Job: Daily Production Run
Status: Failed
Duration: 15 minutes
Started: 2024-01-15 06:00:00 UTC
Finished: 2024-01-15 06:15:00 UTC

Failed Models:
  - dim_customers (5 errors)
  - fct_orders (relation does not exist)

Error Details:
  Database Error in model dim_customers (models/marts/dim_customers.sql)
  column "invalid_column" does not exist

View full run details:
https://cloud.getdbt.com/deploy/12345/projects/67890/runs/111213/
```

### Example 2: Webhook Configuration for Slack

**Step 1: Create Slack Incoming Webhook**

```markdown
In Slack:
1. Go to Apps → Incoming Webhooks
2. Add to your workspace
3. Choose channel: #data-alerts
4. Copy webhook URL: https://hooks.slack.com/services/T00/B00/XXX
```

**Step 2: Configure dbt Cloud Webhook**

```markdown
Navigate to: Job Settings → Webhooks

Add Webhook:
  Event Types:
    ☑ run.completed
    ☑ run.error
    
  Endpoint URL:
    https://hooks.slack.com/services/T00/B00/XXX
    
  Custom Headers:
    Content-Type: application/json
```

**Step 3: Transform Payload (Using Middleware Service)**

```python
# Lambda function or Cloud Function to transform dbt webhook to Slack format
import json
import requests
from datetime import datetime

def lambda_handler(event, context):
    dbt_payload = json.loads(event['body'])
    
    # Extract dbt Cloud run details
    run_id = dbt_payload['data']['runId']
    job_name = dbt_payload['data']['jobName']
    run_status = dbt_payload['data']['runStatus']
    run_duration = dbt_payload['data']['durationSeconds']
    run_url = dbt_payload['data']['href']
    
    # Determine color based on status
    color = "good" if run_status == "Success" else "danger"
    
    # Build Slack message
    slack_message = {
        "attachments": [{
            "color": color,
            "title": f"dbt Job: {job_name}",
            "title_link": run_url,
            "fields": [
                {
                    "title": "Status",
                    "value": run_status,
                    "short": True
                },
                {
                    "title": "Duration",
                    "value": f"{run_duration}s",
                    "short": True
                },
                {
                    "title": "Run ID",
                    "value": run_id,
                    "short": True
                }
            ],
            "footer": "dbt Cloud",
            "ts": int(datetime.now().timestamp())
        }]
    }
    
    # Send to Slack
    slack_webhook_url = dbt_payload['hookUrl']  # Or from environment
    response = requests.post(slack_webhook_url, json=slack_message)
    
    return {
        'statusCode': 200,
        'body': json.dumps('Notification sent')
    }
```

### Example 3: PagerDuty Integration for Critical Failures

```python
# Webhook receiver that creates PagerDuty incidents
from flask import Flask, request
import requests
import json

app = Flask(__name__)

PAGERDUTY_INTEGRATION_KEY = "your_integration_key"
CRITICAL_JOBS = ["Production ETL", "Revenue Calculation", "Customer Data Sync"]

@app.route('/webhook/dbt', methods=['POST'])
def dbt_webhook():
    payload = request.json
    
    job_name = payload['data']['jobName']
    run_status = payload['data']['runStatus']
    run_url = payload['data']['href']
    
    # Only create incident for critical job failures
    if job_name in CRITICAL_JOBS and run_status == "Error":
        create_pagerduty_incident(job_name, run_url, payload)
    
    return {'status': 'received'}, 200

def create_pagerduty_incident(job_name, run_url, dbt_payload):
    incident = {
        "routing_key": PAGERDUTY_INTEGRATION_KEY,
        "event_action": "trigger",
        "payload": {
            "summary": f"CRITICAL: dbt Job Failed - {job_name}",
            "severity": "error",
            "source": "dbt Cloud",
            "custom_details": {
                "job_name": job_name,
                "run_url": run_url,
                "run_id": dbt_payload['data']['runId'],
                "failed_models": extract_failed_models(dbt_payload)
            }
        },
        "links": [{
            "href": run_url,
            "text": "View dbt Cloud Run"
        }]
    }
    
    response = requests.post(
        "https://events.pagerduty.com/v2/enqueue",
        json=incident,
        headers={"Content-Type": "application/json"}
    )
    
    return response

def extract_failed_models(payload):
    # Parse run artifacts to find failed models
    # Implementation depends on payload structure
    return payload.get('data', {}).get('failedModels', [])

if __name__ == '__main__':
    app.run(port=5000)
```

### Example 4: Custom Monitoring Dashboard

```python
# Script to fetch job run history and create monitoring dashboard
import requests
import pandas as pd
from datetime import datetime, timedelta
import matplotlib.pyplot as plt

API_TOKEN = "your_token"
ACCOUNT_ID = "12345"
PROJECT_ID = "67890"

def get_run_history(days=7):
    """Fetch job run history from dbt Cloud API"""
    end_date = datetime.now()
    start_date = end_date - timedelta(days=days)
    
    url = f"https://cloud.getdbt.com/api/v2/accounts/{ACCOUNT_ID}/runs/"
    headers = {"Authorization": f"Token {API_TOKEN}"}
    params = {
        "project_id": PROJECT_ID,
        "order_by": "-finished_at",
        "offset": 0,
        "limit": 100
    }
    
    response = requests.get(url, headers=headers, params=params)
    runs = response.json()['data']
    
    return pd.DataFrame(runs)

def calculate_metrics(df):
    """Calculate key performance metrics"""
    metrics = {
        "total_runs": len(df),
        "success_rate": (df['status'] == 10).sum() / len(df) * 100,
        "avg_duration": df['duration_humanized'].mean(),
        "failures_last_7d": (df['status'] == 20).sum(),
        "slowest_jobs": df.nlargest(5, 'duration')[['job_id', 'job_name', 'duration']]
    }
    return metrics

def generate_alerts(df):
    """Generate alerts based on anomalies"""
    alerts = []
    
    # Alert if success rate drops below 95%
    success_rate = (df['status'] == 10).sum() / len(df) * 100
    if success_rate < 95:
        alerts.append(f"⚠️ Success rate dropped to {success_rate:.1f}%")
    
    # Alert if any job consistently slow
    for job_id in df['job_id'].unique():
        job_runs = df[df['job_id'] == job_id]
        if len(job_runs) >= 5:
            avg_duration = job_runs['duration'].mean()
            recent_duration = job_runs.head(3)['duration'].mean()
            
            if recent_duration > avg_duration * 1.5:
                job_name = job_runs.iloc[0]['job_name']
                alerts.append(
                    f"⚠️ Job '{job_name}' running 50% slower than average"
                )
    
    return alerts

# Usage
df = get_run_history(days=7)
metrics = calculate_metrics(df)
alerts = generate_alerts(df)

print("=== dbt Cloud Monitoring Report ===")
print(f"Total Runs: {metrics['total_runs']}")
print(f"Success Rate: {metrics['success_rate']:.1f}%")
print(f"Failures (7d): {metrics['failures_last_7d']}")
print("\nAlerts:")
for alert in alerts:
    print(alert)
```

### Example 5: Data Quality Monitoring with Monte Carlo

```yaml
# Monte Carlo integration via webhook
# When dbt test fails, create Monte Carlo incident

Webhook Configuration in dbt Cloud:
  Event: run.completed
  URL: https://api.getmontecarlo.com/v1/integrations/dbt-cloud
  Headers:
    Authorization: Bearer YOUR_MC_TOKEN
    Content-Type: application/json

Monte Carlo Configuration:
  Integration: dbt Cloud
  Actions on test failure:
    - Create incident
    - Notify data owners
    - Link to dbt documentation
    - Suggest related monitors
```

## Best Practices

### 1. Tiered Alerting Strategy

```markdown
Tier 1: Critical (PagerDuty/Phone)
  - Production revenue models failed
  - Customer-facing data stale > 4 hours
  - Security/compliance data issues
  - Multiple consecutive failures
  Response: Immediate (24/7)

Tier 2: High Priority (Slack/Email)
  - Non-critical production failures
  - Staging environment issues
  - Performance degradation
  - Data quality warnings
  Response: Within business hours

Tier 3: Informational (Email Digest)
  - Successful completions
  - Duration trends
  - Usage statistics
  - Optimization recommendations
  Response: Review daily/weekly

Tier 4: Metrics Only (Dashboard)
  - All runs tracked
  - Historical trends
  - Resource utilization
  - Model performance
  Response: Proactive monitoring
```

### 2. Alert Fatigue Prevention

```markdown
❌ Avoid:
  - Alerting on every successful run
  - Duplicate notifications to same channel
  - Overly verbose messages
  - Alerts for expected behaviors

✓ Instead:
  - Alert only on anomalies
  - Consolidate related failures
  - Provide actionable information
  - Implement quiet hours for non-critical alerts
  - Use summary digests for high-frequency events

Example:
  Instead of: 50 emails for 50 successful hourly runs
  Use: Daily digest with success rate and anomalies
```

### 3. Actionable Alert Content

```markdown
Poor Alert:
  "Job failed"
  
Good Alert:
  Subject: [CRITICAL] Revenue Model Failed - Production
  
  What: fct_revenue model failed
  When: 2024-01-15 06:15 UTC (15 min ago)
  Impact: Revenue dashboard showing stale data
  
  Error: Column 'discount_amount' not found
  Likely Cause: Upstream schema change in source.orders
  
  Actions:
    1. Check source schema: SELECT * FROM raw.orders LIMIT 1
    2. Update model if schema changed: models/marts/fct_revenue.sql
    3. Review recent source changes: Git history
  
  Run Details: [Link]
  Runbook: [Link]
  On-call: @data-engineer-oncall
```

### 4. Webhook Security

```markdown
Secure Webhook Configuration:

1. Use HTTPS only
   ✗ http://example.com/webhook
   ✓ https://example.com/webhook

2. Implement verification
   - Secret tokens in headers
   - Request signature validation
   - IP allowlisting

3. Example verification:
```python
import hmac
import hashlib

def verify_webhook(request):
    signature = request.headers.get('X-dbt-Signature')
    secret = os.environ['DBT_WEBHOOK_SECRET']
    
    body = request.get_data()
    expected_signature = hmac.new(
        secret.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(signature, expected_signature)

@app.route('/webhook', methods=['POST'])
def webhook_handler():
    if not verify_webhook(request):
        return {'error': 'Invalid signature'}, 403
    
    # Process webhook
    return {'status': 'ok'}, 200
```

```markdown

### 5. Monitoring Coverage

```markdown
What to Monitor:

Job-Level:
  ✓ Success/failure rates
  ✓ Execution duration trends
  ✓ Resource utilization
  ✓ Schedule adherence
  ✓ Dependency wait times

Model-Level:
  ✓ Individual model success rates
  ✓ Row count changes (unexpected spikes/drops)
  ✓ Build times per model
  ✓ Incremental efficiency

Test-Level:
  ✓ Test pass/fail rates
  ✓ Specific test failures
  ✓ Data quality trends
  ✓ Test coverage

Environment-Level:
  ✓ Warehouse query performance
  ✓ Concurrent job execution
  ✓ API rate limits
  ✓ Storage usage
```

### 6. Escalation Paths

```markdown
Incident Escalation Policy:

Level 1: Automated Response (0-5 min)
  - Webhook triggers alert
  - Notification sent to primary channel
  - Automated diagnostics run
  - Runbook link provided

Level 2: On-Call Engineer (5-15 min)
  - No acknowledgment after 5 min
  - Page on-call engineer
  - Escalate to Slack/PagerDuty
  - Incident ticket created

Level 3: Team Lead (15-30 min)
  - Issue not resolved in 15 min
  - Escalate to team lead
  - Assess need for additional resources
  - Stakeholder notification

Level 4: Management (30+ min)
  - Extended outage (> 30 min)
  - Business-critical impact
  - Management notification
  - Post-incident review scheduled
```

## Anti-Patterns (What NOT to Do)

### ❌ DON'T Send All Notifications to Everyone

```markdown
# BAD: Notification overload
Email Recipients:
  - entire-company@company.com
  - all-engineers@company.com
  - executives@company.com

Result:
  - Important alerts buried
  - Alert fatigue
  - People ignore all notifications
```

### ❌ DON'T Alert Without Context

```markdown
# BAD: Vague alert
"Job 12345 failed"

# GOOD: Contextual alert
"Production Revenue Job Failed
- Impact: Q4 revenue dashboard stale
- Failed Model: fct_revenue_daily
- Error: Source table not found
- Action: Check upstream ETL job status
- Runbook: [link]"
```

### ❌ DON'T Ignore Webhook Failures

```markdown
# BAD: Fire and forget
Webhook sent → Assume success → Move on

# GOOD: Verify delivery
Webhook sent → Check response code → Log failures → Retry if needed

Example:
```python
response = requests.post(webhook_url, json=payload, timeout=10)

if response.status_code != 200:
    logger.error(f"Webhook failed: {response.status_code}")
    # Store in dead letter queue for retry
    retry_queue.add(payload)
```

```markdown

### ❌ DON'T Mix Critical and Non-Critical Alerts

```markdown
# BAD: Same channel for everything
#data-alerts:
  - "Hourly job succeeded" (100x per day)
  - "PRODUCTION DOWN" (lost in noise)

# GOOD: Separate channels by severity
#data-critical: Production failures only
#data-alerts: Standard notifications
#data-info: Success confirmations, metrics
```

### ❌ DON'T Forget to Test Alert Systems

```markdown
# BAD: Never test until real incident
Deploy webhooks → Hope they work

# GOOD: Regular testing
- Monthly test alerts
- Verify end-to-end flow
- Check escalation paths
- Update contact information
- Test failover scenarios
```

## Real-World Scenarios and Solutions

### Scenario 1: Implementing SLA Monitoring

**Challenge:** Ensure critical data is available by specific times for business reporting.

**Solution:**

```python
# SLA Monitoring System
from datetime import datetime, time
import pytz

SLAS = {
    "revenue_dashboard": {
        "job_name": "Daily Revenue Calculation",
        "due_time": time(8, 0),  # 8 AM
        "timezone": "America/New_York",
        "severity": "critical"
    },
    "customer_analytics": {
        "job_name": "Customer Metrics Update",
        "due_time": time(9, 30),  # 9:30 AM
        "timezone": "America/New_York",
        "severity": "high"
    }
}

def check_sla_compliance(job_name, completion_time):
    """Check if job completed within SLA"""
    if job_name not in SLAS:
        return None
    
    sla = SLAS[job_name]
    tz = pytz.timezone(sla['timezone'])
    
    # Convert completion time to SLA timezone
    completion_local = completion_time.astimezone(tz)
    sla_deadline = datetime.combine(
        completion_local.date(),
        sla['due_time']
    ).replace(tzinfo=tz)
    
    # Calculate SLA metrics
    on_time = completion_local <= sla_deadline
    delay = (completion_local - sla_deadline).total_seconds() / 60  # minutes
    
    return {
        "on_time": on_time,
        "delay_minutes": max(0, delay),
        "severity": sla['severity'],
        "completion_time": completion_local,
        "sla_deadline": sla_deadline
    }

def handle_sla_violation(job_name, sla_result):
    """Handle SLA violations with appropriate alerting"""
    if not sla_result['on_time']:
        delay = sla_result['delay_minutes']
        severity = sla_result['severity']
        
        if severity == "critical":
            # Page on-call
            send_pagerduty_alert(
                summary=f"CRITICAL SLA VIOLATION: {job_name}",
                details=f"Delayed by {delay:.0f} minutes"
            )
        elif delay > 30:
            # Major delay, escalate
            send_slack_alert(
                channel="#data-critical",
                message=f"⚠️ SLA VIOLATION: {job_name} delayed by {delay:.0f} min"
            )
        else:
            # Minor delay, inform
            send_slack_alert(
                channel="#data-alerts",
                message=f"ℹ️ {job_name} completed {delay:.0f} min late"
            )

# Webhook handler
@app.route('/webhook/sla-check', methods=['POST'])
def sla_webhook():
    payload = request.json
    job_name = payload['data']['jobName']
    finished_at = datetime.fromisoformat(payload['data']['finishedAt'])
    
    sla_result = check_sla_compliance(job_name, finished_at)
    if sla_result:
        handle_sla_violation(job_name, sla_result)
    
    return {'status': 'checked'}, 200
```

### Scenario 2: Cross-System Monitoring Integration

**Challenge:** Integrate dbt Cloud monitoring with existing observability stack (Datadog).

**Solution:**

```python
# dbt Cloud → Datadog Integration
from datadog import initialize, api, statsd
import requests

# Initialize Datadog
options = {
    'api_key': 'your_api_key',
    'app_key': 'your_app_key'
}
initialize(**options)

def send_dbt_metrics_to_datadog(run_data):
    """Send dbt job metrics to Datadog"""
    job_name = run_data['jobName']
    status = run_data['runStatus']
    duration = run_data['durationSeconds']
    
    # Send duration metric
    statsd.histogram(
        'dbt.job.duration',
        duration,
        tags=[f'job:{job_name}', f'status:{status}']
    )
    
    # Send status metric (1 for success, 0 for failure)
    success = 1 if status == 'Success' else 0
    statsd.gauge(
        'dbt.job.success',
        success,
        tags=[f'job:{job_name}']
    )
    
    # Create event for failures
    if status != 'Success':
        api.Event.create(
            title=f"dbt Job Failed: {job_name}",
            text=f"Job failed with status: {status}",
            tags=[f'job:{job_name}', 'source:dbt_cloud'],
            alert_type='error',
            priority='normal'
        )
    
    # Send model-level metrics
    for model in run_data.get('models', []):
        model_duration = model.get('executionTime', 0)
        model_status = model.get('status', 'unknown')
        
        statsd.histogram(
            'dbt.model.duration',
            model_duration,
            tags=[
                f'job:{job_name}',
                f'model:{model["name"]}',
                f'status:{model_status}'
            ]
        )

# Datadog Monitor Configuration (via UI or API)
monitor_config = {
    "name": "dbt Job Failure Rate",
    "type": "metric alert",
    "query": "avg(last_1h):avg:dbt.job.success{*} by {job} < 0.95",
    "message": """
    dbt job {{job.name}} has failed more than 5% of runs in the past hour.
    
    Check dbt Cloud: https://cloud.getdbt.com/
    Runbook: https://wiki.company.com/dbt-failures
    
    @slack-data-team @pagerduty-data-oncall
    """,
    "options": {
        "notify_no_data": True,
        "no_data_timeframe": 60,
        "thresholds": {
            "critical": 0.95,
            "warning": 0.98
        }
    }
}
```

## Sample Exam Questions

### Question 1: Email Notifications

**Question:** You want to be notified when a production job fails, but NOT when it succeeds (to avoid alert fatigue). How should you configure email notifications?

A) Enable "Notify on all job completions"  
B) Enable "Notify on job failure" only  
C) Create a webhook to filter events  
D) Email notifications cannot be configured per success/failure

**Answer:** B) Enable "Notify on job failure" only

**Explanation:**

- dbt Cloud allows separate configuration for success and failure notifications
- Enabling only failure notifications reduces noise
- This is a standard configuration option in job settings

### Question 2: Webhook Events

**Question:** You want to trigger an external system only when a dbt job completes successfully. Which webhook event should you configure?

A) job.run.started  
B) job.run.completed (filtered to status=success in your handler)  
C) job.run.success  
D) job.run.error

**Answer:** B) job.run.completed (filtered to status=success in your handler)

**Explanation:**

- `job.run.completed` fires for all completions
- The payload includes status information
- Your webhook handler filters based on status
- There is no separate `job.run.success` event type

### Question 3: Alert Priority

**Question:** Which of the following should be configured as HIGHEST priority alerts (e.g., PagerDuty)?

A) Development environment job failures  
B) Production revenue calculation failures  
C) Documentation generation delays  
D) CI job failures on feature branches

**Answer:** B) Production revenue calculation failures

**Explanation:**

- Revenue-critical production failures have immediate business impact
- Development and CI failures are important but not critical
- Documentation delays are low priority
- Alert priority should match business criticality

### Question 4: Webhook Security

**Question:** What is the MOST important security practice when implementing webhook receivers?

A) Using HTTPS for the webhook endpoint  
B) Logging all webhook payloads  
C) Sending acknowledgment responses  
D) Using GET instead of POST requests

**Answer:** A) Using HTTPS for the webhook endpoint

**Explanation:**

- HTTPS encrypts data in transit
- Prevents interception of potentially sensitive job information
- Logging is good practice but not primarily a security measure
- GET requests for webhooks would be insecure (visible in logs/history)

### Question 5: Alert Fatigue

**Question:** Your team receives 200+ email notifications daily for successful hourly job runs. What is the BEST approach to reduce alert fatigue?

A) Disable all notifications  
B) Configure notifications for failures only, create a daily success summary  
C) Send all notifications to a separate email address  
D) Increase email filters to automatically archive these emails

**Answer:** B) Configure notifications for failures only, create a daily success summary

**Explanation:**

- Failures require immediate attention; successes are informational
- Daily summaries provide visibility without noise
- Disabling all notifications (A) risks missing critical failures
- Alternative email (C) doesn't solve the underlying problem
- Email filters (D) mask symptoms rather than fixing root cause

### Question 6: Monitoring Integration

**Question:** You want to track dbt job duration trends over time in your existing Datadog monitoring system. What's the best approach?

A) Manually log into dbt Cloud daily and record durations  
B) Configure a webhook that sends job metrics to Datadog API  
C) Export dbt logs to CSV and upload to Datadog  
D) Datadog integration is not possible with dbt Cloud

**Answer:** B) Configure a webhook that sends job metrics to Datadog API

**Explanation:**

- Webhooks enable automated, real-time integration
- Can send metrics to Datadog API programmatically
- Manual logging (A) and CSV exports (C) are not scalable
- Many observability platforms integrate well with webhook-based systems

## Summary

Effective monitoring and alerting keeps data pipelines reliable and teams informed. Key takeaways:

1. **Use tiered alerting**: Critical (page), high (Slack), info (email digest)

2. **Prevent alert fatigue**: Only alert on anomalies, consolidate messages

3. **Provide actionable information**: Include error details, impact, and next steps

4. **Secure webhooks**: Use HTTPS, verify signatures, validate payloads

5. **Integrate with existing tools**: Slack, PagerDuty, Datadog, etc.

6. **Monitor comprehensively**: Jobs, models, tests, and SLAs

7. **Test alert systems** regularly to ensure they work when needed

8. **Establish escalation paths** for incident response

## Next Steps

Continue to [Topic 7: Setting up a dbt Mesh and Leveraging Cross-Project References](./07-dbt-mesh.md) to learn about multi-project architecture.
