---
title: PR Reviewer
description: Comprehensive pull request reviews with specialized agents for code quality, security, test coverage, and performance.
---

The `pr-reviewer` plugin performs comprehensive pull request reviews using specialized sub-agents. It works with GitHub, Azure DevOps, Bitbucket, and any git repository.

**Version:** 1.1.0 · **Maintainer:** 99x / Xianix Team

---

## What This Plugin Does

The plugin runs four specialized reviewers in parallel and compiles findings into a single structured review:

| Reviewer | Focus |
|---|---|
| **Code quality** | Architecture, patterns, readability, maintainability |
| **Security** | Vulnerabilities, exposed secrets, insecure patterns (OWASP) |
| **Test coverage** | Test completeness, quality, untested code paths |
| **Performance** | Bottlenecks, algorithmic inefficiencies, resource waste |

Results are posted directly as review comments on the PR (GitHub/Azure DevOps) or written to `pr-review-report.md` for other platforms.

---

## Local Testing with Claude Code

### Prerequisites

- [Claude Code](https://docs.anthropic.com/claude-code) installed (`claude` CLI)
- A GitHub or Azure DevOps token
- Working directory: your project repo or a clone of the target repo

### 1. Point Claude Code at the Plugin

```bash
claude \
  --plugin-dir /path/to/xianix-plugins-official/plugins/pr-reviewer
```

### 2. Run a Review

In the Claude chat:

```text
/pr-review
```

The plugin detects the current repo and open PR automatically. You can also pass a PR number explicitly.

---

## Automated Run via Xianix Agent

The plugin is designed to be triggered automatically by the Xianix Agent when a PR is opened. Add a rule to your `rules.json`:

```json
{
  "webhook-name": "pull requests",
  "match": [
    { "name": "pr-opened-event", "rule": "action==opened" }
  ],
  "inputs": [
    { "name": "pr-number",        "value": "number" },
    { "name": "repository-url",   "value": "repository.clone_url" },
    { "name": "repository-name",  "value": "repository.full_name" },
    { "name": "pr-title",         "value": "pull_request.title" },
    { "name": "pr-head-branch",   "value": "pull_request.head.ref" },
    { "name": "platform",         "value": "github", "constant": true }
  ],
  "claude-code-plugins": [
    {
      "name": "github",
      "url": "github@claude-plugins-official",
      "marketplace": "anthropics/claude-plugins-official"
    },
    {
      "name": "pr-reviewer",
      "url": "pr-reviewer@xianix-plugins-official",
      "marketplace": "xianix-team/plugins-official",
      "envs": [
        { "name": "GITHUB_PERSONAL_ACCESS_TOKEN", "value": "env.GITHUB_TOKEN" }
      ]
    }
  ],
  "prompt": "Review PR #{{pr-number}} titled \"{{pr-title}}\" in {{repository-name}} (branch: {{pr-head-branch}}).\n\nRun /code-review to perform the automated review."
}
```

---

## Documentation

| Document | Description |
|---|---|
| [Platform Setup](/plugins/pr-reviewer/platform-setup/) | GitHub MCP, `gh` CLI, Azure DevOps CLI setup |
| [Git Authentication](/plugins/pr-reviewer/git-auth/) | Runtime credential injection for `git push` |
