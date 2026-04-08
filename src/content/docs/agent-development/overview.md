---
title: Overview
description: Introduction to developing and extending the Xianix Agent codebase.
---

The Xianix Agent is a .NET 8 console application built around [Temporal](https://temporal.io/) workflows (via the Xians platform SDK). It is deliberately small and easy to extend — the core pipeline is a few hundred lines of code, and most customisation happens in `rules.json` without touching the agent at all.

This section covers the internals for developers who want to extend the agent itself: adding new workflows, activities, orchestration logic, or contributing fixes upstream.

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | .NET 8 |
| Workflow engine | Temporal (via Xians platform SDK) |
| Dependency injection | `Microsoft.Extensions.DependencyInjection` |
| Logging | `Microsoft.Extensions.Logging` + console |
| Container management | `Docker.DotNet` |
| Environment config | `DotNetEnv` |
| Test framework | xUnit + NSubstitute |

## Key Concepts

**Workflows** are long-running Temporal state machines. The agent has two:

- `ActivationWorkflow` — runs indefinitely, receives webhook signals, and fans out to `ProcessingWorkflow` instances.
- `ProcessingWorkflow` — handles a single event end-to-end: create volume → start container → wait → collect output → cleanup.

**Activities** are the side-effectful steps that workflows orchestrate. The agent's only activity class is `ContainerActivities`, which manages the Docker container lifecycle.

**Orchestrator** sits between the webhook handler and the workflow layer. `EventOrchestrator` calls `WebhookRulesEvaluator` to match the incoming event against `rules.json`, extracts inputs, and builds the `OrchestrationResult` that gets signalled to `ActivationWorkflow`.

**Rules Evaluator** is a pure stateless component that reads `rules.json` from Xians Knowledge and evaluates filter expressions, extracts payload values, and resolves plugins and prompts. It is the only component with no Temporal dependency and is therefore straightforward to unit-test.

## Where to Start

| Goal | Where to look |
|---|---|
| Change what happens when a webhook fires | `rules.json` — no code change needed |
| Add a new webhook event type | `rules.json` + optionally extend `WebhookRulesModels.cs` |
| Add a new Temporal activity | `TheAgent/Activities/` |
| Change the container lifecycle | `ContainerActivities.cs` + `ProcessingWorkflow.cs` |
| Add a new workflow | `TheAgent/Workflows/` + register in `XianixAgent.cs` |
| Add a new conversational tool | `MafSubAgentTools.cs` |
| Change environment variable handling | `EnvConfig.cs` |
