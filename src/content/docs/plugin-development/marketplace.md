---
title: Marketplace
description: How to publish your plugin to the Xianix marketplace and contribute to the official registry.
---

The Xianix plugin marketplace is a git repository containing a `marketplace.json` index and one directory per plugin. Users add your marketplace to their agent with a single command, after which they can discover and install your plugins.

---

## `marketplace.json`

The marketplace index lists all available plugins. Place it at the repository root or at `.claude-plugin/marketplace.json`:

```json
{
  "name": "my-plugin-marketplace",
  "description": "Plugins by Example Corp",
  "plugins": [
    {
      "name": "my-plugin",
      "version": "1.0.0",
      "description": "A brief description.",
      "url": "my-plugin@my-marketplace",
      "path": "./plugins/my-plugin"
    }
  ]
}
```

---

## Publishing Your Own Marketplace

1. Create a GitHub repository (e.g. `your-org/my-plugins`).
2. Add a `marketplace.json` at the root.
3. Add your plugin directories under `plugins/`.
4. Users can add your marketplace with:

```
claude plugin marketplace add your-org/my-plugins
```

---

## Contributing to the Official Marketplace

The official Xianix marketplace lives at [`xianix-team/xianix-plugins-official`](https://github.com/xianix-team/xianix-plugins-official).

### Official Plugins

Official plugins are developed and maintained by the 99x / Xianix team. See [`/plugins/pr-reviewer`](https://github.com/xianix-team/xianix-plugins-official/tree/main/plugins/pr-reviewer) for a reference implementation.

### Third-Party Plugins

Third-party partners can submit plugins for inclusion in the marketplace. Submitted plugins must meet quality and security standards for approval.

To submit a new plugin, open an issue or pull request against the [`xianix-plugins-official`](https://github.com/xianix-team/xianix-plugins-official) repository.

**Submission checklist:**

- [ ] `plugin.json` with all required fields
- [ ] `README.md` documenting prerequisites, usage, and configuration
- [ ] At least one command or skill defined
- [ ] License file included
- [ ] Plugin tested locally with Claude Code

---

## Versioning

Plugins use [semver](https://semver.org/). The `version` field in `plugin.json` should be bumped with each release:

| Change type | Version bump |
|---|---|
| New feature (backward compatible) | Minor (`1.0.0` → `1.1.0`) |
| Bug fix | Patch (`1.0.0` → `1.0.1`) |
| Breaking change | Major (`1.0.0` → `2.0.0`) |

---

## Installing from a Marketplace

Once a marketplace is registered:

```bash
# Install a specific plugin
/plugin install my-plugin@my-marketplace

# Or browse and discover
/plugin discover
```

To pin to a specific version, append `@version` after the plugin name when supported by your agent version.
