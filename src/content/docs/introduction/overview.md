---
title: Overview
description: What the Xianix Agent is and how it works end-to-end.
---

The Xianix Agent is a deployable, extensible agent built for the [Xians ACP](https://xians.ai) (Agent Control Plane). It lets you assemble purpose-built AI workers — powered by Claude Code plugins — to automate key stages of your software delivery lifecycle, from PR review and requirement analysis to any custom enterprise workflow you define.

Under the hood, Xianix is a long-running .NET console application that bridges your code platform (GitHub, Azure DevOps) with AI-powered automation. It listens for webhook events, evaluates a declarative rules engine, and orchestrates isolated Docker containers that run Claude Code plugins against your repositories.

## Process flow

```
GitHub / Azure DevOps
        │ webhook
        ▼
  Xians Webhooks
  (app.xians.ai)
        │ dispatches task
        ▼
    TheAgent
  (.NET Console App)
        │ spawns & mounts socket
        ▼
     Executor
  (Docker container)
        │ runs
        ▼
 ClaudeCode Plugins
 (review, triage…)
```

| Layer | Role |
|---|---|
| **GitHub / Azure DevOps** | Source of events — a PR is opened, a comment is posted, a work item changes. |
| **Xians Webhooks** | Receives platform events and routes them to the registered agent instance. |
| **TheAgent** | Long-running .NET process that polls Xians, interprets incoming tasks, and orchestrates execution. |
| **Executor** | Ephemeral Docker container spawned per task — fully isolated with no persistent state. |
| **ClaudeCode Plugins** | Skills (e.g. PR review, requirement analysis) that run inside the Executor and interact with the LLM. |

## How an event flows

1. A webhook arrives at Xians from GitHub or Azure DevOps (e.g. a pull request is opened).
2. The agent's `WebhookWorkflow` receives the event and passes it to the `EventOrchestrator`.
3. The `RulesEvaluator` matches the event against `rules.json`, extracts inputs, and identifies which plugin to run.
4. A `ProcessingWorkflow` is signalled — it creates an isolated workspace, clones the repository, installs the plugin, and invokes it.
5. The plugin executes inside an ephemeral Docker container and posts its results back to the platform.

## Repositories

| Repository | Description |
|---|---|
| `the-agent` | .NET agent deployed on Xians — includes workflows, orchestrator, and rules engine |
| `plugins-official` | Official Claude Code plugins maintained by the 99x / Xianix team |
