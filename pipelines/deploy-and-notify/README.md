# Deploy and Notify Pipeline

Automate your complete deployment workflow from code to production with automated health checks, smoke tests, and team notifications.

## Overview

This pipeline demonstrates a production-grade deployment workflow:

1. **Build** — Compile Docker image with build metadata
2. **Push** — Upload image to container registry
3. **Deploy** — Apply Kubernetes manifests with image overrides
4. **Verify** — Run health checks with exponential backoff retry
5. **Test** — Execute smoke tests against deployed service
6. **Notify** — Alert team of successful deployment
7. **Handle Failures** — Create incident tickets and notify team on errors

## Features Demonstrated

- **Retry Logic** — Health checks use exponential backoff (5s → 7.5s → 11.25s → 16.9s → 25.3s)
- **Error Handling** — `on_failure` hooks create Jira tickets and send Slack alerts
- **Variable Interpolation** — Dynamic image tags, namespaces, and URLs
- **Sequential Dependencies** — Steps enforce correct ordering (build → push → deploy → verify)
- **Environment Configuration** — Configurable registries, namespaces, and URLs
- **Timeout Management** — Individual and pipeline-level timeouts prevent hanging steps

## Prerequisites

- Lobster CLI installed and configured
- Docker daemon access for image building
- Kubernetes cluster (staging) with `kubectl` configured
- Container registry credentials (Docker Hub, ECR, etc.)
- Slack workspace with webhook URL
- Jira project for incident tracking (optional)

## Configuration

### Environment Variables

Set these before running the pipeline:

```bash
export IMAGE_TAG=v1.2.3                    # Docker image tag
export DOCKER_REGISTRY=docker.io           # Registry URL (optional)
export SLACK_WEBHOOK=https://hooks.slack.com/...  # Slack webhook
```

### Customization

Edit `pipeline.yml` to customize:

| Parameter | Default | Purpose |
|-----------|---------|---------|
| `DOCKER_REGISTRY` | `docker.io` | Container registry |
| `STAGING_NAMESPACE` | `staging` | Kubernetes namespace |
| `HEALTH_CHECK_URL` | `https://staging.example.com/health` | Service health endpoint |
| `TEST_TIMEOUT` | `300` | Smoke test timeout (seconds) |

### Kubernetes Manifest

Ensure your manifest (`k8s/staging.yml`) includes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: staging
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: docker.io/myapp:latest  # Will be overridden
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
```

## Running the Pipeline

### Basic Usage

```bash
openclaw pipeline run deploy-and-notify --env IMAGE_TAG=v1.2.3
```

### With Custom Registry

```bash
openclaw pipeline run deploy-and-notify \
  --env IMAGE_TAG=v1.2.3 \
  --env DOCKER_REGISTRY=registry.example.com
```

### Dry Run (Preview Only)

```bash
openclaw pipeline run deploy-and-notify \
  --env IMAGE_TAG=v1.2.3 \
  --dry-run
```

### Resume from Failed Step

If a deployment fails, resume from the health check step:

```bash
openclaw pipeline resume deploy-and-notify \
  --from health-check \
  --env IMAGE_TAG=v1.2.3
```

## Step Details

### 1. build-image
Builds Docker image with build metadata (date, git SHA).

**Timeout:** 600s (10 minutes)

### 2. push-image
Pushes built image to configured registry.

**Timeout:** 300s (5 minutes)

### 3. deploy-staging
Applies Kubernetes manifest and overrides image tag.

**Timeout:** 300s (5 minutes)

### 4. health-check
Verifies service is responsive with retry logic.

**Retry Pattern:**
- Max attempts: 5
- Initial delay: 5s
- Backoff multiplier: 1.5x
- Max delay: 30s

**Timeout:** 120s (includes retries)

### 5. smoke-tests
Runs automated smoke tests via npm test.

**Timeout:** 400s (includes npm install time)

### 6. notify-success
Sends Slack message with deployment details and action buttons.

**Timeout:** 30s

## Failure Handling

When any step fails, the `on_failure` block executes:

1. **notify-failure** — Slack alert with error details
2. **create-ticket** — Jira incident ticket for investigation
3. **log-failure** — Captures kubectl logs and pod status

All notifications include:
- Image tag that was deployed
- Failed step name
- Error message
- Links to logs and diagnostics

## Troubleshooting

### "Health check failed after 5 retries"

Service is not responding. Check:

```bash
kubectl -n staging logs -l app=myapp --tail=100
kubectl -n staging describe pods -l app=myapp
curl -v https://staging.example.com/health
```

### "Image push failed: unauthorized"

Registry credentials not configured. Update Docker credentials:

```bash
docker login docker.io
```

### "Kubectl apply failed: permission denied"

Check Kubernetes RBAC permissions:

```bash
kubectl auth can-i create deployments --namespace=staging
```

### "Smoke tests timeout"

Increase `TEST_TIMEOUT` in pipeline config or test execution time. Check test logs:

```bash
kubectl -n staging exec -it <pod> -- npm test -- --verbose
```

## Monitoring and Observability

### View Pipeline Logs

```bash
openclaw pipeline logs deploy-and-notify
```

### Check Deployment Status

```bash
kubectl -n staging rollout status deployment/myapp
kubectl -n staging get pods -l app=myapp -w
```

### Slack Notifications

All deployments are logged to `#deployments` with:
- Success/failure status
- Image tag and registry
- Health check results
- Links to logs and action items

## Advanced Patterns

### Deployment to Multiple Environments

Modify `STAGING_NAMESPACE` to deploy to prod:

```bash
openclaw pipeline run deploy-and-notify \
  --env IMAGE_TAG=v1.2.3 \
  --env STAGING_NAMESPACE=production
```

### Conditional Deployments

Add approval step before pushing to prod (advanced Lobster feature):

```yaml
- id: approval
  skill: slack
  action: request-reaction
  params:
    message: "Approve prod deployment?"
    requiredReaction: ":white_check_mark:"
```

### Integration with CI/CD

Trigger from GitHub Actions:

```yaml
- name: Deploy
  run: |
    openclaw pipeline run deploy-and-notify \
      --env IMAGE_TAG=${{ github.sha }} \
      --env SLACK_WEBHOOK=${{ secrets.SLACK_WEBHOOK }}
```

## Contributing

Found an issue or want to improve? See [CONTRIBUTING.md](../../CONTRIBUTING.md).

## License


This project is currently licensed under the Apache 2.0 License 。

