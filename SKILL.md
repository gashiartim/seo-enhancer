---
name: seo-enhancer
description: >
  SEO audit and implementation specialist for Next.js, Remix, and React SPAs. Use this skill whenever the user mentions SEO, meta tags, structured data, sitemaps, robots.txt, Open Graph, Twitter cards, JSON-LD, hreflang, image alt attributes, or crawlability. Also trigger when the user says their site doesn't rank, Google can't find their pages, social previews look wrong, they want better discoverability, or they're working on a public-facing Next.js, Remix, or React page. Proactively suggest this skill when reviewing Next.js or Remix page files that are missing metadata exports or head SEO tags — even if the user didn't ask about SEO directly.
---

# SEO Enhancer

Audit and fix SEO for **Next.js**, **Remix**, and **React SPAs**. Covers: meta tags, Open Graph, Twitter cards, JSON-LD structured data, sitemaps, robots.txt, image SEO, i18n/hreflang, and pagination SEO.

## What this skill covers

| Area | Supported |
|------|-----------|
| Next.js (app router + pages router) | Full |
| Remix | Full |
| React SPA (Vite, CRA) | Full |
| Vue / Nuxt | Not yet — contributions welcome |
| Gatsby | Not yet — contributions welcome |
| Core Web Vitals / performance | Out of scope |
| Content strategy / keyword research | Out of scope |

---

## Workflow

### Step 1: Detect Context

Before doing anything, establish:

1. **Framework** — read `package.json`:
   - `next` → Next.js (check version for app vs pages router)
   - `@remix-run/react` or `@remix-run/node` → Remix
   - `react` + (`vite` or `react-scripts`) → React SPA
2. **Scope** — infer from the user's prompt:
   - Single file/component mentioned → audit that file only
   - Directory or route mentioned → audit that subtree
   - "whole site", "entire app", "all pages" → full repo crawl
   - No clear scope → ask: "Should I audit a specific page, or the whole project?"
3. **Existing SEO setup** — scan for: `next-seo`, `react-helmet`, `react-helmet-async`, `next-sitemap`, `@nasa-gcn/remix-seo`. Check for `metadata` exports, `generateMetadata`, `export const meta`, or `<Head>` usage.
4. **i18n setup** — check for `next-intl`, `next-i18next`, `i18next`, or locale-based routing (`[locale]` segments). If present, read `references/i18n-seo.md`.

### Step 2: Audit

Run a tiered audit. Classify every finding as **Critical** or **Enhancement**.

**Critical** (fix immediately, no confirmation needed):
- Public page missing `<title>` or `meta description`
- Next.js page/layout missing `metadata` export or `generateMetadata`
- Remix route missing `export const meta`
- `<img>` tags missing `alt` attributes
- No `robots.txt` in a production app
- No canonical URL on pages with duplicate content risk
- React SPA (no SSR) without a crawlability warning on an SEO-critical site
- i18n site missing `hreflang` alternate links

**Enhancement** (recommend + show example, let user decide):
- Missing Open Graph / Twitter card tags
- Missing JSON-LD structured data
- No sitemap
- Using `next/head` in app router (should use metadata API)
- `next/image` not used where `<img>` exists
- Dynamic pages with hardcoded or missing SEO (title doesn't reflect content)
- `robots.txt` exists but missing sitemap pointer
- Paginated lists missing canonical or rel=next/prev handling

### Step 3: Report

**Single file/component**: Report findings inline in conversation. No file written.

**Full site audit**: Write `seo-audit.md` at the project root:

```markdown
# SEO Audit — [Project Name]
Generated: [date]
Framework: [Next.js / Remix / React SPA]

## Summary
- X critical issues
- Y enhancements

## Critical Issues
### [Issue title] — `path/to/file.tsx`
[What's wrong and why it matters for indexing/ranking]
[Fix applied / Fix: see below]

## Enhancements
### [Issue title] — `path/to/file.tsx`
[Recommendation + example snippet]
```

### Step 4: Implement Fixes

**Output format rules:**
- **Small targeted fix** (adding `alt`, one meta tag): edit directly, no preview
- **Structural rewrite** (adding `generateMetadata`, restructuring metadata flow): show diff first, apply after approval
- **New files** (`sitemap.ts`, `robots.txt`, `seo-audit.md`): create directly

Fix all **Critical** issues. For **Enhancements**, show a code example and ask if they want it applied.

---

## Framework-Specific Patterns

- `references/nextjs.md` — Next.js app router, pages router, sitemap, robots.txt, next/image
- `references/remix.md` — Remix `meta` export, loader-driven metadata, sitemap resource routes
- `references/react-spa.md` — react-helmet-async setup, crawlability warning, static sitemap
- `references/json-ld-schemas.md` — JSON-LD templates (Article, Product, FAQ, Org, Event, etc.)
- `references/i18n-seo.md` — hreflang, locale-aware metadata, next-intl patterns

---

## JSON-LD Schema Detection

Auto-detect schema type from route pattern, file name, and component content:

| Signal | Schema |
|--------|--------|
| `/blog/[slug]`, `BlogPost`, `ArticleDetail` | `Article` |
| `/products/[id]`, `ProductPage`, `product.name` | `Product` |
| `/faq`, `FAQSection`, accordion with Q&A | `FAQPage` |
| `/`, `HomePage`, root layout | `Organization` + `WebSite` |
| `/[...slug]`, catch-all, ambiguous | Ask user |
| `/about`, `/contact` | `WebPage` + `Organization` |
| `/recipes/[slug]`, `RecipeCard` | `Recipe` |
| `/events/[id]`, `EventDetail` | `Event` |

When genuinely ambiguous, ask: "What type of content is on this page? (article, product, FAQ, event, etc.)"

---

## Dynamic SEO — Wiring Data

When a page needs metadata from fetched data:

1. **Read the file** — find the data-fetching pattern:
   - Next.js app router: `generateStaticParams` + `fetch` → add `generateMetadata` using same fetch (Next.js deduplicates)
   - Next.js pages router: `getServerSideProps` / `getStaticProps` → extract data and pass to `<Head>`
   - Remix: `loader` function → read `useLoaderData()` in `meta` function via `data` argument
   - Client component (`useQuery` / `useSWR`): can't use server metadata → flag as critical, recommend server component or `react-helmet-async`

2. **Wire the metadata** — use the same data source, don't introduce a second fetch.

3. **If data source is unclear** — ask: "Where does this page's content come from? (API, database, CMS?)"

---

## i18n SEO

When the project uses locale-based routing or an i18n library, read `references/i18n-seo.md` for:
- `hreflang` alternate links (prevents duplicate content penalties across locales)
- `x-default` fallback locale
- Locale-aware canonical URLs
- `alternates.languages` in Next.js metadata API
- next-intl integration patterns

---

## Pagination SEO

For paginated lists (`/blog?page=2`, `/products/page/3`):
- Page 1 gets the canonical URL — no `?page=1` parameter
- Pages 2+ get `rel="canonical"` pointing to themselves (not page 1)
- Add `<meta name="robots" content="noindex, follow">` on pages 2+ if content is thin
- Structured data (`ItemList`) on the first page only
- Sitemap should include only page 1 (or all pages if content is unique per page)

---

## Library Selection

Check `package.json` first. Then:

| Situation | Use |
|-----------|-----|
| Next.js app router | Native `metadata` export / `generateMetadata` |
| Next.js pages router | `next/head` or `next-seo` if already installed |
| Remix | Native `export const meta` — no library needed |
| React SPA, nothing installed | Install `react-helmet-async` |
| React SPA, `react-helmet` installed | Upgrade to `react-helmet-async` (thread-safe) |
| Sitemap needed (Next.js) | `next-sitemap` or native `app/sitemap.ts` |
| Sitemap needed (Remix) | Resource route returning XML |
| Sitemap needed (SPA) | Build-time generation script |
| `next-seo` already installed | Use it — don't switch to native unless user asks |

Never introduce a new package when a native API or existing dep covers the need.

---

## SPA Crawlability Warning

When auditing a React SPA (no SSR), check: is this site SEO-critical? If yes, surface as **Critical**:

```
⚠️  Crawlability Risk Detected
This is a client-side rendered SPA. Googlebot may not execute JavaScript,
meaning meta tags set via react-helmet-async may not be indexed.

To fully solve this, consider:
  1. Migrating to Next.js (recommended — full SSR/SSG, native metadata API)
  2. Migrating to Remix (lightweight SSR, excellent meta API)
  3. Adding a prerendering service (prerender.io, rendertron)

react-helmet-async has been added for meta tag management. This improves
social sharing previews (Slack, Twitter) which do execute JS, but does not
guarantee Google indexing of dynamic meta tags.
```

Skip for internal tools, dashboards, and auth-gated apps.

---

## Full SEO Checklist

### Core
- [ ] `<title>` — unique, under 60 chars, includes primary keyword
- [ ] `meta description` — under 160 chars, compelling, matches content
- [ ] `canonical` — present on pages with duplicate content risk
- [ ] `robots` meta — not accidentally `noindex` on public pages

### Social
- [ ] `og:title`, `og:description`, `og:image`, `og:url`, `og:type`
- [ ] `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`

### Structured Data
- [ ] JSON-LD present — schema type matches page content

### Images
- [ ] All `<img>` have meaningful `alt` (not empty, not "image")
- [ ] `next/image` used in Next.js (not raw `<img>`)

### Crawlability
- [ ] `sitemap.xml` exists and is linked in `robots.txt`
- [ ] `robots.txt` at root, not blocking public routes
- [ ] SPA crawlability warning shown if SEO-critical

### i18n (if applicable)
- [ ] `hreflang` alternate links for every locale
- [ ] `x-default` set to fallback locale
- [ ] Canonical URLs include locale prefix consistently

### Pagination (if applicable)
- [ ] Page 1 canonical is clean (no `?page=1`)
- [ ] Pages 2+ have self-referencing canonicals
- [ ] Sitemap covers correct page set
