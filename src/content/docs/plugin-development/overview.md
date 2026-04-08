---
title: Plugin Structure
description: How to build a Xianix plugin — directory layout, required files, and available extension points.
---

Xianix plugins are Claude Code plugins — self-contained directories that extend what the agent can do when it runs against a repository. Each plugin follows a standard structure that the Xianix executor knows how to install and invoke.

---

## Directory Layout

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata (required)
├── .mcp.json            # MCP server configuration (optional)
├── commands/            # Slash commands (optional)
├── agents/              # Sub-agent definitions (optional)
├── skills/              # Skill definitions (optional)
├── hooks/               # Lifecycle hooks (optional)
├── providers/           # Provider-specific configuration (optional)
├── styles/              # Output style definitions (optional)
└── README.md            # Documentation
```

---

## `plugin.json` (required)

The plugin manifest. Located at `.claude-plugin/plugin.json`.

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "A brief description of what this plugin does.",
  "author": {
    "name": "Your Name",
    "email": "you@example.com",
    "url": "https://github.com/your-org"
  },
  "homepage": "https://github.com/your-org/my-plugin",
  "repository": "https://github.com/your-org/my-plugin",
  "license": "MIT",
  "keywords": ["example", "review"],
  "commands": ["./commands/my-command.md"],
  "skills": "./skills/",
  "hooks": "./hooks/hooks.json",
  "outputStyles": "./styles/",
  "providers": "./providers/"
}
```

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Plugin identifier (lowercase, hyphens only) |
| `version` | Yes | Semver version string |
| `description` | Yes | One-line description |
| `commands` | No | Array of paths to command markdown files |
| `skills` | No | Path to skills directory |
| `hooks` | No | Path to hooks config JSON |
| `outputStyles` | No | Path to output style definitions |
| `providers` | No | Path to provider-specific config |

---

## Commands

Slash commands are markdown files that define a prompt Claude runs when invoked. Place them in `commands/`.

```markdown
---
description: Review the current pull request
allowed-tools:
  - Bash
  - WebSearch
---

Review the pull request at the current HEAD. Check for:
- Code quality issues
- Security vulnerabilities
- Missing tests
- Performance concerns

Post findings as review comments.
```

The command is invoked as `/my-command` in the Claude chat.

---

## Hooks

Hooks run shell scripts at specific lifecycle points. Defined in `hooks/hooks.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/validate-prerequisites.sh"
          }
        ]
      }
    ]
  }
}
```

| Hook | When it runs |
|---|---|
| `PreToolUse` | Before a tool is invoked |
| `PostToolUse` | After a tool completes |

---

## Providers

Provider files contain platform-specific instructions. Place them in `providers/` as markdown files:

- `providers/github.md` — GitHub-specific instructions
- `providers/azure-devops.md` — Azure DevOps-specific instructions
- `providers/generic.md` — Fallback for other platforms

---

## Styles

Output style definitions in `styles/` control how the plugin formats its output. These are markdown or JSON files that describe the expected structure of results (e.g., review report format).

---

## MCP Configuration

If your plugin needs external tools (APIs, databases, search), configure MCP servers in `.mcp.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

---

## Reference Implementation

The [`pr-reviewer`](https://github.com/xianix-team/xianix-plugins-official/tree/main/plugins/pr-reviewer) plugin is a full reference implementation that uses commands, hooks, providers, and styles. Study it when building a new plugin.
