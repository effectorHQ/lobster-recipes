# Content Publish Pipeline

Streamline your content publishing workflow from Notion drafts to published articles across social media, with automatic spell checking, scheduling, and team notifications.

## Overview

This pipeline automates end-to-end content publishing:

1. **Fetch Draft** — Retrieve content from Notion workspace
2. **Validate** — Spell check and grammar validation
3. **Generate Snippets** — Create social media variants (Twitter, LinkedIn)
4. **Schedule** — Automatically schedule tweets and post to LinkedIn
5. **Track** — Set up analytics and metrics tracking
6. **Document** — Update CHANGELOG with publication reference
7. **Notify** — Alert team on Slack with links and metrics
8. **Archive** — Mark Notion draft as published

## Features Demonstrated

- **Document Processing** — Parse Notion pages and extract metadata
- **Content Validation** — Spell check and grammar verification
- **Conditional Logic** — Request manual review if errors found
- **Content Generation** — AI-generated social media snippets
- **Multi-Channel Publishing** — Twitter, LinkedIn, Slack
- **Scheduling** — Tweet scheduling at optimal times
- **File Management** — Update CHANGELOG with publication records
- **Analytics Setup** — Auto-configured tracking campaigns

## Prerequisites

- Lobster CLI configured
- Notion workspace with content drafts
- Twitter API access with account token
- LinkedIn API access
- Slack workspace webhook
- Email service configured
- Content validation tools (spell check, grammar)

## Configuration

### Environment Variables

```bash
# Content source
export CONTENT_URL=https://notion.so/draft-page-id  # Notion page
export CONTENT_TITLE="My Article Title"

# Publishing accounts
export TWITTER_ACCOUNT=effectorhq              # Twitter handle
export SLACK_CHANNEL=#content                  # Slack notification channel
export EMAIL_RECIPIENT=content-team@example.com

# Optional
export PUBLISH_DATE=$(date +%Y-%m-%d)         # Override publication date
```

### API Credentials

```bash
# Notion
export NOTION_API_KEY=notiontokenxxxxx

# Twitter
export TWITTER_API_KEY=xxxxx
export TWITTER_API_SECRET=xxxxx
export TWITTER_ACCESS_TOKEN=xxxxx
export TWITTER_ACCESS_SECRET=xxxxx

# LinkedIn
export LINKEDIN_ACCESS_TOKEN=xxxxx

# Email/SMTP
export SMTP_HOST=smtp.sendgrid.net
export SMTP_USER=apikey
export SMTP_PASSWORD=SG.xxxxx
```

### Notion Setup

1. Create content database in Notion with fields:
   - Title
   - Description
   - Category
   - Author
   - Tags
   - Status (Draft/Published)
   - PublishedDate

2. Share page with Notion API token

3. Get page URL or ID for `CONTENT_URL`

## Running the Pipeline

### Manual Publishing

```bash
openclaw pipeline run content-publish \
  --env CONTENT_URL=https://notion.so/xxxxx \
  --env CONTENT_TITLE="My Article"
```

### Batch Publishing

Publish multiple articles:

```bash
# Create batch file
cat > content-batch.json << EOF
[
  {
    "CONTENT_URL": "https://notion.so/article-1",
    "CONTENT_TITLE": "First Article"
  },
  {
    "CONTENT_URL": "https://notion.so/article-2",
    "CONTENT_TITLE": "Second Article"
  }
]
EOF

# Run batch
for item in $(jq -r '.[] | @base64' content-batch.json); do
  export CONTENT=$(echo $item | base64 -d)
  openclaw pipeline run content-publish --env CONTENT_URL ... --env CONTENT_TITLE ...
done
```

### Scheduled Publishing

Schedule daily content publishing:

```bash
openclaw scheduler create daily-publish \
  --cron "0 9 * * *" \
  --pipeline content-publish \
  --env-file .env.publishing
```

### Dry Run

Preview publishing without sending:

```bash
openclaw pipeline run content-publish \
  --env CONTENT_URL=https://notion.so/xxxxx \
  --dry-run
```

## Step Details

### fetch-notion-draft
Retrieves page from Notion workspace with all children blocks.

**Returns:**
- Page content
- Metadata (title, description, tags, author)
- All child blocks (text, images, code)

### extract-content
Parses Notion content and converts to structured format.

**Transformations:**
- Extract YAML-style frontmatter
- Convert to Markdown
- Normalize formatting (whitespace, lists, etc.)

### spell-check
Validates spelling, grammar, and readability.

**Checks:**
- Spelling errors
- Grammar issues
- Readability metrics
- Style guide compliance (AP style)

**Returns suggestions** if errors found

### check-validation-errors
Evaluates if errors require manual review.

Condition: `errorCount > 0`

### request-review
Sends Slack message requesting team approval if errors found.

Waits for ✅ reaction to proceed or ❌ to abort.

### apply-corrections
Auto-corrects minor spelling/grammar issues if review approved.

Only applies obvious corrections (not contentious changes).

### generate-snippets
Creates social media variants using AI:

**Twitter:**
- 280 character limit
- Includes hashtags from content tags
- Includes link to article
- 3 variants for A/B testing

**LinkedIn:**
- 3,000 character limit
- Professional tone
- Call-to-action
- 2 variants

**Slack:**
- 500 character limit
- Includes emoji
- Shareable format

### schedule-tweets
Schedules tweets to Twitter account:

**Timing:**
- Tweet 1: Immediately (or +2h)
- Tweet 2: +24 hours
- Tweet 3: +48 hours

Spreads engagement and reaches different timezones.

### post-linkedin
Creates LinkedIn article or post:

**Parameters:**
- Title, description, content
- Visibility: PUBLIC
- Tags from article metadata

### generate-changelog-entry
Creates formatted changelog entry:

Includes:
- Title and author
- Publication date
- Category
- Links to all published versions
- Summary
- Tags

### update-changelog
Prepends entry to CHANGELOG.md file.

Maintains publishing history in repo.

### post-slack-notification
Posts rich Slack notification to team:

**Includes:**
- Article title and metadata
- Publication date and author
- Social media platform counts
- Action buttons (view on Notion, LinkedIn, etc.)
- Sharing options

### setup-metrics-tracking
Creates analytics campaign for tracking:

**Tracks:**
- Twitter engagement
- LinkedIn impressions
- Click-through rates
- Attribution

Creates UTM parameters:
- utm_source: social
- utm_campaign: article-slug
- utm_content: publication-date

### generate-report
Creates comprehensive publishing report:

**Contains:**
- Summary of all actions taken
- Social media metrics
- Links to published content
- Tracking campaign details
- Next steps for monitoring

### email-report
Sends report to content team:

**Recipients:**
- Content team primary
- Marketing cc'd

**Attachments:**
- Social media snippets JSON

### archive-notion-draft
Updates Notion page properties:

**Updates:**
- Status: Published
- PublishedDate
- TwitterUrl
- LinkedInUrl

Marks draft as complete.

## Social Media Strategy

### Twitter Strategy

Spreads messaging across 3 tweets at different times:

1. **Announcement** (now) — Introduces topic with teaser
2. **Deep Dive** (+24h) — Key insights or statistics
3. **Call-to-Action** (+48h) — Link to full article with engagement prompt

### LinkedIn Strategy

Professional, longer-form article post:

- Positions content as thought leadership
- Includes company/personal branding
- Drives traffic to full article
- Encourages professional engagement

## Monitoring Publishing

### Check Publishing History

```bash
openclaw pipeline runs content-publish --limit 50
```

### View Published Analytics

Check metrics dashboard:
- Twitter engagement
- LinkedIn impressions
- Click-through rates
- Traffic attribution

### Update Content Metadata

If changes needed, edit Notion page and re-publish:

```bash
openclaw pipeline run content-publish \
  --env CONTENT_URL=https://notion.so/xxxxx \
  --resume-from extract-content
```

## Customization Examples

### Add Instagram Publishing

Insert new step after `generate-snippets`:

```yaml
- id: post-instagram
  skill: instagram
  action: create-post
  params:
    caption: "${generate-snippets.instagram}"
    imageUrl: "${extract-content.featuredImage}"
```

### Custom Review Process

Replace Slack reaction approval with email:

```yaml
- id: request-approval-email
  skill: email
  action: send-approval
  params:
    to: "${REVIEWER_EMAIL}"
    content: "${spell-check.suggestions}"
```

### Adjust Tweet Scheduling

Modify timing for different timezones:

```yaml
scheduleTime: |
  [
    "now() + 1h",     # Early morning US
    "now() + 14h",    # Evening US / Morning EU
    "now() + 26h"     # Asia timezone
  ]
```

### Add Google Analytics Tracking

Extend `setup-metrics-tracking`:

```yaml
- id: setup-google-analytics
  skill: google-analytics
  action: create-goal
  params:
    goalName: "Article Clicks"
    goalType: "destination"
    goalUrl: "/article/${content-slug}"
```

### Pre-Check Against Content Policy

Add validation before publishing:

```yaml
- id: compliance-check
  skill: ai-content
  action: evaluate
  params:
    text: "${extract-content.markdown}"
    policies:
      - brand_voice
      - legal_compliance
      - diversity_inclusion
```

## Troubleshooting

### "Notion authentication failed"

Check API key and page access:

```bash
curl -H "Authorization: Bearer ${NOTION_API_KEY}" \
  https://api.notion.com/v1/pages/${CONTENT_ID}
```

### "Grammar check exceeded timeout"

Content too long. Split into sections or increase timeout in pipeline config.

### "Twitter rate limit exceeded"

Twitter allows 300 tweets per 3 hours. Spread scheduling or use different account.

### "LinkedIn post not visible"

Check visibility setting:

```yaml
visibility: "PUBLIC"  # Not "CONNECTIONS_ONLY" or "PRIVATE"
```

### "CHANGELOG.md corrupted after update"

Keep backup. Manually verify or restore:

```bash
git diff CHANGELOG.md
git checkout HEAD -- CHANGELOG.md
```

## Performance Considerations

- **Parallel Steps** — Validation and snippet generation run in parallel
- **Caching** — Content fetches cached for 1 hour
- **Timeouts** — Overall pipeline limited to 10 minutes
- **Resumable** — Can resume from specific step if failure occurs

## Best Practices

1. **Content Quality** — Fix spelling/grammar issues before publishing
2. **Scheduling** — Space tweets across timezones for better reach
3. **Hashtags** — Use relevant tags for discoverability
4. **Analytics** — Monitor performance and adjust strategy
5. **Team Input** — Get approval before publishing to official accounts
6. **Archives** — Keep CHANGELOG updated for audit trail

## Contributing

Found issues or have improvements? See [CONTRIBUTING.md](../../CONTRIBUTING.md).

## License

MIT © 2026 effectorHQ Contributors
