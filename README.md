# cartershades.com

Personal website for Carter Shades. Built with [Astro](https://astro.build).

## Local development

```sh
npm install
npm run dev      # http://localhost:4321
npm run build    # output to ./dist
npm run preview  # preview the production build locally
```

## Project structure

```
.
├── public/
│   └── CNAME                # GitHub Pages custom domain
├── src/
│   ├── components/          # Header, Footer, Nav — shared across pages
│   ├── layouts/
│   │   └── BaseLayout.astro # the single layout every page wraps in
│   ├── pages/               # one file per route (index, work, writing, about)
│   ├── config/
│   │   └── site.ts          # site metadata + nav links (single source of truth)
│   ├── content/
│   │   └── writing/         # markdown posts (empty for now)
│   └── content.config.ts    # content collections schema
├── astro.config.mjs
├── tsconfig.json
└── CLAUDE.md                # navigation guide for AI agents
```

## Adding a page

1. Create `src/pages/<slug>.astro` and wrap content in `<BaseLayout title="…">`.
2. Add an entry to `navLinks` in [src/config/site.ts](src/config/site.ts).

## Adding a writing post

Drop a markdown file into `src/content/writing/` with frontmatter:

```md
---
title: "Post title"
pubDate: 2026-01-01
description: "Optional summary"
---
```

The schema is defined in [src/content.config.ts](src/content.config.ts).

## Deployment

Static build deployed via GitHub Pages. The `public/CNAME` file binds the
deployed site to `www.cartershades.com`.
