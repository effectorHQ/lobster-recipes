# Lobster Recipes

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Lobster](https://img.shields.io/badge/powered%20by-Lobster-blue.svg)](https://github.com/effectorHQ/lobster)
[![effectorHQ](https://img.shields.io/badge/by-effectorHQ-black.svg)](https://github.com/effectorHQ)

**[中文文档 →](./README.zh.md)**

Real-world workflow orchestration pipelines built with Lobster, OpenClaw's deterministic, resumable, and token-efficient workflow engine.

## What is Lobster?

Lobster is a workflow orchestration engine that chains skills (modular, composable AI capabilities) into deterministic, resumable pipelines. Unlike traditional orchestration tools, Lobster:

- **Chains skills deterministically** — Each step is reproducible and composable
- **Resumes gracefully** — Failed pipelines pick up where they left off without restarting
- **Optimizes tokens** — Minimizes LLM calls through intelligent caching and step reuse
- **Integrates seamlessly** — Works with Slack, GitHub, Kubernetes, Docker, Jira, and custom APIs
- **Handles errors intelligently** — Retry logic, fallbacks, and conditional execution paths

Learn more at [OpenClaw Documentation](https://docs.openclaw.dev/lobster).

## Getting Started

### Installation

Install Lobster via the [OpenClaw CLI](https://docs.openclaw.dev/cli):

```bash
pip install openclaw-cli
openclaw init
```

### Using These Recipes

1. Clone or fork this repository:
   ```bash
   git clone https://github.com/effectorHQ/lobster-recipes.git
   cd lobster-recipes
   ```

2. Browse the [pipelines](#available-pipelines) directory and choose a recipe

3. Adapt the pipeline to your needs (update skill parameters, secrets, etc.)

4. Deploy and run:
   ```bash
   openclaw pipeline deploy pipelines/your-pipeline/pipeline.yml
   openclaw pipeline run your-pipeline
   ```

## Available Pipelines

| Pipeline | Purpose | Features |
|----------|---------|----------|
| [**deploy-and-notify**](./pipelines/deploy-and-notify) | Automated deployment with notifications | Docker build/push, K8s apply, health checks, retry logic, failure notifications |
| [**daily-standup**](./pipelines/daily-standup) | Morning team briefing | Jira issue tracking, GitHub PR status, Slack thread summarization, email digest |
| [**pr-review-triage**](./pipelines/pr-review-triage) | Smart PR review workflow | Skill linting, test execution, risk classification, auto-assignment, comment generation |
| [**incident-response**](./pipelines/incident-response) | Incident handling automation | Health checks, log aggregation, deployment history, Jira ticketing, on-call alerting |
| [**content-publish**](./pipelines/content-publish) | Content publishing workflow | Notion drafts, spell checking, social media snippets, tweet scheduling, changelog updates |

## Pipeline Features Demonstrated

Each recipe showcases different Lobster capabilities:

- **Retry Logic** — `deploy-and-notify` demonstrates exponential backoff for health checks
- **Error Handling** — `deploy-and-notify` includes `on_failure` hooks for rollback notifications
- **Variable Interpolation** — All pipelines use `${VARIABLE}` syntax for dynamic values
- **Conditional Steps** — `pr-review-triage` and `incident-response` show branching logic
- **Sequential Execution** — All pipelines enforce step ordering and dependencies
- **Skill Composition** — Multi-step workflows combining diverse integrations (K8s, Docker, Slack, etc.)

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### How to Contribute a New Recipe

1. Create a folder under `pipelines/your-recipe/`
2. Add `pipeline.yml` with your workflow
3. Add `README.md` explaining the pipeline's purpose and usage
4. Update this main README with your pipeline in the table
5. Submit a pull request

## Documentation

- [Lobster Documentation](https://docs.openclaw.dev/lobster) — Full language reference
- [Skills Library](https://docs.openclaw.dev/skills) — Available integrations
- [OpenClaw CLI Reference](https://docs.openclaw.dev/cli) — Command reference

## License

MIT © 2026 effectorHQ Contributors. See [LICENSE](LICENSE) for details.

## Community

- **Issues & Discussions**: [GitHub Issues](https://github.com/effectorHQ/lobster-recipes/issues)
- **Slack Community**: [Join OpenClaw Slack](https://openclaw.dev/slack)
- **Twitter**: [@effectorHQ](https://twitter.com/effectorHQ)

---

**Made with ❤️ by the effectorHQ team**
