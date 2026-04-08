---
title: Marketplace Overview
description: The Xianix Plugins Official marketplace — available plugins and how to install them.
---

A curated directory of high-quality plugins for the Xianix agent — AI-powered automation tools for your development lifecycle.

:::caution
Make sure you trust a plugin before installing, updating, or using it. 99x does not control what MCP servers, files, or other software are included in community plugins and cannot verify that they will work as intended or that they won't change.
:::

---

## Available Plugins

| Plugin | Version | Description | Category |
|--------|---------|-------------|----------|
| [pr-reviewer](/plugins/pr-reviewer/overview/) | 1.1.0 | Comprehensive PR review with specialized agents for code quality, security, test coverage, and performance analysis. Works with GitHub, Azure DevOps, Bitbucket, and any git repository. | code-review |
| [req-analyst](/plugins/req-analyst/overview/) | 1.0.0 | Requirement grooming plugin focused on user experience. Analyzes user intent, domain knowledge, competitive context, and workflow to produce well-understood, groomed requirements. | requirements |

---

## Adding This Marketplace

Add this marketplace to the Xianix agent using the `plugin marketplace add` command.

**From GitHub (recommended):**

```
claude plugin marketplace add xianix-team/xianix-plugins-official
```

**Pin to a specific branch or tag:**

```
claude plugin marketplace add xianix-team/xianix-plugins-official@main
```

**From a git URL:**

```
claude plugin marketplace add https://github.com/xianix-team/xianix-plugins-official.git#main
```

**From a remote `marketplace.json` URL:**

```
claude plugin marketplace add https://raw.githubusercontent.com/xianix-team/xianix-plugins-official/main/.claude-plugin/marketplace.json
```

---

## Installing a Plugin

Once the marketplace is added:

```
/plugin install {plugin-name}@xianix-plugins-official
```

Or browse for the plugin via `/plugin > Discover`.

---

## Plugin Structure

Each plugin follows a standard structure:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata (required)
├── .mcp.json            # MCP server configuration (optional)
├── commands/            # Slash commands (optional)
├── agents/              # Agent definitions (optional)
├── skills/              # Skill definitions (optional)
├── hooks/               # Lifecycle hooks (optional)
├── providers/           # Provider-specific configuration (optional)
├── styles/              # Output style definitions (optional)
└── README.md            # Documentation
```

See the [Plugin Development guide](/plugin-development/overview/) for details on building your own plugins.
