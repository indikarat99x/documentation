# Xianix Documentation

Documentation site for Xianix, built with [Starlight](https://starlight.astro.build/).

## Local Development

**Prerequisites:** Node.js 18+

Install dependencies:

```bash
npm install
```

Start the dev server:

```bash
npm run dev
```

The site will be available at [http://localhost:4321](http://localhost:4321). Pages hot-reload as you edit content in `src/content/docs/`.

## Build

```bash
npm run build
```

The static output is written to `dist/`. Preview the built site locally with:

```bash
npm run preview
```

## Content Structure

```
src/content/docs/
├── index.mdx                        # Homepage
├── agent/                           # The Agent docs
│   ├── overview.md
│   ├── architecture.md
│   ├── setup.md
│   ├── rules.md
│   ├── executor.md
│   ├── tenant-isolation.md
│   └── azure-deployment.md
├── plugins/                         # Plugin docs
│   ├── overview.md
│   ├── pr-reviewer/
│   └── req-analyst/
└── plugin-development/              # Building plugins
    ├── overview.md
    └── marketplace.md
```

Add new pages by creating `.md` or `.mdx` files in `src/content/docs/`. Update the sidebar in `astro.config.mjs` to include them in the navigation.
