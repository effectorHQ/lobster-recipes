# Daily Standup Pipeline

Generate automated morning briefings for your engineering team by aggregating Jira issues, GitHub PR status, and Slack updates into a single daily digest.

## Overview

This pipeline creates a comprehensive morning briefing from multiple sources:

1. **Jira Sync** — Fetch issues assigned to users with priority and due dates
2. **GitHub PRs** — Aggregate open PRs, reviews needed, and recent activity
3. **Slack Digest** — Summarize mentions and important threads
4. **Email** — Send formatted HTML email to team inbox
5. **Slack Post** — Cross-post to engineering channel with action items

## Features Demonstrated

- **Data Aggregation** — Fetches from multiple external APIs in parallel
- **Template Rendering** — Formats data with Jinja2-style templates
- **Email Composition** — Builds HTML emails with embedded data
- **Scheduled Execution** — Runs daily at specified time via cron
- **Conditional Formatting** — Adapts output based on available data
- **Error Resilience** — Gracefully handles API failures with team notification

## Prerequisites

- Lobster CLI with scheduler support
- Jira API access (cloud or self-hosted)
- GitHub organization and personal access token
- Slack workspace with webhook or OAuth token
- Email service integration (SMTP or API)

## Configuration

### Environment Variables

```bash
# Jira
export JIRA_USER=john.smith              # Jira username or "current"
export JIRA_URL=https://company.atlassian.net

# GitHub
export GITHUB_ORG=mycompany              # GitHub organization
export GITHUB_USER=jsmith                # GitHub username
export GITHUB_TOKEN=ghp_xxxxx            # GitHub personal access token

# Slack
export SLACK_CHANNEL=#engineering        # Slack channel for posts
export SLACK_WEBHOOK=https://hooks.slack.com/services/...

# Email
export EMAIL_RECIPIENT=engineering@example.com
export SMTP_HOST=smtp.sendgrid.net
export SMTP_USER=apikey
export SMTP_PASSWORD=SG.xxxxx
```

### Credentials Storage

Store sensitive credentials in `.env` or secrets manager:

```bash
# .env (do not commit)
JIRA_API_TOKEN=xxxxxxxxxxx
GITHUB_TOKEN=ghp_xxxxxxxxxx
SLACK_WEBHOOK=https://hooks.slack.com/services/...
```

Load before running:

```bash
source .env
openclaw pipeline run daily-standup
```

### Customization

Edit `pipeline.yml` to modify:

| Setting | Default | Purpose |
|---------|---------|---------|
| `SLACK_CHANNEL` | `#engineering` | Where to post briefing |
| `JIRA_USER` | `current` | Whose issues to fetch |
| `EMAIL_RECIPIENT` | `engineering@example.com` | Email recipient |
| `GITHUB_ORG` | `myorg` | GitHub organization |
| Schedule | `0 8 * * *` (daily 8 AM UTC) | Run frequency |

## Running the Pipeline

### Manual Execution

```bash
openclaw pipeline run daily-standup
```

### With Custom Recipients

```bash
openclaw pipeline run daily-standup \
  --env EMAIL_RECIPIENT=my-team@example.com \
  --env SLACK_CHANNEL=#my-team
```

### Dry Run (Preview)

```bash
openclaw pipeline run daily-standup --dry-run
```

### Enable Scheduled Execution

Register for daily execution:

```bash
openclaw scheduler create daily-standup \
  --cron "0 8 * * *" \
  --timezone America/New_York
```

View scheduled tasks:

```bash
openclaw scheduler list
```

### Disable or Update Schedule

```bash
openclaw scheduler update daily-standup \
  --cron "0 9 * * MON-FRI"  # Weekdays only at 9 AM
```

## Step Details

### fetch-jira-issues
Retrieves issues assigned to user with recent activity.

**JQL Filter:**
- Status: In Progress, In Review, To Do
- Updated: Within last 24 hours
- Max results: 20

**Fields Returned:** key, summary, status, priority, updated, dueDate

### fetch-github-prs
Finds open PRs assigned to user or awaiting their review.

**Query:**
- Open PRs assigned to you
- Your open PRs awaiting review
- PRs where review is requested
- Sort: Most recently updated

### fetch-slack-threads
Retrieves unread messages and direct mentions from last 24 hours.

**Includes:**
- Direct mentions (@username)
- Important threads
- Reaction highlights

### summarize-jira
Formats Jira issues into readable summary with priorities and due dates.

### summarize-github
Organizes PRs by type: assigned to you, your authored PRs, reviews needed.

### summarize-slack
Highlights mentions and important conversation threads.

### compose-briefing
Creates HTML email combining all summaries with styling and links.

### send-email
Delivers email to configured recipients.

### post-slack
Posts rich-formatted message to Slack channel with block kit.

## Email Template

The pipeline generates HTML emails with:

- Header with date
- Jira issues section with priority colors
- GitHub PR summary with statuses
- Slack mentions and highlights
- Quick links to tools
- Professional styling

Customize template in `compose-briefing` step of `pipeline.yml`.

## Slack Post Format

The Slack message includes:

- **Header** — Date and time
- **Jira Section** — Issues with status and priority
- **GitHub Section** — PRs needing attention
- **Slack Section** — Key mentions and threads
- **Thread Prompt** — Encourage team responses

## Monitoring and Debugging

### Check Scheduled Runs

```bash
openclaw scheduler logs daily-standup --lines 50
```

### View Generated Briefings

Retrieve recent email subjects:

```bash
openclaw pipeline runs daily-standup --limit 10
```

### Debug API Calls

Run with verbose logging:

```bash
openclaw pipeline run daily-standup --verbose
```

Check logs for:
- Jira API response times
- GitHub rate limit status
- Slack message delivery status

### Test Email Delivery

Send test briefing to specific address:

```bash
openclaw pipeline run daily-standup \
  --env EMAIL_RECIPIENT=test@example.com \
  --dry-run
```

## Customization Examples

### Include Additional Jira Fields

Modify `fetch-jira-issues` fields list:

```yaml
fields:
  - key
  - summary
  - status
  - priority
  - updated
  - dueDate
  - assignee      # Add assignee
  - reporter      # Add reporter
  - labels        # Add labels
```

### Filter to Specific GitHub Repos

Update `fetch-github-prs` query:

```yaml
query: |
  is:pr is:open
  repo:myorg/critical-app
  repo:myorg/api-service
  assignee:${GITHUB_USER}
```

### Add Sprint Information

Extend `summarize-jira` template:

```jinja
- **[{{ issue.key }}](...)**: {{ issue.summary }}
  - Sprint: {{ issue.sprint }}
  - Story Points: {{ issue.storyPoints }}
```

### Schedule for Different Timezones

Adjust the cron expression and timezone:

```yaml
scheduler:
  cron: "0 8 * * *"
  timezone: "America/Los_Angeles"  # or Europe/London, Asia/Tokyo, etc.
```

### Weekdays Only

```yaml
scheduler:
  cron: "0 8 * * 1-5"  # Monday to Friday
```

## Troubleshooting

### "Jira API authentication failed"

Check JIRA credentials:

```bash
curl -u username:token https://jira.example.com/rest/api/3/myself
```

Update JIRA_API_TOKEN and retry.

### "GitHub rate limit exceeded"

GitHub allows 5,000 requests per hour. If you hit the limit:
- Add delay between pipeline runs
- Use GitHub App authentication (higher limits)
- Cache results for longer period

### "Slack message too long"

If briefing exceeds Slack's 4,000 character limit:
- Reduce `maxLength` in summarize steps
- Filter to fewer repositories in GitHub
- Limit Jira issue count

### "Email not received"

Check SMTP credentials and delivery:

```bash
echo "Test" | sendmail -S ${SMTP_HOST} -au ${SMTP_USER} -ap ${SMTP_PASSWORD} ${EMAIL_RECIPIENT}
```

## Advanced Patterns

### Conditional Briefings

Skip briefing on weekends:

```yaml
- id: check-day
  skill: lobster
  action: eval
  params:
    expr: "now().weekday() < 5"  # Monday-Friday

- id: send-briefing
  dependsOn:
    - check-day
  condition: "${check-day.result}"
```

### Team-Based Briefings

Generate separate briefings for teams:

```bash
openclaw pipeline run daily-standup \
  --env JIRA_USER=alice --env EMAIL_RECIPIENT=frontend-team@example.com &
openclaw pipeline run daily-standup \
  --env JIRA_USER=bob --env EMAIL_RECIPIENT=backend-team@example.com &
wait
```

### Slack Thread Responses

Enable team to reply in Slack thread and post back to Jira.

## Contributing

Found an issue or want improvements? See [CONTRIBUTING.md](../../CONTRIBUTING.md).

## License


This project is currently licensed under the Apache 2.0 License 。

