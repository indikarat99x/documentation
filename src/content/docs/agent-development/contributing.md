---
title: Contributing
description: How to contribute to the Xianix Agent codebase — branching, code style, and PR guidelines.
---

## Getting the Code

```bash
git clone https://github.com/99x/xianix-the-agent.git
cd xianix-the-agent
cp .env.example .env   # fill in required values
```

Verify everything works:

```bash
dotnet build
dotnet test TheAgent.Tests/TheAgent.Tests.csproj
```

## Branching

| Branch | Purpose |
|---|---|
| `main` | Production-ready code. Protected — merges via PR only. |
| `feature/<short-description>` | New features |
| `fix/<short-description>` | Bug fixes |
| `chore/<short-description>` | Dependency updates, CI changes, non-functional improvements |

Branch from `main`:

```bash
git checkout main && git pull
git checkout -b feature/my-feature
```

## Code Style

The project follows standard C# conventions:

- **Namespaces** match the folder path (`Xianix.Workflows`, `Xianix.Activities`, etc.).
- **Interfaces** are prefixed with `I` (`IEventOrchestrator`, `IWebhookRulesEvaluator`).
- **Async methods** are suffixed with `Async`.
- **Private fields** use `_camelCase`.
- **Constants and static readonly** use `PascalCase`.
- Keep workflow classes **deterministic** — see [Workflows & Activities](/agent-development/workflows-and-activities/) for the rules.

## Adding New Features

Before writing code, consider whether the change can be made in `rules.json` alone. The rules file is the primary extension point and requires no compilation or deployment.

If a code change is needed:

1. Add or extend an interface before adding implementation.
2. Register new services in `Program.cs`.
3. Add env-var accessors to `EnvConfig.cs` for any new configuration.
4. Write unit tests in `TheAgent.Tests/` before or alongside the implementation.
5. Update relevant documentation in `the-agent/Docs/` and this documentation site.

## Pull Request Guidelines

- **One concern per PR.** Avoid bundling unrelated changes.
- **Tests required** for logic changes to orchestrator, rules, or activity code.
- **Passing CI** — `dotnet build` and `dotnet test` must pass with no warnings.
- **Description** — include what changed, why, and how to test it manually.
- **Documentation** — update `Docs/` and/or the Starlight documentation site for user-facing changes.

## Releasing

Releases are triggered by pushing a version tag. Both the agent and executor images are published automatically:

```bash
VERSION=v1.2.0
git tag $VERSION
git push origin $VERSION
```

This triggers the GitHub Actions workflows that publish `99xio/xianix-agent` and `99xio/xianix-executor` to Docker Hub with semver tags.

See the [Setup guide](/agent/setup/#publishing) for tag naming conventions and pre-release behaviour.
