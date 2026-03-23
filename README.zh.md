# Lobster 食谱库

[![欢迎 PRs](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

[![Lobster](https://img.shields.io/badge/powered%20by-Lobster-blue.svg)](https://github.com/effectorHQ/lobster)
[![effectorHQ](https://img.shields.io/badge/by-effectorHQ-black.svg)](https://github.com/effectorHQ)

**[English Documentation →](./README.md)**

使用 Lobster 构建的真实工作流编排管道集合。Lobster 是 OpenClaw 的确定性、可恢复的、令牌高效的工作流引擎。

## 什么是 Lobster？

Lobster 是一个工作流编排引擎，将技能（模块化、可组合的 AI 能力）链接成确定性的、可恢复的管道。与传统编排工具不同，Lobster：

- **确定性链接技能** — 每一步都是可重现和可组合的
- **优雅地恢复** — 失败的管道从中断处继续，无需重启
- **优化 Token** — 通过智能缓存和步骤重用最小化 LLM 调用
- **无缝集成** — 支持 Slack、GitHub、Kubernetes、Docker、Jira 和自定义 API
- **智能错误处理** — 重试逻辑、回退和条件执行路径

了解更多请访问 [OpenClaw 文档](https://docs.openclaw.dev/lobster)。

## 快速开始

### 安装

通过 [OpenClaw CLI](https://docs.openclaw.dev/cli) 安装 Lobster：

```bash
pip install openclaw-cli
openclaw init
```

### 使用这些食谱

1. 克隆或 Fork 此仓库：
   ```bash
   git clone https://github.com/effectorHQ/lobster-recipes.git
   cd lobster-recipes
   ```

2. 浏览 [pipelines](#可用管道) 目录并选择一个食谱

3. 根据需要调整管道（更新技能参数、密钥等）

4. 部署并运行：
   ```bash
   openclaw pipeline deploy pipelines/your-pipeline/pipeline.yml
   openclaw pipeline run your-pipeline
   ```

## 可用管道

| 管道 | 用途 | 特性 |
|------|------|------|
| [**deploy-and-notify**](./pipelines/deploy-and-notify) | 自动化部署与通知 | Docker 构建/推送、K8s 应用、健康检查、重试逻辑、失败通知 |
| [**daily-standup**](./pipelines/daily-standup) | 晨会团队简报 | Jira 问题跟踪、GitHub PR 状态、Slack 线程总结、邮件摘要 |
| [**pr-review-triage**](./pipelines/pr-review-triage) | 智能 PR 评审工作流 | 技能 Linting、测试执行、风险分类、自动分配、评论生成 |
| [**incident-response**](./pipelines/incident-response) | 事件处理自动化 | 健康检查、日志聚合、部署历史、Jira 工单、值班人员告警 |
| [**content-publish**](./pipelines/content-publish) | 内容发布工作流 | Notion 草稿、拼写检查、社交媒体片段、推文调度、更新变更日志 |

## 管道特性演示

每个食谱都展示了 Lobster 的不同能力：

- **重试逻辑** — `deploy-and-notify` 展示了健康检查的指数退避
- **错误处理** — `deploy-and-notify` 包含用于回滚通知的 `on_failure` 钩子
- **变量插值** — 所有管道都使用 `${VARIABLE}` 语法表示动态值
- **条件步骤** — `pr-review-triage` 和 `incident-response` 展示分支逻辑
- **顺序执行** — 所有管道强制步骤排序和依赖性
- **技能组合** — 多步工作流结合多个集成（K8s、Docker、Slack 等）

## 贡献

我们欢迎贡献！请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 了解指南。

### 如何贡献新食谱

1. 在 `pipelines/your-recipe/` 下创建一个文件夹
2. 添加 `pipeline.yml` 包含您的工作流
3. 添加 `README.md` 解释管道的目的和用法
4. 在此主 README 中的表格中更新您的管道
5. 提交拉取请求

## 文档

- [Lobster 文档](https://docs.openclaw.dev/lobster) — 完整的语言参考
- [技能库](https://docs.openclaw.dev/skills) — 可用的集成
- [OpenClaw CLI 参考](https://docs.openclaw.dev/cli) — 命令参考

## 许可证

This project is currently licensed under the [Apache License, Version 2.0](LICENSE.md) 。

## 社区

- **问题与讨论**: [GitHub Issues](https://github.com/effectorHQ/lobster-recipes/issues)
- **Slack 社区**: [加入 OpenClaw Slack](https://openclaw.dev/slack)
- **Twitter**: [@effectorHQ](https://twitter.com/effectorHQ)

---

**由 effectorHQ 团队用❤️制作**
