---
title: Extending the Agent
description: How to add workflows, activities, and new features — plus testing patterns.
---

Before writing code, ask yourself: **can this change be made in `rules.json` alone?** The rules file is the primary extension point and requires no compilation or deployment. See [Rules Configuration](/agent-configuration/rules/) for the syntax.

If a code change is needed, here's how.

## Adding a New Activity

Activities are the side-effectful steps that workflows orchestrate. The agent's existing activity class is `ContainerActivities`.

1. Add a method to `ContainerActivities.cs` or create a new class in `TheAgent/Activities/`:

```csharp
[Activity]
public async Task<string> MyNewActivityAsync(string input)
{
    // side-effectful work here
    return result;
}
```

2. Register the activity class in `XianixAgent.cs` (the SDK scans for `[Activity]` methods on registered types):

```csharp
xiansAgent.Workflows
    .DefineCustom<ProcessingWorkflow>(new WorkflowOptions { Activable = false })
    .AddActivity<ContainerActivities>()
    .AddActivity<MyNewActivities>();   // add here
```

3. Call it from a workflow:

```csharp
var result = await Workflow.ExecuteActivityAsync(
    (MyNewActivities a) => a.MyNewActivityAsync(input),
    new ActivityOptions { StartToCloseTimeout = TimeSpan.FromMinutes(5) });
```

## Adding a New Workflow

1. Create a class in `TheAgent/Workflows/`:

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

3. Start it from another workflow:

```csharp
await XiansContext.Workflows.StartAsync<MyNewWorkflow>(
    new object[] { input }, Guid.NewGuid().ToString());
```

### Temporal Determinism Rules

Workflows must be **deterministic** — the same inputs must always produce the same commands. This means:

- **Don't** use `DateTime.Now`, `Guid.NewGuid()`, or `Random` directly. Use `Workflow.UtcNow`, `Workflow.NewGuid()`, `Workflow.Random`.
- **Don't** call external services from a workflow. Put all I/O in activities.
- **Don't** use non-Temporal `async`/`await`. Use `Workflow.ExecuteActivityAsync` or `Workflow.WaitConditionAsync`.

## Adding New Configuration

All environment variable access goes through `EnvConfig.cs`:

```csharp
public static string MyNewSetting => Get("MY_NEW_SETTING", "default-value");
```

Add the variable to `.env.example` too.

## Registering New Services

New services are registered in `Program.cs` inside `ConfigureServices()`:

```csharp
services.AddSingleton<IMyService, MyService>();
```

## Testing

The test project lives at `TheAgent.Tests/` and uses xUnit with NSubstitute for mocking.

### Running Tests

```bash
dotnet test TheAgent.Tests/TheAgent.Tests.csproj
```

### Writing Tests

Tests follow a standard Arrange / Act / Assert pattern. Mocks are created via `Substitute.For<TInterface>()`:

```csharp
public class MyComponentTests
{
    private readonly IMyDependency _dep = Substitute.For<IMyDependency>();
    private readonly MyComponent _sut;

    public MyComponentTests()
    {
        _sut = new MyComponent(_dep);
    }

    [Fact]
    public async Task DoWork_WhenInputValid_ReturnsExpected()
    {
        _dep.GetValueAsync().Returns(Task.FromResult("hello"));

        var result = await _sut.DoWork();

        Assert.Equal("hello", result);
    }
}
```

### What to Test

| Component | Focus |
|---|---|
| `WebhookRulesEvaluator` | Filter matching, input extraction, path resolution, edge cases |
| `EventOrchestrator` | Handled vs. ignored results, input propagation, exceptions |
| `EnvConfig` | Required-variable validation, fallback defaults, tenant-scoped keys |
| `ContainerActivities` | Integration tests or manual testing (requires Docker) |

### Conventions

- Mirror the source folder structure under `TheAgent.Tests/`
- Name tests `MethodName_Condition_ExpectedOutcome`
- Use `Substitute.For<T>()` for dependencies, real instances for the SUT

## Next Step

Ready to ship? See [Deployment](/agent-development/deployment/) for Docker publishing and Azure setup.
