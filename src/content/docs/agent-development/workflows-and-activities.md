---
title: Workflows & Activities
description: How Temporal workflows and activities are structured, and how to add new ones.
---

The agent uses [Temporal](https://temporal.io/) (via the Xians platform SDK) for durable execution. Workflows are deterministic state machines; activities are the non-deterministic, side-effectful steps that workflows schedule.

## Existing Workflows

### `ActivationWorkflow`

Registered as `"Xianix AI-DLC Agent:Activation Workflow"`.

Runs indefinitely. It holds a queue of `OrchestrationResult` objects and drains them one at a time, starting a child `ProcessingWorkflow` for each. It uses `ContinueAsNew` to avoid unbounded history growth:

```csharp
[WorkflowRun]
public async Task WorkflowRun()
{
    await ProcessActivationLoopAsync();
}

[WorkflowSignal]
public Task ProcessWebhook(OrchestrationResult result)
{
    _webhookResults.Enqueue(result);
    return Task.CompletedTask;
}
```

`EventOrchestrator` calls `Signal(inputs)` on this workflow whenever a matched webhook arrives.

### `ProcessingWorkflow`

Registered as `"Xianix AI-DLC Agent:Processing Workflow"`.

Handles one event from start to finish. The workflow run method orchestrates the full container lifecycle:

```
1. EnsureWorkspaceVolumeAsync  →  volumeName
2. StartContainerAsync         →  containerId
3. WaitAndCollectOutputAsync   →  ContainerExecutionResult
4. CleanupContainerAsync       →  (always, in finally block)
5. ParseExecutorOutput         →  cost/token metrics from JSON stdout
6. ReportExecutionMetricsAsync →  report to Xians platform
```

Activity options are defined as static fields so timeouts and retry policies are centralised:

```csharp
private static readonly ActivityOptions ContainerActivityOptions = new()
{
    StartToCloseTimeout = TimeSpan.FromMinutes(20),
    RetryPolicy = new() { MaximumAttempts = 3, BackoffCoefficient = 2 },
};
```

## Existing Activities

### `ContainerActivities`

The only activity class. Manages the Docker container lifecycle per execution:

| Activity | What it does |
|---|---|
| `EnsureWorkspaceVolumeAsync` | Creates (or verifies) a named Docker volume for the tenant+repo pair |
| `StartContainerAsync` | Pulls the executor image, creates the container with env vars and volume mount, starts it |
| `WaitAndCollectOutputAsync` | Streams stdout/stderr until the container exits; enforces a timeout |
| `CleanupContainerAsync` | Stops and removes the container (volume is kept) |

## Adding a New Activity

1. Add a method to `ContainerActivities.cs` (or create a new activity class in `TheAgent/Activities/`):

```csharp
[Activity]
public async Task<string> MyNewActivityAsync(string input)
{
    // side-effectful work here
    return result;
}
```

2. Register the activity class with the Xians worker in `XianixAgent.cs`. The SDK scans for `[Activity]`-annotated methods on registered types.

3. Call it from a workflow using `Workflow.ExecuteActivityAsync`:

```csharp
var result = await Workflow.ExecuteActivityAsync(
    (ContainerActivities a) => a.MyNewActivityAsync(input),
    new ActivityOptions { StartToCloseTimeout = TimeSpan.FromMinutes(5) });
```

## Adding a New Workflow

1. Create a new class in `TheAgent/Workflows/` and decorate it with `[Workflow]`:

```csharp
[Workflow(Constants.AgentName + ":My New Workflow")]
public class MyNewWorkflow
{
    [WorkflowRun]
    public async Task WorkflowRun(string input)
    {
        // orchestrate activities here
    }
}
```

2. Register it in `XianixAgent.cs` alongside the existing workflows.

3. Start it from another workflow using `XiansContext.Workflows.StartAsync<MyNewWorkflow>`.

## Workflow Design Rules (Temporal Determinism)

Temporal workflows must be **deterministic** — the same inputs must always produce the same sequence of commands. This means:

- **Never** use `DateTime.Now`, `Guid.NewGuid()`, or `Random` directly inside a workflow. Use `Workflow.UtcNow`, `Workflow.NewGuid()`, and `Workflow.Random` instead.
- **Never** call external services (HTTP, database, Docker) from a workflow. Put all I/O in activities.
- **Never** use `async`/`await` on non-Temporal tasks inside a workflow body. Use `Workflow.ExecuteActivityAsync` or `Workflow.WaitConditionAsync`.

Violating these rules causes non-determinism errors during workflow replay.
