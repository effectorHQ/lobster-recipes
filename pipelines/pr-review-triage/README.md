# PR Review Triage Pipeline

Automate pull request validation, classification, and reviewer assignment with intelligent code analysis and comprehensive summary comments.

## Overview

This pipeline streamlines PR workflows by automating:

1. **Skill Validation** — Lint SKILL.md if skill definitions changed
2. **Test Execution** — Run unit tests, integration tests, and linters
3. **Code Analysis** — Extract metrics on complexity, coverage, duplication
4. **Risk Classification** — Categorize by size (small/medium/large) and risk (low/medium/high)
5. **Reviewer Routing** — Assign experts based on code changes and workload
6. **Summary Comments** — Post comprehensive analysis with recommendations
7. **Labeling** — Auto-tag PRs for visibility and filtering

## Features Demonstrated

- **Conditional Steps** — Skill validation only runs if SKILL.md changed
- **Code Metrics** — Complexity, coverage, duplication analysis
- **Risk Scoring** — Multi-factor risk assessment
- **Skill Integration** — Validates OpenClaw skill definitions
- **GitHub Integration** — Comments, labels, reviewer assignment
- **Rich Formatting** — Markdown tables, status indicators, recommendations

## Prerequisites

- Lobster CLI configured
- GitHub repository with webhook support
- npm/Node.js for test execution
- Code quality tools (eslint, jest, coverage reporters)
- CODEOWNERS file for reviewer routing (optional)

## Configuration

### GitHub Setup

1. **Webhook Configuration**

Add webhook to your GitHub repo:

- **Payload URL:** `https://your-lobster-instance/webhooks/github`
- **Content type:** `application/json`
- **Events:** Pull requests, Push

2. **Token Setup**

Create GitHub personal access token with permissions:

```bash
export GITHUB_TOKEN=ghp_xxxxx  # With repo, read:org scopes
```

3. **Environment Variables**

```bash
export GITHUB_REPO_OWNER=myorg
export GITHUB_REPO_NAME=myapp
export GITHUB_PR_NUMBER=<auto-filled by webhook>
export GITHUB_PR_AUTHOR=<auto-filled by webhook>
export GITHUB_BRANCH=<auto-filled by webhook>
```

### Customization

Edit `pipeline.yml` to adjust:

| Setting | Default | Purpose |
|---------|---------|---------|
| Small PR threshold | 200 LOC | Below this = small |
| Medium PR threshold | 500 LOC | 200-500 = medium |
| Risk factors | 5 total | High risk = ≥3 factors |
| Coverage threshold | 80% | Acceptable coverage |
| Max complexity | 10 | Cyclomatic complexity limit |

## Running the Pipeline

### Manual Execution

```bash
openclaw pipeline run pr-review-triage \
  --env GITHUB_PR_NUMBER=123 \
  --env GITHUB_REPO_OWNER=myorg \
  --env GITHUB_REPO_NAME=myapp
```

### GitHub Actions Integration

Add to `.github/workflows/pr-triage.yml`:

```yaml
name: PR Triage

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run PR Triage
        run: |
          openclaw pipeline run pr-review-triage \
            --env GITHUB_PR_NUMBER=${{ github.event.number }} \
            --env GITHUB_REPO_OWNER=${{ github.repository_owner }} \
            --env GITHUB_REPO_NAME=${{ github.event.repository.name }} \
            --env GITHUB_PR_AUTHOR=${{ github.actor }} \
            --env GITHUB_BRANCH=${{ github.head_ref }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Webhook-Triggered Execution

Configure GitHub webhook to auto-trigger on PR events. The pipeline receives context automatically:

- PR number, author, branch
- Changed files
- Diff information

## Step Details

### check-skill-changes
Detects if SKILL.md or skill configuration files were modified.

**Outputs:**
- `hasMatches` — Boolean indicating changes detected
- Files matching pattern

### validate-skill
Validates skill definition if SKILL.md changed.

**Only runs if:** `check-skill-changes.hasMatches == true`

**Checks:** Metadata, parameters, examples, documentation

### run-tests
Executes full test suite with coverage reporting.

**Commands:**
- Unit tests with coverage JSON output
- Integration tests (optional)
- Linter with JSON report

**Coverage metrics:** Extracted to coverage.json

### analyze-metrics
Calculates code quality metrics:

- Cyclomatic complexity
- Maintainability index
- Lines of code changed
- Test coverage percentage
- Code duplication ratio

### classify-pr
Determines PR size and risk level.

**Size Logic:**
- Small: < 200 LOC
- Medium: 200-500 LOC
- Large: > 500 LOC

**Risk Scoring (0-5 factors):**
- Cyclomatic complexity > 10
- Test coverage < 80%
- Skill definition changes
- Test failures
- Code duplication > 10%

**Risk Levels:**
- Low: 0 factors
- Medium: 1-2 factors
- High: 3+ factors

### assign-reviewers
Routes PR to appropriate reviewers based on:

- Changed files (via CODEOWNERS)
- Reviewer expertise
- Current workload
- Max 5 PRs per reviewer

### generate-summary
Creates detailed comment with:

- Size/risk classification with indicators
- Quality metrics table
- Test results
- Skill changes (if applicable)
- Recommendations
- Suggested reviewers

### post-comment
Posts summary to PR as comment.

### assign-pr-reviewers
Requests code review from assigned reviewers and teams.

### apply-labels
Tags PR with:

- `size/{small|medium|large}`
- `risk/{low|medium|high}`
- `skill-changed` (if SKILL.md modified)
- `low-coverage` (if < 80%)
- `needs-architecture-review` (if large + high risk)

## Example Output

PR summary comment looks like:

```
## 🤖 Automated PR Review

### Classification
**Size:** 🟡 MEDIUM
**Risk:** 🟡 MEDIUM

### Quality Metrics
| Metric | Value | Status |
|--------|-------|--------|
| Lines Changed | 325 | ✅ |
| Test Coverage | 82% | ✅ |
| Complexity | 8 | ✅ |
| Duplication | 5% | ✅ |

### Test Results
- ✅ Unit Tests PASSED
- ✅ Integration Tests PASSED
- ✅ Lint Checks PASSED

### Recommendations
- 📊 Improve Test Coverage — Current coverage 82%, target 80%+

### Suggested Reviewers
- @alice (3 files)
- @bob (2 files)
```

## CODEOWNERS Integration

Use `CODEOWNERS` file for better reviewer routing:

```
# backend
src/api/**         @alice @backend-team
src/database/**    @bob @data-team

# frontend
src/components/**  @charlie @frontend-team

# skills
SKILL.md           @dave @skill-experts
```

## Monitoring and Debugging

### Check Pipeline Runs

```bash
openclaw pipeline runs pr-review-triage --limit 20
```

### View PR Comment History

Each PR accumulates comments. Lobster updates the latest comment instead of spamming.

### Debug Code Analysis

Run individual steps:

```bash
openclaw pipeline run pr-review-triage \
  --only analyze-metrics \
  --env GITHUB_PR_NUMBER=123
```

### Check Test Coverage Reports

Generated during `run-tests`:

```bash
cat coverage.json | jq '.coverage'
```

## Customization Examples

### Add Custom Quality Gates

Modify `classify-pr` logic:

```javascript
let requiresSecurityReview = skillChanges && risk !== "low";
let requiresDatabaseReview = changedFiles.includes("src/database");
let requiresLoadTesting = size === "large" && modifiesPerformance;

return {
  size: size,
  risk: risk,
  requiresSecurityReview,
  requiresDatabaseReview,
  requiresLoadTesting
};
```

### Adjust Risk Thresholds

Modify scoring factors in `classify-pr`:

```yaml
riskFactors = 0;
if (metrics.cyclomatic_complexity > 15) riskFactors++;  // More lenient
if (metrics.test_coverage < 0.75) riskFactors++;        // Lower target
```

### Add Performance Testing

Insert before `generate-summary`:

```yaml
- id: performance-test
  skill: shell
  action: exec
  params:
    command: npm run test:performance
  condition: "${classify-pr.size == 'large'}"
```

### Slack Notification on High Risk

Add to `on_failure` or create separate notification step:

```yaml
- id: notify-slack
  skill: slack
  action: send
  condition: "${classify-pr.risk == 'high'}"
  params:
    channel: "#code-review"
    message: "High-risk PR #${PR_NUMBER} by @${PR_AUTHOR}"
```

## Troubleshooting

### "Skill validation failed"

Check SKILL.md syntax:

```bash
openclaw skill validate SKILL.md
```

### "Test coverage below threshold"

Increase test coverage in your code:

```bash
npm run test:coverage
open coverage/index.html
```

### "Too many complex functions"

Refactor high-complexity code:

```bash
npm run analyze:complexity
```

### "Reviewers not assigned"

Ensure CODEOWNERS file exists and is formatted correctly:

```bash
cat .github/CODEOWNERS
```

## Performance Considerations

- **Parallel Execution** — Tests, analysis, and metrics run in parallel
- **Caching** — Results cached for 1 hour to avoid redundant analysis
- **Timeout** — Overall pipeline limited to 15 minutes
- **Resumable** — Failed runs can resume from last step

## Contributing

Found issues or have improvements? See [CONTRIBUTING.md](../../CONTRIBUTING.md).

## License

MIT © 2026 OpenClawHQ Contributors
