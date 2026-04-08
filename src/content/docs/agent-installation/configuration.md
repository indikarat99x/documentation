---
title: Configuration Reference
description: All environment variables accepted by the Xianix Agent.
---

The agent is configured entirely via environment variables. Copy `.env.example` to `.env` and populate the values below.

## Core

| Variable | Required | Description |
|---|---|---|
| `XIANS_SERVER_URL` | Yes | Xians platform base URL, e.g. `https://app.xians.ai` |
| `XIANS_API_KEY` | Yes | API key for your agent, obtained from Xians Agent Studio |
| `LLM_API_KEY` | Yes | Anthropic API key used by the Claude Code SDK inside executor containers |

## Platform Tokens

At least one platform token is required. Include both if your agent handles events from both GitHub and Azure DevOps.

| Variable | Required | Description |
|---|---|---|
| `GITHUB_TOKEN` | Conditional | GitHub Personal Access Token. Required when handling GitHub webhooks. Scopes: `repo`, `read:org` |
| `AZURE_DEVOPS_TOKEN` | Conditional | Azure DevOps Personal Access Token. Required when handling Azure DevOps webhooks. Scopes: `Code (Read & Write)`, `Pull Request Threads (Read & Write)` |

## Executor Container

| Variable | Default | Description |
|---|---|---|
| `EXECUTOR_IMAGE` | `99xio/xianix-executor:latest` | Docker image used for executor containers |
| `CONTAINER_MEMORY_MB` | `1024` | Memory limit per executor container (in MB) |
| `CONTAINER_CPU_COUNT` | `1` | CPU allocation per executor container |

## Docker

| Variable | Default | Description |
|---|---|---|
| `DOCKER_HOST` | *(system default)* | Override the Docker daemon socket, e.g. `unix:///var/run/docker.sock` |

## Example `.env`

```env
# Core
XIANS_SERVER_URL=https://app.xians.ai
XIANS_API_KEY=xns_...
LLM_API_KEY=sk-ant-...

# Platforms (include whichever you need)
GITHUB_TOKEN=ghp_...
# AZURE_DEVOPS_TOKEN=...

# Executor (optional overrides)
EXECUTOR_IMAGE=99xio/xianix-executor:latest
CONTAINER_MEMORY_MB=1024
CONTAINER_CPU_COUNT=1
```

## Azure Key Vault (Production)

When running on Azure, secrets are stored in Key Vault and fetched at startup via the VM's managed identity — no `.env` file is kept on disk. Secret names mirror the variable names above with hyphens instead of underscores (e.g. `XIANS-API-KEY`).

See [Azure Deployment](/agent/azure-deployment/) for the full setup.
