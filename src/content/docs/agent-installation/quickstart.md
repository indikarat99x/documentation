---
title: Quick Start
description: Get the Xianix Agent running from scratch in a few minutes.
---

This guide walks you through getting the agent running end-to-end — from cloning the repo to receiving your first automated PR review.

## 1. Clone the Repository

```bash
git clone https://github.com/99x/xianix-the-agent.git
cd xianix-the-agent
```

## 2. Configure Environment Variables

Copy the example env file and fill in your credentials:

```bash
cp .env.example .env
```

Open `.env` and set at minimum:

```env
XIANS_SERVER_URL=https://app.xians.ai
XIANS_API_KEY=<your-xians-api-key>
LLM_API_KEY=<your-anthropic-api-key>

# Pick the platform(s) you're connecting to
GITHUB_TOKEN=<your-github-pat>
# AZURE_DEVOPS_TOKEN=<your-ado-pat>
```

See [Configuration Reference](/agent-installation/configuration/) for the full list of variables.

## 3. Pull the Executor Image

The agent spawns executor containers to run plugins. Pull the image now so the first run doesn't incur a cold pull:

```bash
docker pull 99xio/xianix-executor:latest
```

## 4. Run the Agent

```bash
dotnet run --project TheAgent/TheAgent.csproj
```

A healthy start looks like this — four workflow queues registered and workers listening:

```
│ REGISTERED WORKFLOWS (4)                                      │
  Agent:    Xianix AI-DLC Agent
  ...
✓ Worker listening on queue 'xianix:...:Processing Workflow'
✓ Worker listening on queue 'xianix:...:Activation Workflow'
✓ Worker listening on queue 'xianix:...:Supervisor Workflow'
✓ Worker listening on queue 'xianix:...:Integrator Workflow'
```

## 5. Configure a Webhook in Xians

Log in to [app.xians.ai](https://app.xians.ai) and connect your GitHub or Azure DevOps repository to the agent. Xians will generate a webhook URL to add to your repository settings.

## 6. Test with a Simulated Webhook

Before opening a real PR, verify the end-to-end flow with a simulated event:

```bash
export WEBHOOK_URL=https://app.xians.ai/webhooks/<your-agent-id>
./Scripts/simulate-pr-opened.sh
```

Expected response:

```json
{ "status": "success" }
```

You're ready. The next PR opened in your connected repository will trigger the agent automatically.

## Next Steps

- Customise which events trigger which plugins → [Rules Configuration](/agent/rules/)
- Deploy to production on Azure → [Azure Deployment](/agent/azure-deployment/)
- Run inside Docker → [Setup](/agent/setup/)
