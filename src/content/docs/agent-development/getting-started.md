---
title: Getting Started
description: Clone, configure, and run the Xianix Agent locally in under 5 minutes.
---

## Prerequisites

- **.NET SDK 10** — pinned in `global.json`
- **Docker** — the agent spawns executor containers via the Docker API
- **A Xians account** — get an API key from [app.xians.ai](https://app.xians.ai)
- **An Anthropic API key** — used by Claude Code inside executor containers
- **A GitHub or Azure DevOps PAT** — for repo access

## 1. Clone and Configure

```bash
git clone https://github.com/xianix-team/the-agent.git
cd the-agent
cp TheAgent/.env.example TheAgent/.env
```

Open `TheAgent/.env` and fill in the required values:

```env
# Required
XIANS_SERVER_URL=https://app.xians.ai
XIANS_API_KEY=xns_...
ANTHROPIC_API_KEY=sk-ant-...

# Platform tokens — include whichever you need
GITHUB_TOKEN=ghp_...
# AZURE_DEVOPS_TOKEN=...

# Executor (optional overrides)
EXECUTOR_IMAGE=99xio/xianix-executor:latest
CONTAINER_MEMORY_MB=1024
CONTAINER_CPU_COUNT=1
```

### Environment Variable Reference

| Variable | Required | Description |
|---|---|---|
| `XIANS_SERVER_URL` | Yes | Xians platform URL |
| `XIANS_API_KEY` | Yes | API key from Xians Agent Studio |
| `ANTHROPIC_API_KEY` | Yes | Anthropic API key for Claude Code |
| `GITHUB_TOKEN` | Conditional | GitHub PAT. Required for GitHub webhooks. |
| `AZURE_DEVOPS_TOKEN` | Conditional | Azure DevOps PAT. Required for ADO webhooks. |
| `EXECUTOR_IMAGE` | No | Docker image for the executor (default: `99xio/xianix-executor:latest`) |
| `CONTAINER_MEMORY_MB` | No | Memory limit per container in MB (default: `1024`) |
| `CONTAINER_CPU_COUNT` | No | CPU cores per container (default: `1`) |

:::tip[Per-tenant tokens]
You can set tenant-specific tokens using the format `<TENANT_ID>_GITHUB_TOKEN`. The agent checks for a tenant-scoped token first, then falls back to the shared `GITHUB_TOKEN`.
:::

### Multiple Environments

The agent supports environment-specific `.env` files via the `APP_ENV` variable:

```bash
APP_ENV=prod dotnet run --project TheAgent/TheAgent.csproj  # loads .env.prod
```

## 2. Run the Agent

```bash
dotnet run --project TheAgent/TheAgent.csproj
```

A healthy agent prints a banner and starts listening on its workflow queues:

```
╔══════════════════════════════╗
║     The Xianix Agent v1.0    ║
╚══════════════════════════════╝
✓ Worker listening on queue 'xianix:...:Processing Workflow'
✓ Worker listening on queue 'xianix:...:Integrator Workflow'
...
```

### Running in Docker

```bash
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --env-file TheAgent/.env \
  99xio/xianix-agent:latest
```

The Docker socket mount is required — the agent creates executor containers via the Docker API.

## 3. Verify with a Test Webhook

With the agent running, simulate a GitHub PR event:

```bash
export WEBHOOK_URL=https://app.xians.ai/webhooks/<your-agent-id>
./TestScripts/simulate-pr-opened.sh
```

You should see `{ "status": "success" }` for a matching event or `{ "status": "ignored" }` for an unmatched one.

## 4. Run Tests

```bash
dotnet test TheAgent.Tests/TheAgent.Tests.csproj
```

## Next Step

Now that the agent is running, learn [How It Works](/agent-development/how-it-works/) to understand the webhook-to-container pipeline.
