---
title: Git Authentication
description: How runtime git credentials are injected for git push operations in the PR Reviewer plugin.
---

The `pr-reviewer` plugin can apply code fixes and push them directly to the PR branch. Since the agent runtime may operate against different repositories with different access levels, git credentials are passed at runtime via environment variables — never hardcoded, never written to disk or `~/.gitconfig`.

---

## How It Works

The plugin uses **`GIT_CONFIG_COUNT` environment variables** (Git 2.31+) to inject a token transparently into every `git push` command for the session. This rewrites any HTTPS remote URL to use the token inline, scoped only to the current shell process.

The `validate-prerequisites.sh` hook sets this up automatically before every `git push`, detecting the platform from the remote URL and injecting the correct token.

---

## Credentials by Platform

### GitHub

| Variable | Used by | Purpose |
|---|---|---|
| `GITHUB_TOKEN` | GitHub MCP server | Read PR metadata, post review comments via GitHub API |
| `GIT_TOKEN` | Local `git push` | Authenticate HTTPS pushes to the PR branch |

These are typically the same PAT. The hook injects `GIT_TOKEN` as:

```bash
GIT_CONFIG_COUNT=1
GIT_CONFIG_KEY_0="url.https://x-access-token:<GIT_TOKEN>@github.com/.insteadOf"
GIT_CONFIG_VALUE_0="https://github.com/"
```

**Generating a GitHub PAT:**
1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. Click **Generate new token (classic)**
3. Select scopes: `repo` (private repos) or `public_repo` (public repos only)

### Azure DevOps

| Variable | Used by | Purpose |
|---|---|---|
| `AZURE_DEVOPS_PAT` | `az` CLI + Local `git push` | Authenticate API calls and HTTPS pushes |

A single PAT covers both API access and git push. The hook injects `AZURE_DEVOPS_PAT` for both `dev.azure.com` and `*.visualstudio.com` remote URLs.

**Generating an Azure DevOps PAT:**
1. Go to `https://dev.azure.com/<your-org>/_usersSettings/tokens`
2. Click **New Token**
3. Select scopes: `Code (Read & Write)`, `Pull Request Threads (Read & Write)`

---

## Passing Credentials at Runtime

### Inline (single session)

**GitHub:**
```bash
GITHUB_TOKEN=ghp_xxx GIT_TOKEN=ghp_xxx claude --mcp-config ~/.claude/my-mcp-config.json
```

**Azure DevOps:**
```bash
AZURE_DEVOPS_PAT=<pat> claude
```

### Via Shell Export

```bash
# GitHub
export GITHUB_TOKEN=ghp_xxx
export GIT_TOKEN=ghp_xxx

# Azure DevOps
export AZURE_DEVOPS_PAT=<pat>
```

### Via `.env` File (per-project, never committed)

```bash
# GitHub
GITHUB_TOKEN=ghp_xxx
GIT_TOKEN=ghp_xxx

# Azure DevOps
AZURE_DEVOPS_PAT=<pat>
```

Then source it before launching:

```bash
source .env && claude
```

---

## What Happens If a Token Is Missing

The `validate-prerequisites.sh` hook blocks any `git push` attempt if the required token is not set:

**GitHub:**
```
blocked: GIT_TOKEN is not set. Pass it at runtime: GIT_TOKEN=ghp_xxx claude ...
```

**Azure DevOps:**
```
blocked: AZURE_DEVOPS_PAT is not set. Pass it at runtime: AZURE_DEVOPS_PAT=<pat> claude ...
```

`git commit` and other local operations are unaffected — only push requires the token.

---

## Verification

After setting the token, verify git can push with a dry-run:

```bash
git push --dry-run origin HEAD
```

If it completes without a credential prompt, the token is injected correctly.

---

## Summary

| Platform | Token for API | Token for git push |
|---|---|---|
| GitHub (MCP) | `GITHUB_TOKEN` | `GIT_TOKEN` |
| GitHub (CLI) | `gh auth login` | `GIT_TOKEN` |
| Azure DevOps | `AZURE_DEVOPS_PAT` | `AZURE_DEVOPS_PAT` (same) |
