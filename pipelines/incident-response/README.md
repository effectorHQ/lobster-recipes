# Incident Response Pipeline

Automated incident detection, diagnostics, and team notification with intelligent root cause analysis and comprehensive incident tracking.

## Overview

This pipeline provides rapid incident response by automating:

1. **Health Checks** — Verify service health across all critical endpoints
2. **Metrics Collection** — Gather CPU, memory, error rates, and latency
3. **Log Aggregation** — Pull recent error logs from all sources
4. **Deployment Analysis** — Check for recent deployments as root cause
5. **Infrastructure Diagnostics** — Inspect pod status and restarts
6. **Root Cause Analysis** — AI-driven analysis of symptoms
7. **Incident Ticketing** — Create Jira incident with diagnostics
8. **On-Call Notification** — Alert on-call engineer immediately
9. **Timeline Document** — Start incident tracking in Notion
10. **Enhanced Monitoring** — Escalate monitoring during recovery

## Features Demonstrated

- **Parallel Health Checks** — API, database, and cache checks run concurrently
- **Conditional Steps** — Enhanced monitoring only for high severity
- **Root Cause Analysis** — Intelligent evaluation of multiple data sources
- **Rich Notifications** — Slack blocks with actionable buttons
- **External Integrations** — Jira, PagerDuty, Notion, Prometheus, Kubernetes
- **Error Recovery** — Graceful handling of diagnostic failures

## Prerequisites

- Lobster CLI configured
- Kubernetes cluster access (`kubectl` configured)
- Prometheus metrics available
- Centralized logging (ELK, CloudWatch, Datadog, etc.)
- Jira project for incident tracking
- Slack workspace with webhook
- PagerDuty account for on-call scheduling
- Notion workspace for timeline documents

## Configuration

### Environment Variables

```bash
# Service configuration
export SERVICE_NAME=api-service          # Service to monitor
export ENVIRONMENT=production            # Deployment environment

# Incident details
export SEVERITY=high                     # low|medium|high|critical
export TRIGGERED_BY=prometheus-alert     # Source of incident trigger

# API endpoints (customize for your infrastructure)
export API_HEALTH_URL=https://prod-api.example.com/health
export DB_HOST=prod-db.example.com
export CACHE_HOST=prod-cache.example.com
```

### Credentials

```bash
export JIRA_API_TOKEN=xxxxx
export SLACK_WEBHOOK=https://hooks.slack.com/services/...
export PAGERDUTY_API_KEY=xxxxx
export NOTION_API_KEY=xxxxx
export PROMETHEUS_URL=https://prometheus.example.com
```

### Service Configuration

Customize endpoints in `pipeline.yml`:

```yaml
steps:
  - id: check-api-health
    params:
      url: "https://${ENVIRONMENT}-api.example.com/health"
```

## Running the Pipeline

### Manual Trigger

```bash
openclaw pipeline run incident-response \
  --env SERVICE_NAME=api-service \
  --env SEVERITY=high \
  --env ENVIRONMENT=production
```

### Triggered by Alert (Prometheus)

Configure Prometheus alert rule:

```yaml
groups:
  - name: incident_response
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        annotations:
          summary: "High error rate in {{ $labels.service }}"
        actions:
          - type: webhook
            url: https://your-lobster/webhooks/incident
            params:
              SERVICE_NAME: "{{ $labels.service }}"
              SEVERITY: "high"
              TRIGGERED_BY: "prometheus"
```

### Programmatic Trigger

```bash
curl -X POST https://your-lobster/pipelines/incident-response/run \
  -H "Content-Type: application/json" \
  -d '{
    "SERVICE_NAME": "api-service",
    "SEVERITY": "critical",
    "ENVIRONMENT": "production"
  }'
```

### Dry Run (Test Configuration)

```bash
openclaw pipeline run incident-response \
  --env SERVICE_NAME=api-service \
  --env SEVERITY=medium \
  --dry-run
```

## Step Details

### Health Checks (Parallel)

**check-api-health** — GET /health endpoint
- Expected status: 200
- Timeout: 5 seconds

**check-database-health** — Database connectivity
- Ping database host
- Timeout: 5 seconds

**check-cache-health** — Redis cache health
- Ping cache host
- Timeout: 5 seconds

### Metrics Collection

**gather-metrics** — Prometheus queries for:
- CPU usage (rate)
- Memory usage (bytes converted to MB)
- Error rate (5xx responses)
- Latency P95 percentile
- Active connections

Time range: Last 1 hour

### Log Aggregation

**get-application-logs** — Application errors
- Filter: ERROR and CRITICAL level logs
- Time range: Last 30 minutes
- Limit: 100 entries

**get-infrastructure-logs** — Deployment and crash logs
- Filter: deployment events, crashes, restarts
- Time range: Last 1 hour
- Limit: 50 entries

### Deployment Analysis

**get-recent-deployments** — Check recent deployments
- Retrieves last 5 deployments
- Looks for failed deployments
- Correlates with incident timing

### Pod Status

**check-pod-status** — Kubernetes pod inspection
- Lists pods in namespace matching service
- Checks restart counts
- Identifies unstable pods

### Root Cause Analysis

**analyze-incident** — Evaluates:
- Database connectivity issues
- Cache degradation
- Application error spikes
- Recent deployment correlation
- Pod restart patterns

Produces output with:
- `rootCause` — Identified root cause
- `severity` — Incident severity level
- `affectedComponents` — Array of affected services
- `errorRate` — Current error rate
- `latencyP95` — P95 latency
- `recommendedAction` — Suggested mitigation

### Incident Ticketing

**create-jira-ticket** — Creates incident ticket with:
- Auto-prioritized based on SEVERITY
- Comprehensive diagnostics snapshot
- Links to logs and metrics
- Recent deployment information
- Labels for filtering

### On-Call Notification

**get-oncall-engineer** — Retrieves on-call engineer from PagerDuty

**notify-oncall** — Sends Slack notification with:
- Incident ID and service
- Root cause analysis
- Error rate and latency
- Affected components
- Quick-action buttons (logs, Jira, metrics)

### Timeline Document

**create-timeline-doc** — Notion page with:
- Incident metadata
- Timeline section for updates
- Investigation findings
- Links to Jira ticket
- Ready for post-incident review

## Severity Levels

| Severity | Priority | Response | Examples |
|----------|----------|----------|----------|
| **Low** | P4 | Standard | Isolated request failures, single user impact |
| **Medium** | P3 | Prompt | Elevated error rate, performance degradation |
| **High** | P2 | Urgent | Multiple services affected, significant user impact |
| **Critical** | P1 | Immediate | Service down, data loss risk, security incident |

## Example Workflow

### T+0 minutes — Detection
Prometheus alert fires for error rate > 5%
→ Incident response pipeline auto-triggered
→ Health checks run in parallel
→ Metrics gathered

### T+1 minute — Analysis
- Database returns healthy
- Cache returns healthy
- Error rate confirmed at 8%
- Recent deployment found (3 minutes ago)
- Analysis suggests: "recent deployment side effect"

### T+2 minutes — Response
- Jira ticket created as P2 incident
- On-call engineer notified via Slack
- Timeline document started
- Enhanced monitoring enabled

### T+3 minutes — Investigation
On-call engineer checks logs, decides to rollback recent deployment

### T+8 minutes — Recovery
Deployment rolled back
Error rate drops to 0.1%
Incident marked resolved in Jira
Post-incident review scheduled

## Customization Examples

### Add Custom Health Check

Insert new health check step:

```yaml
- id: check-queue-health
  skill: http
  action: get
  params:
    url: "https://${ENVIRONMENT}-queue.example.com/status"
```

### Modify Root Cause Logic

Edit `analyze-incident` logic to add:

```javascript
if (metrics.disk_usage > 0.9) {
  rootCause = "disk space exhaustion";
  affectedComponents.push("disk");
}
```

### Add Custom Notifications

Insert new notification step:

```yaml
- id: notify-customers
  skill: email
  action: send
  condition: "${SEVERITY == 'critical'}"
  params:
    to: "support@example.com"
    subject: "Service Status Update"
    body: "We are investigating..."
```

### Extend Incident Documentation

Modify `create-timeline-doc` template to include:

```yaml
affectedUsers: "${gather-metrics.affected_user_count}"
estimatedImpact: "${analyze-incident.impact_percentage}"
businessImpact: "Revenue impact: $XXX per minute"
```

### Auto-Scaling on High Error Rates

Add remediation step:

```yaml
- id: scale-up-deployment
  skill: kubernetes
  action: scale
  condition: "${analyze-incident.errorRate > 0.1}"
  params:
    deployment: "${SERVICE_NAME}"
    namespace: "${ENVIRONMENT}"
    replicas: 5  # Scale up
```

## Monitoring and Debugging

### Check Incident Responses

```bash
openclaw pipeline runs incident-response --limit 20
```

### View Specific Incident Details

```bash
openclaw pipeline logs incident-response --id incident-123
```

### Validate Jira Connection

```bash
curl -u user:token https://jira.example.com/rest/api/3/myself
```

### Test Slack Notification

```bash
curl -X POST ${SLACK_WEBHOOK} \
  -H 'Content-Type: application/json' \
  -d '{"text":"Test notification"}'
```

### Check PagerDuty On-Call

```bash
curl -H "Authorization: Token token=${PAGERDUTY_API_KEY}" \
  "https://api.pagerduty.com/schedules/SERVICE_SCHEDULE_ID/users"
```

## Performance Considerations

- **Parallel Execution** — Health checks run concurrently (4 max steps)
- **No Caching** — Fresh data only for real-time incident response
- **5-Minute Timeout** — Quick incident detection and response
- **Resumable** — Can resume analysis if individual checks fail

## SLA Targets

| Step | Target | Actual |
|------|--------|--------|
| Health checks | < 30s | ~10-15s |
| Metrics collection | < 30s | ~15-20s |
| Root cause analysis | < 30s | ~10-15s |
| Jira ticket creation | < 30s | ~15-20s |
| On-call notification | < 30s | ~5-10s |
| **Total Response Time** | **< 2.5 min** | **~1-2 min** |

## Post-Incident Review

After incident resolved:

1. Check timeline document in Notion
2. Review root cause findings
3. Create action items in Jira
4. Schedule postmortem with team
5. Update runbooks based on findings

## Contributing

Found improvements? See [CONTRIBUTING.md](../../CONTRIBUTING.md).

## License


This project is currently licensed under the Apache 2.0 License 。

