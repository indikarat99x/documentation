---
title: Marketplace
description: How to publish your plugin to the Xianix marketplace and contribute to the official registry.
---

Plugin marketplaces are git repositories that host one or more plugins and expose them through a `marketplace.json` index. Claude Code has built-in support for discovering and installing plugins from marketplaces — see the [official marketplace documentation](https://code.claude.com/docs/en/plugin-marketplaces) for the full specification.

---

## Official Xianix Marketplace

The official Xianix marketplace lives at [`xianix-team/plugins-official`](https://github.com/xianix-team/plugins-official).

- **Official plugins** are developed and maintained by the 99x / Xianix team.
- **Third-party plugins** can be submitted for inclusion via a pull request against the repository.

### Submission Checklist

- [ ] `plugin.json` with all required fields
- [ ] `README.md` documenting prerequisites, usage, and configuration
- [ ] At least one command or skill defined
- [ ] License file included
- [ ] Plugin tested locally with Claude Code

---

## Using Your Own Marketplace

You don't need to publish to the official marketplace. Any git repository that follows the marketplace structure works — just reference it in your [`rules.json`](/agent-configuration/rules/#4-use-plugins--plugin-installation):

```json
"use-plugins": [
  {
    "plugin-name": "my-plugin@my-marketplace",
    "marketplace": "your-org/your-plugins-repo"
  }
]
```

For the full marketplace format (`marketplace.json` schema, versioning, and distribution), refer to the [Claude Code plugin marketplaces docs](https://code.claude.com/docs/en/plugin-marketplaces).
