# seo-enhancer

An SEO audit and implementation assistant for **Next.js**, **Remix**, and **React SPA** projects. Works with Claude Code, Cursor, GitHub Copilot, Windsurf, Cline, OpenAI Codex, and any AI coding tool.

## What it does

- **Audits** pages for missing meta tags, broken sitemaps, crawlability issues, and more
- **Fixes** critical issues automatically — missing titles, alt tags, metadata exports
- **Recommends** enhancements like JSON-LD structured data, Open Graph tags, and sitemaps
- **Detects** your framework and existing setup — works with what you already have
- **Warns** about React SPA crawlability risks that react-helmet alone can't solve

### Coverage

| Framework | Meta tags | OG/Twitter | JSON-LD | Sitemap | robots.txt | i18n/hreflang |
|-----------|-----------|------------|---------|---------|------------|---------------|
| Next.js (app router) | Full | Full | Full | Full | Full | Full |
| Next.js (pages router) | Full | Full | Full | Full | Full | Full |
| Remix | Full | Full | Full | Full | Full | Full |
| React SPA (Vite/CRA) | Full | Full | Full | Partial | Full | Partial |
| Vue / Nuxt | Contributions welcome | | | | | |
| Gatsby | Contributions welcome | | | | | |

---

## Install

### Universal (all tools)

```bash
npx skills add gashiartim/seo-enhancer
```

The `skills` CLI auto-detects your AI tools (Claude Code, Cursor, Windsurf, Cline, Copilot, Codex, and more) and installs to the right place for each. Works globally or per-project.

Install globally for all your projects:

```bash
npx skills add gashiartim/seo-enhancer -g
```

Install for specific agents only:

```bash
npx skills add gashiartim/seo-enhancer --agent claude-code cursor windsurf
```

---

### Manual setup by tool

If you prefer to install manually, copy the adapter file for your tool:

| Tool | File | Destination |
|------|------|-------------|
| Claude Code (omc) | `SKILL.md` | `~/.claude/skills/seo-enhancer/` |
| Cursor | `adapters/cursor.mdc` | `.cursor/rules/seo-enhancer.mdc` |
| GitHub Copilot | `adapters/copilot.md` | `.github/copilot-instructions.md` |
| Windsurf | `adapters/windsurf.md` | `.windsurfrules` |
| Cline | `adapters/cline.md` | `.clinerules` |
| Codex / AGENTS.md | `adapters/agents.md` | `AGENTS.md` |
| Any other tool | `INSTRUCTIONS.md` | paste into system prompt |

---

## Usage

Once set up, just describe what you need:

```
Audit the SEO on my homepage
Fix the meta tags on my blog post page
My site doesn't show up in Google — what's wrong?
Add structured data to my product pages
Set up a sitemap for my Next.js app
My social previews are broken
Add hreflang to my i18n site
```

---

## Examples

### Next.js — full site audit

```
Audit the entire Next.js project for SEO issues
```

Produces a `seo-audit.md` at the project root with critical issues and enhancements, then fixes all critical ones.

### Remix — add metadata to a route

```
Add SEO meta tags to app/routes/blog.$slug.tsx
```

Reads the existing loader, wires `export const meta` to the same data, adds OG/Twitter tags.

### React SPA — baseline SEO setup

```
Set up SEO for my Vite React app
```

Installs `react-helmet-async`, adds a `HelmetProvider`, wires meta tags on each page, generates `public/robots.txt`, and flags the crawlability risk.

### i18n site — add hreflang

```
My Next.js site uses next-intl with en/fr/de locales — add hreflang
```

Detects locale routing, adds `alternates.languages` with `x-default` to `generateMetadata` for every route.

---

## How it decides what to fix vs. recommend

**Fixes immediately (Critical):**
- Missing `<title>` or `meta description` on a public page
- Next.js page without `generateMetadata` or `metadata` export
- Remix route without `export const meta`
- `<img>` tags missing `alt` attributes
- No `robots.txt`
- i18n site with no hreflang

**Recommends (Enhancement):**
- Open Graph / Twitter card tags
- JSON-LD structured data
- Sitemap generation
- `next/image` migration
- Pagination canonicals

---

## File structure

```
seo-enhancer/
├── INSTRUCTIONS.md             — universal instructions (any AI tool)
├── SKILL.md                    — Claude Code skill (oh-my-claudecode)
├── adapters/
│   ├── cursor.mdc              — Cursor rules (.cursor/rules/)
│   ├── copilot.md              — GitHub Copilot (.github/copilot-instructions.md)
│   ├── windsurf.md             — Windsurf (.windsurfules)
│   ├── cline.md                — Cline (.clinerules)
│   └── agents.md               — OpenAI Codex / AGENTS.md
└── references/
    ├── nextjs.md               — Next.js patterns (app router, pages router, sitemap)
    ├── remix.md                — Remix meta export, loader-driven metadata, resource routes
    ├── react-spa.md            — react-helmet-async setup, crawlability warning
    ├── json-ld-schemas.md      — JSON-LD templates (Article, Product, FAQ, Event, etc.)
    └── i18n-seo.md             — hreflang, x-default, next-intl, locale-aware sitemaps
```

---

## Contributing

Contributions welcome. The most valuable additions:

- **Vue / Nuxt support** — add `references/nuxt.md` + `adapters/` variant, update framework detection
- **Gatsby support** — add `references/gatsby.md` with Gatsby Head API patterns
- **New JSON-LD schemas** — add templates to `references/json-ld-schemas.md` (Course, JobPosting, Review, etc.)
- **New tool adapters** — add to `adapters/` following the existing format

### Adding a new framework

1. Add `references/<framework>.md` with implementation patterns
2. Update framework detection in `SKILL.md` → Step 1 and `INSTRUCTIONS.md`
3. Update all adapter files with the new framework's patterns
4. Update the coverage table in this README

---

## License

MIT

---

Made by [Artim Gashi](https://www.artimgashi.dev)
