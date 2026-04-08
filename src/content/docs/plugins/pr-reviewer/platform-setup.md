---
title: Platform Setup
description: How to configure GitHub MCP, gh CLI, and Azure DevOps CLI for the PR Reviewer plugin.
---

The `pr-reviewer` plugin works with any git hosting platform. All diff analysis uses standard git commands. Only the **review posting** step is platform-specific.

---

## GitHub

### Option A: GitHub MCP Server (recommended for inline comments)

The MCP server enables the richest review experience — inline comments posted directly on PR files.

Create `~/.claude/my-mcp-config.json`:

```json
{
  "mcpServers": {
    "github": {
      "url": "https://api.github.com",
      "token": "${GITHUB_TOKEN}"
    }
  }
}
```

Launch with:

```bash
export GITHUB_TOKEN=ghp_your_token_here
claude --mcp-config ~/.claude/my-mcp-config.json
```

**Verify:** Run `/mcp` inside Claude Code — `github` should show as `connected`.

**Token scopes needed:** `repo` (private repos) or `public_repo` (public only), `read:org` (optional).

### Option B: `gh` CLI (fallback)

If MCP is unavailable, the plugin falls back to the `gh` CLI automatically.

```bash
gh auth login
```

### Credentials for `git push` (fix mode)

When using `--fix`, the agent pushes commits. Pass the token at runtime:

```bash
GIT_TOKEN=ghp_your_token_here claude ...
```

---

## Azure DevOps

### Prerequisites

```bash
az extension add --name azure-devops
```

### Authentication

**Interactive login:**

```bash
az login
az devops configure --defaults organization=https://dev.azure.com/<your-org>
```

**Personal Access Token (recommended for CI):**

```bash
export AZURE_DEVOPS_PAT=<your-pat>
echo $AZURE_DEVOPS_PAT | az devops login --org https://dev.azure.com/<your-org>
```

**PAT scopes needed:**
- `Code` → Read & Write
- `Pull Request Threads` → Read & Write

The plugin reuses `AZURE_DEVOPS_PAT` for `git push` credential injection automatically — no separate `GIT_TOKEN` is needed for Azure DevOps remotes.

### Generating a PAT

1. Go to `https://dev.azure.com/<your-org>/_usersSettings/tokens`
2. Click **New Token**
3. Set the scopes listed above
4. Export as `AZURE_DEVOPS_PAT`

---

## Bitbucket / Other Platforms

For platforms without native CLI support, the plugin writes the review report to `pr-review-report.md` in the repository root. No additional setup is required beyond a working git installation.

---

## Summary

| Platform | Review posting method | Token variable | Fix mode push |
|---|---|---|---|
| GitHub (MCP) | `mcp__github__create_pull_request_review` | `GITHUB_TOKEN` | `GIT_TOKEN` |
| GitHub (CLI) | `gh pr review` | `gh auth login` | `GIT_TOKEN` |
| Azure DevOps | `az repos pr` + REST API | `AZURE_DEVOPS_PAT` | `AZURE_DEVOPS_PAT` |
| Generic | Write to `pr-review-report.md` | — | `GIT_TOKEN` |
