---
title: Deployment
description: Docker image publishing, CI pipelines, and Azure VM deployment.
---

## Docker Images

The project produces two Docker images:

| Image | Dockerfile | Purpose |
|-------|------------|---------|
| `99xio/xianix-agent` | `TheAgent/Dockerfile` | The .NET control plane |
| `99xio/xianix-executor` | `Executor/Dockerfile` | The per-execution Python/Node container |

### Building Locally

```bash
# Agent
cd TheAgent/
docker build -t 99xio/xianix-agent:latest .

# Executor
cd Executor/
docker build -t 99xio/xianix-executor:latest .
```

### CI Publishing

Both images are published automatically via GitHub Actions when you push a version tag:

```bash
git tag v1.2.0
git push origin v1.2.0
```

Tags are derived from the version:

| Git Tag | Docker Hub Tags |
|---------|-----------------|
| `v1.2.3` | `1.2.3`, `1.2`, `1`, `latest` |
| `v2.0.0-beta.1` | `2.0.0-beta.1` (no `latest`) |

CI builds multi-arch images (`linux/amd64` + `linux/arm64`).

## Azure VM Deployment

The production agent runs on an Azure Linux VM with:

- **No public IP** — all traffic is outbound-only via NAT Gateway
- **Secrets in Key Vault** — fetched at runtime via managed identity (no `.env` on disk)
- **Systemd service** — auto-restarts on crash or reboot

### Architecture

```
Azure VNet (outbound-only via NAT Gateway)
  └─ VM: xianix-agent-vm (Ubuntu 22.04, Standard_B2s)
       ├─ systemd: xianix-agent.service
       │    └─ /etc/xianix/start-agent.sh
       │         ├─ 1. Get token from Azure IMDS (managed identity)
       │         ├─ 2. Fetch secrets from Key Vault
       │         └─ 3. docker run xianix-agent (with secrets as env vars)
       └─ Docker Engine
            └─ xianix-executor containers (spawned per event)
```

### Key Vault Secrets

Secret names mirror env var names with hyphens: `XIANS-SERVER-URL`, `ANTHROPIC-API-KEY`, etc.

```bash
az keyvault secret set --vault-name xianix-kv-agent \
  --name XIANS-API-KEY --value "<your-key>"
```

### Common Operations

```bash
# Define a shortcut (optional)
alias vmrun='az vm run-command invoke \
  --resource-group xianix-agent-rg \
  --name xianix-agent-vm \
  --command-id RunShellScript --scripts'

# Start / restart / stop
vmrun "sudo systemctl start xianix-agent"
vmrun "sudo systemctl restart xianix-agent"
vmrun "sudo systemctl stop xianix-agent"

# Check logs
vmrun "docker logs --tail 50 xianix-agent 2>&1"

# Update images
vmrun "docker pull 99xio/xianix-agent:latest && sudo systemctl restart xianix-agent"
vmrun "docker pull 99xio/xianix-executor:latest"

# Rotate a secret
az keyvault secret set --vault-name xianix-kv-agent \
  --name ANTHROPIC-API-KEY --value "<new-key>"
vmrun "sudo systemctl restart xianix-agent"
```

For live log tailing, connect via **Azure Bastion** (Developer SKU) through the Azure Portal.

## Next Step

See [Contributing](/agent-development/contributing/) for branching conventions and PR guidelines.
