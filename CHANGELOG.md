# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-05

### Added
- Initial release of Lobster recipes repository
- **deploy-and-notify** pipeline: Automated Docker image building, pushing, Kubernetes deployment, health checks, and team notifications with retry logic and failure handlers
- **daily-standup** pipeline: Morning briefing automation pulling Jira issues, GitHub PR status, Slack thread summaries, and generating email digests
- **pr-review-triage** pipeline: Smart PR review workflow with skill linting, test execution, risk classification, reviewer auto-assignment, and summary comments
- **incident-response** pipeline: Incident handling automation with service health checks, log aggregation, deployment history, Jira ticket creation, and on-call notifications
- **content-publish** pipeline: Content publishing workflow from Notion drafts through spell checking, social media snippet generation, tweet scheduling, and changelog updates
- Comprehensive documentation and README in English and Chinese
- MIT License
- Contributing guidelines template

### Features Demonstrated
- **Retry Logic** — Exponential backoff patterns for transient failures
- **Error Handling** — `on_failure` hooks for recovery and notifications
- **Variable Interpolation** — Dynamic parameterization using `${VARIABLE}` syntax
- **Conditional Steps** — Branching logic based on previous step outcomes
- **Sequential Execution** — Enforced step ordering and dependencies
- **Skill Composition** — Integration with Docker, Kubernetes, Jira, Slack, GitHub, and custom APIs
- **Token Optimization** — Caching and step reuse patterns throughout recipes

---

## Release Guidelines

For maintainers:

- Update version in documentation references
- Add release notes to this changelog
- Tag releases in git: `git tag v1.0.0`
- Update GitHub releases page with changelog entries
- Announce in OpenClaw community channels
