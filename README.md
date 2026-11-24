# ReviewForest Skills

Display your [ReviewForest](https://reviewforest.org) reviews on your website. Works with Claude Code, Cursor, Codex, Copilot, and 35+ AI coding agents.

## Install

```bash
npx skills add reviewforest/skills
```

## reviewforest-widgets

Show your ReviewForest reviews, star ratings, and tree-planting stats on any website. Two approaches:

- **Widget embed (recommended)** — Create and customize a widget in the [ReviewForest dashboard](https://app.reviewforest.org), then add a small code snippet to your site. 8 widget types available (testimonials carousel, review list, review score badge, tree counter badge, and more). Best for most use cases.
- **Custom rendering via API** — Fetch review data from the ReviewForest API and render it in your own framework (React, Vue, HTML, etc.). Full styling control, no external JavaScript. Best when you need a fully custom design.

**What you need:** A ReviewForest account at [app.reviewforest.org](https://app.reviewforest.org). For widget embed — a widget UUID. For custom rendering — a widget UUID too; an API key only for extra forest data, used on your server or at build time.

## Links

- [ReviewForest](https://reviewforest.org)
- [Help Center](https://help.reviewforest.org)
- [Agent Skills Specification](https://agentskills.io)
