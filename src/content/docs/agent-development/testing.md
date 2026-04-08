---
title: Testing
description: How to write and run tests for the Xianix Agent using xUnit and NSubstitute.
---

The test project lives at `TheAgent.Tests/` and uses [xUnit](https://xunit.net/) with [NSubstitute](https://nsubstitute.github.io/) for mocking.

## Running Tests

```bash
dotnet test TheAgent.Tests/TheAgent.Tests.csproj
```

With verbose output:

```bash
dotnet test TheAgent.Tests/TheAgent.Tests.csproj --logger "console;verbosity=detailed"
```

## Test Structure

Tests are organised by component under `TheAgent.Tests/`, mirroring the source layout:

```
TheAgent.Tests/
├── TheAgent.Tests.csproj
└── Orchestrator/
    └── EventOrchestratorTests.cs
```

## Testing Pattern

The existing tests follow a standard Arrange / Act / Assert pattern with constructor-based dependency injection and NSubstitute mocks.

### Example: `EventOrchestratorTests`

```csharp
public class EventOrchestratorTests
{
    private readonly IWebhookRulesEvaluator _evaluator = Substitute.For<IWebhookRulesEvaluator>();
    private readonly ILogger<EventOrchestrator> _logger = Substitute.For<ILogger<EventOrchestrator>>();
    private readonly EventOrchestrator _sut;

    public EventOrchestratorTests()
    {
        _sut = new EventOrchestrator(_evaluator, _logger);
    }

    [Fact]
    public async Task OrchestrateAsync_WhenEvaluatorReturnsInputs_ReturnsMatchedResult()
    {
        // Arrange
        var inputs = new Dictionary<string, object?> { ["pr_id"] = 42L };
        var evaluation = new EvaluationResult(inputs, [], "");
        _evaluator.EvaluateAsync("github-pr", Arg.Any<object?>())
                  .Returns(Task.FromResult<EvaluationResult?>(evaluation));

        // Act
        var result = await _sut.OrchestrateAsync("github-pr", new { }, "tenant-1");

        // Assert
        Assert.True(result.Handled);
        Assert.Equal("github-pr", result.WebhookName);
        Assert.Equal(42L, result.Inputs["pr_id"]);
    }

    [Fact]
    public async Task OrchestrateAsync_WhenEvaluatorReturnsNull_ReturnsIgnoredResult()
    {
        _evaluator.EvaluateAsync("unknown-webhook", Arg.Any<object?>())
                  .Returns(Task.FromResult<EvaluationResult?>(null));

        var result = await _sut.OrchestrateAsync("unknown-webhook", new { }, "tenant-1");

        Assert.False(result.Handled);
        Assert.Empty(result.Inputs);
    }
}
```

### Key conventions

- Mocks are created as `Substitute.For<TInterface>()` in the constructor.
- The system-under-test (`_sut`) is always a real instance, not a mock.
- Test names follow `MethodName_Condition_ExpectedOutcome`.

## What to Test

| Component | What to test |
|---|---|
| `WebhookRulesEvaluator` | Filter matching (== / !=), input extraction, path resolution, missing paths, constant inputs |
| `EventOrchestrator` | Handled vs ignored results, input propagation, exception propagation |
| `ContainerActivities` | Unit-test is impractical (needs Docker); use integration tests or manual testing |
| `EnvConfig` | Required-variable throwing, fallback defaults, tenant-scoped key resolution |

## Adding Tests for a New Component

1. Create a folder under `TheAgent.Tests/` matching the source namespace.
2. Create a `<ClassName>Tests.cs` file.
3. Inject interfaces via constructor and use `Substitute.For<T>()`.
4. Use `Assert.True/False/Equal/Throws*` from xUnit — no assertion library required.

```csharp
public class MyNewComponentTests
{
    private readonly IMyDependency _dep = Substitute.For<IMyDependency>();
    private readonly MyNewComponent _sut;

    public MyNewComponentTests()
    {
        _sut = new MyNewComponent(_dep);
    }

    [Fact]
    public async Task DoWork_WhenDependencySucceeds_ReturnsExpected()
    {
        _dep.GetValueAsync().Returns(Task.FromResult("hello"));

        var result = await _sut.DoWork();

        Assert.Equal("hello", result);
    }
}
```
