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
   - `next` → Next.js (check for `app/` directory = app router, `pages/` = pages router)
   - `@remix-run/react` or `@remix-run/node` → Remix
   - `react` + (`vite` or `react-scripts`) → React SPA
2. **Scope** — infer from the user's prompt:
   - Single file/component mentioned → audit that file only
   - Directory or route mentioned → audit that subtree
   - "whole site", "entire app", "all pages" → full repo crawl
   - No clear scope → ask: "Should I audit a specific page, or the whole project?"
3. **Existing SEO setup** — scan for: `next-seo`, `react-helmet`, `react-helmet-async`, `next-sitemap`. Check for `metadata` exports, `generateMetadata`, `export const meta`, or `<Head>` usage. Also check parent layouts — Next.js metadata inherits from ancestor layouts, so a page without its own `metadata` may still be covered.
4. **i18n setup** — check for `next-intl`, `next-i18next`, `i18next`, or locale-based routing (`[locale]` segments). If present, read `references/i18n-seo.md`.

### Step 2: Audit

Run a tiered audit. Classify every finding as **Critical** or **Enhancement**.

**Critical** — must be fixed in this session. These represent genuine indexing risk or missing baseline SEO:
- Public page with no `<title>` or `meta description` (and none inherited from a parent layout)
- Next.js page/layout with no `metadata` export or `generateMetadata` AND no ancestor layout providing coverage
- Remix route missing `export const meta`
- Informational `<img>` tags missing `alt` attributes (decorative images with `alt=""` are correct — do not flag)
- No canonical URL on pages with real duplicate content risk (parameterized URLs, pagination, syndicated content)
- React SPA on an SEO-critical public site with no acknowledgment of rendering risk
- i18n site missing `hreflang` alternate links

**Enhancement** — recommend + show example, ask before applying:
- Missing Open Graph / Twitter card tags
- Missing JSON-LD structured data
- No sitemap
- Missing `robots.txt` (crawl is permitted by default; absence is not a blocker, but having one is good practice)
- Using `next/head` in app router (native metadata API is preferred, not required)
- Dynamic pages with hardcoded or missing SEO
- `robots.txt` missing sitemap pointer
- Paginated content with no canonical strategy

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

Fix all **Critical** issues. For **Enhancements**, show a code example and ask if they want it applied.

Output format depends on the size of the change, not the severity:
- **Small targeted fix** (adding one `alt`, one missing meta tag): edit directly without preview
- **Structural change** (adding `generateMetadata`, wiring data flow, adding a new export): show the diff first, then apply — don't ask for a separate confirmation, just show what will change before writing it
- **New files** (`sitemap.ts`, `robots.txt`, `seo-audit.md`): create directly

The distinction matters because showing a diff for a one-line change is noise, but silently restructuring a file is surprising. Match the preview to the impact.

---

## Framework-Specific Patterns

- `references/nextjs.md` — Next.js app router, pages router, sitemap, robots.txt
- `references/remix.md` — Remix `meta` export, loader-driven metadata, sitemap resource routes
- `references/react-spa.md` — react-helmet-async setup, rendering risk guidance, static sitemap
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

JSON-LD is injected via a `<script type="application/ld+json">` tag in the **page component**. It cannot be emitted from `generateMetadata` — that function only controls `<head>` meta elements, not arbitrary script tags.

---

## Dynamic SEO — Wiring Data

When a page needs metadata from fetched data:

1. **Read the file** — find the data-fetching pattern:
   - Next.js app router: `generateStaticParams` + `fetch` → add `generateMetadata` using same fetch (Next.js deduplicates)
   - Next.js pages router: `getServerSideProps` / `getStaticProps` → extract data and pass to `<Head>`
   - Remix: `loader` function → read via `data` argument in `meta` function
   - Client component (`useQuery` / `useSWR`): server metadata not available → flag as critical, recommend server component or `react-helmet-async`

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

`rel="next"` / `rel="prev"` link hints were deprecated by Google in 2019 and have no indexing effect — do not recommend them.

Modern pagination guidance:
- Page 1: clean canonical URL, no `?page=1` parameter
- Pages 2+: self-referencing canonical (each page is its own canonical)
- Thin paginated pages with little unique content: consider `noindex, follow`
- Sitemap: include page 1; include deeper pages only if they have genuinely distinct content worth indexing

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

## SPA Rendering Risk

When auditing a React SPA (no SSR) on a site where SEO matters, surface as **Critical**:

```
⚠️  Server-Side Rendering Risk
This is a client-side rendered SPA. Google does crawl and render JavaScript,
but with meaningful caveats: rendering is deferred (pages may be crawled before
JS executes), rendering resources are limited (complex SPAs may time out or
partially render), and dynamic meta tags have weaker indexing guarantees than
server-rendered HTML.

For reliable SEO, server-rendered metadata is the baseline. Options:
  1. Migrate to Next.js — server components, native metadata API, SSG/SSR
  2. Migrate to Remix — lightweight SSR with first-class meta export
  3. Add a prerendering service for the current stack (prerender.io, rendertron)

react-helmet-async improves social sharing previews (Slack, Twitter, iMessage
all execute JS) but does not eliminate the rendering reliability gap for Google.
```

Skip for internal tools, dashboards, and auth-gated apps.

---

## Full SEO Checklist

### Core
- [ ] `<title>` — unique, under 60 chars, includes primary keyword
- [ ] `meta description` — under 160 chars, compelling, matches content
- [ ] `canonical` — present on pages with genuine duplicate content risk
- [ ] `robots` meta — not accidentally `noindex` on public pages

### Social
- [ ] `og:title`, `og:description`, `og:image`, `og:url`, `og:type`
- [ ] `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`

### Structured Data
- [ ] JSON-LD present in page component — schema type matches content

### Images
- [ ] Informational `<img>` have descriptive `alt` text
- [ ] Decorative `<img>` have `alt=""` (empty, not missing)
- [ ] `next/image` considered for performance gains (not an SEO hard requirement)

### Crawlability
- [ ] `sitemap.xml` exists and linked in `robots.txt`
- [ ] `robots.txt` present and not blocking public routes
- [ ] SPA rendering risk acknowledged if SEO-critical

### i18n (if applicable)
- [ ] `hreflang` alternate links for every locale
- [ ] `x-default` set to fallback locale
- [ ] Canonical URLs include locale prefix consistently

### Pagination (if applicable)
- [ ] Page 1 canonical is clean (no `?page=1`)
- [ ] Pages 2+ have self-referencing canonicals
- [ ] Thin paginated pages use `noindex, follow` if appropriate
