---
title: Plugin Structure
description: How Xianix plugins map to the Claude Code plugin format and where to find detailed development guidance.
---

Xianix plugins are standard [Claude Code plugins](https://code.claude.com/docs/en/plugins) — self-contained directories that extend what the agent can do when it runs against a repository. Because the format is defined by Claude Code, the upstream documentation is the authoritative reference for building plugins.

---

## Quick Reference

A minimal plugin looks like this:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata (required)
├── commands/            # Slash commands
├── skills/              # Skill definitions
├── hooks/               # Lifecycle hooks
├── providers/           # Provider-specific configuration
└── README.md
```

For the full specification of each directory, manifest fields, and available extension points, see the official docs:

- [Plugin development guide](https://code.claude.com/docs/en/plugins) — creating, testing, and distributing plugins
- [Plugins reference](https://code.claude.com/docs/en/plugins-reference) — detailed schema and field descriptions

---

## Xianix-Specific Conventions

While the plugin format itself is standard Claude Code, Xianix plugins follow a few additional conventions:

| Convention | Detail |
|---|---|
| **Providers** | Place platform-specific instructions in `providers/github.md`, `providers/azure-devops.md`, etc. The executor selects the correct provider based on the configured platform. |
| **Output Styles** | Define output formatting templates in `styles/` to control how the plugin formats results (e.g., review report structure). |

---

## Where to Host Your Plugin

You can develop and host plugins **anywhere** — the plugin does not need to live in the official Xianix marketplace. Any git repository (public or private) that follows the Claude Code plugin structure works.

To use a plugin hosted in your own repository, point to it via the [`use-plugins`](/agent-configuration/rules/#4-use-plugins--plugin-installation) section in your `rules.json`:

```json
"use-plugins": [
  {
    "plugin-name": "my-plugin@my-marketplace",
    "marketplace": "your-org/your-plugins-repo"
  }
]
```

If you'd like to contribute to the **official** Xianix plugin collection, submit a pull request to [`xianix-team/plugins-official`](https://github.com/xianix-team/plugins-official). See the [Marketplace](/plugin-development/marketplace/) page for details.

---

## Reference Implementation

The [`pr-reviewer`](https://github.com/xianix-team/plugins-official/tree/main/plugins/pr-reviewer) plugin is a full reference implementation that uses commands, hooks, providers, and styles. Study it when building a new plugin.
