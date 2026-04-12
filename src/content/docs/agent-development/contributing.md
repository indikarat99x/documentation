---
title: Contributing
description: Branching conventions, code style, and PR guidelines for the Xianix Agent.
---

## Getting the Code

```bash
git clone https://github.com/xianix-team/the-agent.git
cd the-agent
cp TheAgent/.env.example TheAgent/.env  # fill in required values
dotnet build
dotnet test TheAgent.Tests/TheAgent.Tests.csproj
```

## Branching

| Branch | Purpose |
|---|---|
| `main` | Production-ready. Protected — merges via PR only. |
| `feature/<short-name>` | New features |
| `fix/<short-name>` | Bug fixes |
| `chore/<short-name>` | CI, deps, non-functional changes |

```bash
git checkout main && git pull
git checkout -b feature/my-feature
```

## Code Style

Standard C# conventions:

- **Namespaces** match folder paths (`Xianix.Workflows`, `Xianix.Activities`)
- **Interfaces** prefixed with `I` (`IEventOrchestrator`)
- **Async methods** suffixed with `Async`
- **Private fields** use `_camelCase`
- **Constants** use `PascalCase`
- Workflow classes must be **deterministic** — see [Extending the Agent](/agent-development/extending/)

## Before You Code

Consider whether the change can be made in `rules.json` alone — it's the primary extension point and requires no compilation.

If code is needed:

1. Add or extend an **interface** before the implementation
2. Register new services in **`Program.cs`**
3. Add env-var accessors to **`EnvConfig.cs`**
4. Write **unit tests** alongside the implementation
5. Update docs if user-facing behavior changes

## Pull Request Guidelines

- **One concern per PR** — don't bundle unrelated changes
- **Tests required** for logic changes to orchestrator, rules, or activities
- **CI must pass** — `dotnet build` and `dotnet test` with no warnings
- **Describe what, why, and how to test** in the PR description

## Releasing

Push a version tag to trigger CI:

```bash
git tag v1.2.0
git push origin v1.2.0
```

This publishes both `99xio/xianix-agent` and `99xio/xianix-executor` to Docker Hub. See [Deployment](/agent-development/deployment/) for tag conventions.
