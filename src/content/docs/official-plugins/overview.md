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

| Plugin | Description |
|--------|-------------|
| [PR Reviewer](/official-plugins/pr-reviewer/) | Parallel code-quality, security, test, and performance review — posted straight to your PR. |
| [Requirement Analyst](/official-plugins/req-analyst/) | Multi-phase requirement grooming: intent, domain, journeys, personas, and gap analysis. |

---

## Adding This Marketplace

```bash
claude plugin marketplace add xianix-team/xianix-plugins-official
```

Pin to a branch or tag:

```bash
claude plugin marketplace add xianix-team/xianix-plugins-official@main
```

---

## Installing a Plugin

Once the marketplace is added:

```bash
/plugin install {plugin-name}@xianix-plugins-official
```

Or browse via `/plugin > Discover`.
