---
title: Project Structure
description: Directory layout and the purpose of every source file in the Xianix Agent codebase.
---

## Repository Layout

```
the-agent/
├── .env.example                    # Environment variable template
├── global.json                     # .NET SDK version pin
├── README.md
│
├── Docs/                           # Architecture & design docs
│   ├── architecture.md
│   ├── architecture-diagrams.html
│   ├── azure-deployment.md
│   ├── executor.md
│   ├── rules-json.md
│   └── tenant-isolation-design.md
│
├── TheAgent/                       # .NET control plane
│   ├── TheAgent.csproj
│   ├── Program.cs                  # Entry point — DI wiring, startup
│   ├── Constants.cs                # Agent name and well-known strings
│   ├── EnvConfig.cs                # Typed env-var accessors
│   │
│   ├── Agent/
│   │   ├── XianixAgent.cs          # Top-level agent — registers workflows and runs
│   │   ├── MafSubAgent.cs          # Conversational LLM sub-agent
│   │   ├── MafSubAgentTools.cs     # Tools available to the conversational agent
│   │   └── ChatHistoryProvider.cs  # Manages conversation context
│   │
│   ├── Workflows/
│   │   ├── ActivationWorkflow.cs   # Long-running; fans out to ProcessingWorkflow per event
│   │   └── ProcessingWorkflow.cs   # Single-event lifecycle: volume → container → wait → cleanup
│   │
│   ├── Activities/
│   │   ├── ContainerActivities.cs        # Docker container lifecycle management
│   │   ├── ContainerExecutionInput.cs    # Input model for container activities
│   │   └── ContainerExecutionResult.cs   # Output model — stdout, stderr, exit code, cost
│   │
│   ├── Orchestrator/
│   │   ├── IEventOrchestrator.cs         # Interface
│   │   ├── EventOrchestrator.cs          # Routes webhooks → RulesEvaluator → workflow signal
│   │   └── OrchestrationResult.cs        # Result model carrying inputs, plugins, prompt
│   │
│   ├── Rules/
│   │   ├── IWebhookRulesEvaluator.cs     # Interface
│   │   ├── WebhookRulesEvaluator.cs      # Evaluates rules.json against webhook payload
│   │   └── WebhookRulesModels.cs         # Deserialization models for rules.json
│   │
│   └── Knowledge/
│       └── rules.json                    # Active rules configuration
│
├── TheAgent.Tests/                 # xUnit test project
│   ├── TheAgent.Tests.csproj
│   └── Orchestrator/
│       └── EventOrchestratorTests.cs
│
└── Executor/                       # Python Docker image (separate artifact)
    ├── Dockerfile
    ├── entrypoint.sh
    ├── execute_plugin.py
    └── requirements.txt
```

## Key Files

### `Program.cs`

Entry point. Loads `.env`, wires up the DI container, and calls `agent.RunAsync()`:

```csharp
EnvConfig.Load();

var services = new ServiceCollection();
services.AddSingleton<IWebhookRulesEvaluator, WebhookRulesEvaluator>();
services.AddSingleton<IEventOrchestrator, EventOrchestrator>();
services.AddSingleton<XianixAgent>();

var agent = serviceProvider.GetRequiredService<XianixAgent>();
await agent.RunAsync();
```

Adding a new service means registering it here before `BuildServiceProvider()`.

### `EnvConfig.cs`

All environment variable access goes through `EnvConfig`. It provides typed, named properties and throws clearly on missing required values:

```csharp
public static string XiansServerUrl => GetRequired("XIANS_SERVER_URL");
public static string GithubToken    => Get("GITHUB_TOKEN");

// Tenant-scoped token fallback: MY_ORG_GITHUB_TOKEN → GITHUB_TOKEN
public static string GetGithubToken(string tenantId) =>
    Get(TenantKey(tenantId, "GITHUB_TOKEN"), Get("GITHUB_TOKEN"));
```

Add new variables here when introducing new configuration.

### `Constants.cs`

Centralises strings that must stay in sync across the codebase (e.g. the agent name used in Temporal workflow registrations).

### `OrchestrationResult.cs`

The data contract that flows from `EventOrchestrator` through `ActivationWorkflow` into `ProcessingWorkflow`. Carries `TenantId`, `WebhookName`, `Inputs`, and the `Execution` spec (plugins + prompt).
