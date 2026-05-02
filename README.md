# seo-enhancer

A Claude Code skill that audits and fixes SEO in **Next.js**, **Remix**, and **React SPA** projects.

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

### Option 1 — oh-my-claudecode

```bash
omc skill install seo-enhancer
```

### Option 2 — Manual

Clone this repo into your Claude skills directory:

```bash
git clone https://github.com/gashiartim/seo-enhancer ~/.claude/skills/seo-enhancer
```

### Option 3 — .skill file

Download `seo-enhancer.skill` from the [releases page](https://github.com/gashiartim/seo-enhancer/releases) and run:

```bash
omc skill install ./seo-enhancer.skill
```

---

## Usage

The skill triggers automatically. Just describe what you need:

```
Audit the SEO on my homepage
Fix the meta tags on my blog post page
My site doesn't show up in Google — what's wrong?
Add structured data to my product pages
Set up a sitemap for my Next.js app
My social previews are broken
Add hreflang to my i18n site
```

Or open any Next.js/Remix page file and Claude will proactively flag missing metadata.

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
├── SKILL.md                    — main skill logic and checklist
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

- **Vue / Nuxt support** — add `references/nuxt.md` with Nuxt's `useHead` / `useSeoMeta` patterns and update the framework detection in `SKILL.md`
- **Gatsby support** — add `references/gatsby.md` with `gatsby-plugin-react-helmet` or Gatsby Head API patterns
- **New JSON-LD schemas** — add templates to `references/json-ld-schemas.md` (Course, JobPosting, Review, etc.)
- **Edge cases and fixes** — open an issue or PR

### How skills work

A skill is a markdown file (`SKILL.md`) with YAML frontmatter that Claude Code loads into context. The `description` field controls when it triggers. Reference files in `references/` are loaded on demand. See the [oh-my-claudecode docs](https://github.com/oh-my-claudecode/oh-my-claudecode) for the full skill authoring guide.

### Adding a new framework

1. Add `references/<framework>.md` with implementation patterns
2. Update the framework detection table in `SKILL.md` → Step 1
3. Add the framework to the `description` frontmatter
4. Add a row to the Library Selection table
5. Update the coverage table in this README

---

## License

MIT
