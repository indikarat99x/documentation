---
title: Requirement Analyst
description: Grooming backlog items by analyzing user intent, domain knowledge, and competitive context.
---

The `req-analyst` plugin grooms requirements by understanding *why* a feature exists, *what* success means to users, and *how* it fits the user journey. It runs four analysts in two phases and compiles findings into a single structured requirement.

**Version:** 1.0.0 · **Maintainer:** 99x / Xianix Team

---

## What This Plugin Does

**Phase 1 — Context Gathering (parallel):**

| Agent | Focus |
|---|---|
| **intent-analyst** | Intent decomposition, user context, workflow, decision points |
| **domain-analyst** | Domain knowledge, data meaning, business rules, competitive insights (via web search) |

**Phase 2 — Gap & Risk Analysis:**

| Agent | Focus |
|---|---|
| **gap-risk-analyst** | Gaps, risks, value/priority, dependencies |

### Output

- **Verdict:** `GROOMED` | `NEEDS CLARIFICATION` | `NEEDS DECOMPOSITION`
- **Summary** with intent decomposition (stated need → underlying intent → success definition)
- **User Context & Workflow** — who, when/where, constraints, before/during/after
- **Domain Context** — data meaning, business rules, competitive insights
- **Gaps & Unresolved Questions**
- **Risks, Dependencies & Assumptions**
- Automatic posting to the backlog platform (GitHub Issues or Azure DevOps)

---

## Local Testing with Claude Code

### Prerequisites

- [Claude Code](https://docs.anthropic.com/claude-code) installed
- GitHub Personal Access Token with `repo` scope
- Working directory: your project repo

### 1. Point Claude Code at the Plugin

```bash
claude \
  --plugin-dir /path/to/xianix-plugins-official/plugins/req-analyst \
  --mcp-config ~/.claude/my-mcp-config.json
```

### 2. Configure MCP (GitHub + web search)

See [MCP Configuration](/plugins/req-analyst/mcp-config/) for the full setup.

### 3. Invoke the Command

```text
/requirement-analysis 42
```

Elaborate issue #42. The agent fetches the issue, runs all analysts (including competitive/market research via DuckDuckGo), and posts the elaborated requirement to GitHub.

**Post as comment instead of updating the issue body:**

```text
/requirement-analysis 42 --comment
```

---

## Central Run with Script

The script `scripts/run-requirement-analysis.sh` is designed for **server/CI** runs: it clones the target repo into an isolated worktree, injects MCP config, runs the analysis, and cleans up.

### GitHub

```bash
PLATFORM=github \
REPO_URL=https://github.com/org/repo.git \
ISSUE_NUMBER=42 \
GITHUB_TOKEN=ghp_xxx \
./scripts/run-requirement-analysis.sh
```

### Azure DevOps

```bash
PLATFORM=azure-devops \
REPO_URL=https://dev.azure.com/org/project/_git/repo \
ISSUE_NUMBER=123 \
AZURE_TOKEN=pat_xxx \
GIT_TOKEN=pat_xxx \
./scripts/run-requirement-analysis.sh
```

### Required Environment Variables

#### GitHub

| Variable | Description |
|---|---|
| `PLATFORM` | `github` |
| `REPO_URL` | Full HTTPS clone URL |
| `ISSUE_NUMBER` | GitHub issue number to elaborate |
| `GITHUB_TOKEN` | PAT with `repo` scope |

#### Azure DevOps

| Variable | Description |
|---|---|
| `PLATFORM` | `azure-devops` |
| `REPO_URL` | Full HTTPS clone URL |
| `ISSUE_NUMBER` | Work Item ID to elaborate |
| `AZURE_TOKEN` | PAT with Work Items (Read & Write) scopes |
| `GIT_TOKEN` | PAT for git clone (often same as `AZURE_TOKEN`) |

---

## Documentation

| Document | Description |
|---|---|
| [MCP Configuration](/plugins/req-analyst/mcp-config/) | GitHub MCP server and DuckDuckGo web search setup |
| [Backlog Setup](/plugins/req-analyst/backlog-setup/) | GitHub Issues labels and recommended issue structure |
