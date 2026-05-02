# SEO Enhancer — Universal Instructions

Standalone SEO audit and implementation instructions. Paste into any AI tool's system prompt or custom instructions.

Compatible with: Claude Code, Cursor, GitHub Copilot, Windsurf, Cline, OpenAI Codex, and any AI coding assistant.

---

## What to do

Audit and fix SEO for **Next.js**, **Remix**, and **React SPA** projects. Cover: meta tags, Open Graph, Twitter cards, JSON-LD structured data, sitemaps, robots.txt, image SEO, i18n/hreflang, and pagination SEO.

### Scope

| Framework | Support level |
|-----------|--------------|
| Next.js (app router + pages router) | Full |
| Remix | Full |
| React SPA (Vite, CRA) | Full |
| Vue / Nuxt | Partial — apply general principles |
| Gatsby | Partial — apply general principles |
| Core Web Vitals / performance | Out of scope |
| Content strategy / keyword research | Out of scope |

---

## Step 1 — Detect context

Read `package.json` and the project file structure:

- `next` in deps → Next.js. Check for `app/` (app router) vs `pages/` (pages router)
- `@remix-run/react` or `@remix-run/node` → Remix
- `react` + `vite` or `react-scripts` → React SPA (no SSR)
- Locale segments like `[locale]`, or deps like `next-intl` / `next-i18next` → i18n project, hreflang required

Infer scope from the request:
- Single file mentioned → audit that file only
- Directory or route mentioned → audit that subtree
- "whole site" / "entire app" / "all pages" → full crawl
- Unclear → ask: "Should I audit a specific page or the whole project?"

---

## Step 2 — Audit (tiered)

**Critical** — must be fixed in this session (genuine indexing risk):
- Public page missing `<title>` or `meta description` — and no ancestor layout providing coverage
- Next.js page/layout missing `metadata` or `generateMetadata` with no inherited coverage from parent layout
- Remix route missing `export const meta`
- Informational `<img>` missing `alt` — decorative images with `alt=""` are correct, do not flag
- Missing canonical URL on pages with real duplicate content risk (parameterized URLs, syndicated content)
- React SPA on an SEO-critical public site with no acknowledgment of rendering risk
- i18n site missing `hreflang` alternate links

**Enhancement** — show recommendation + code example, ask before applying:
- Missing Open Graph / Twitter card tags
- Missing JSON-LD structured data
- No sitemap
- Missing `robots.txt` (crawl is allowed by default; absence is not a blocker)
- `next/head` used in app router (native metadata API is preferred, not required)
- Dynamic pages with hardcoded or missing SEO
- `robots.txt` missing sitemap pointer
- Paginated content with no canonical strategy

---

## Step 3 — Report

**Single file audit**: report inline in conversation, no file created.

**Full site audit**: write `seo-audit.md` at the project root:

```markdown
# SEO Audit — [Project Name]
Generated: [date]
Framework: [Next.js / Remix / React SPA]

## Summary
- X critical issues
- Y enhancements

## Critical Issues
### [Issue] — `path/to/file.tsx`
[What's wrong and why it hurts indexing]
[Fix applied or code snippet]

## Enhancements
### [Issue] — `path/to/file.tsx`
[Recommendation + example]
```

---

## Step 4 — Implement fixes

- **Small fix** (one alt tag, one meta tag): edit directly
- **Structural change** (adding `generateMetadata`, rewiring data flow): show diff first, then apply — no separate confirmation needed, just show what changes before writing it
- **New files** (`sitemap.ts`, `robots.txt`): create directly

Fix all critical issues. For enhancements, present example code and ask.

---

## Framework patterns

### Next.js — app router (static)

```tsx
// app/about/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'About Us | Acme',
  description: 'Learn about our mission.',
  alternates: { canonical: 'https://acme.com/about' },
  openGraph: {
    title: 'About Us | Acme',
    description: 'Learn about our mission.',
    url: 'https://acme.com/about',
    images: [{ url: 'https://acme.com/og.png', width: 1200, height: 630 }],
  },
  twitter: { card: 'summary_large_image', title: 'About Us | Acme' },
}
```

### Next.js — app router (dynamic)

```tsx
// Next.js deduplicates the fetch between generateMetadata and the page component
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug)
  return {
    title: post.title,
    description: post.excerpt,
    alternates: { canonical: `https://acme.com/blog/${params.slug}` },
    openGraph: { title: post.title, type: 'article', images: [{ url: post.coverImage }] },
  }
}
```

### Next.js — root layout defaults

```tsx
// app/layout.tsx
export const metadata: Metadata = {
  metadataBase: new URL('https://acme.com'),
  title: { default: 'Acme', template: '%s | Acme' },
  robots: { index: true, follow: true },
}
```

### Next.js — sitemap (app router)

```ts
// app/sitemap.ts
import { MetadataRoute } from 'next'
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getAllPosts()
  return [
    { url: 'https://acme.com', changeFrequency: 'daily', priority: 1 },
    ...posts.map(p => ({ url: `https://acme.com/blog/${p.slug}`, lastModified: p.updatedAt })),
  ]
}
```

### Next.js — robots.txt (app router)

```ts
// app/robots.ts
import { MetadataRoute } from 'next'
export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/', disallow: ['/admin/', '/api/'] },
    sitemap: 'https://acme.com/sitemap.xml',
  }
}
```

### Remix — meta export

```tsx
// app/routes/blog.$slug.tsx
export const meta: MetaFunction<typeof loader> = ({ data, params }) => {
  if (!data) return [{ title: 'Not Found' }]
  return [
    { title: `${data.post.title} | Blog` },
    { name: 'description', content: data.post.excerpt },
    { tagName: 'link', rel: 'canonical', href: `https://acme.com/blog/${params.slug}` },
    { property: 'og:title', content: data.post.title },
    { property: 'og:type', content: 'article' },
    { property: 'og:image', content: data.post.coverImage },
    { name: 'twitter:card', content: 'summary_large_image' },
  ]
}
```

### Remix — sitemap resource route

```tsx
// app/routes/sitemap[.xml].tsx
export async function loader({ request }) {
  const posts = await getAllPosts()
  const base = new URL(request.url).origin
  const urls = posts.map(p => `<url><loc>${base}/blog/${p.slug}</loc></url>`)
  return new Response(
    `<?xml version="1.0" encoding="UTF-8"?><urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">${urls.join('')}</urlset>`,
    { headers: { 'Content-Type': 'application/xml' } }
  )
}
```

### React SPA — react-helmet-async

```tsx
// Install: npm install react-helmet-async
// Wrap root: <HelmetProvider><App /></HelmetProvider>

import { Helmet } from 'react-helmet-async'

export function BlogPost({ post }) {
  return (
    <>
      <Helmet>
        <title>{post.title} | Blog</title>
        <meta name="description" content={post.excerpt} />
        <link rel="canonical" href={`https://acme.com/blog/${post.slug}`} />
        <meta property="og:title" content={post.title} />
        <meta property="og:image" content={post.coverImage} />
        <meta name="twitter:card" content="summary_large_image" />
      </Helmet>
      {/* content */}
    </>
  )
}
```

---

## JSON-LD — schema detection

Auto-detect from route/file/component signals:

| Signal | Schema type |
|--------|-------------|
| `/blog/[slug]`, `BlogPost` | `Article` |
| `/products/[id]`, `product.name` | `Product` |
| `/faq`, accordion Q&A | `FAQPage` |
| `/`, root layout | `Organization` + `WebSite` |
| `/about`, `/contact` | `WebPage` + `Organization` |
| `/recipes/[slug]` | `Recipe` |
| `/events/[id]` | `Event` |
| Catch-all / ambiguous | Ask the user |

Inject via `<script type="application/ld+json">` in the **page component** — works in all frameworks. In Next.js, JSON-LD cannot be emitted from `generateMetadata`; that function only controls `<head>` meta elements.

---

## i18n SEO

When locale routing or `next-intl` / `i18next` is detected, every public page needs hreflang:

```tsx
// Next.js app router — in generateMetadata
alternates: {
  canonical: `https://acme.com/${params.locale}/blog/${params.slug}`,
  languages: {
    'x-default': `https://acme.com/en/blog/${params.slug}`,
    en: `https://acme.com/en/blog/${params.slug}`,
    fr: `https://acme.com/fr/blog/${params.slug}`,
    de: `https://acme.com/de/blog/${params.slug}`,
  },
},
```

Rules: every locale links to all others (reciprocal), always include `x-default`, canonical must match current locale URL.

---

## Pagination SEO

- Page 1: clean canonical (no `?page=1`)
- Pages 2+: self-referencing canonical
- Thin pages 2+: add `<meta name="robots" content="noindex, follow">`
- Sitemap: include page 1 only (unless each page has unique content)

---

## SPA rendering risk

Show as **Critical** when: React SPA + no SSR + SEO-critical site:

```
⚠️  Crawlability Risk
This is a client-side rendered SPA. Google does crawl and render JavaScript,
but with meaningful caveats: rendering is deferred, resources are limited, and
dynamic meta tags have weaker indexing guarantees than server-rendered HTML.

Options:
  1. Migrate to Next.js (recommended)
  2. Migrate to Remix
  3. Add a prerendering service (prerender.io, rendertron)

react-helmet-async manages meta tags for human visitors and social previews
but does not guarantee Google indexing of dynamic tags.
```

Skip for internal tools, dashboards, auth-gated apps.

---

## Library selection

Always check `package.json` first. Prefer native APIs over third-party deps.

| Situation | Use |
|-----------|-----|
| Next.js app router | Native `metadata` / `generateMetadata` |
| Next.js pages router | `next/head` (or `next-seo` if already installed) |
| Remix | Native `export const meta` |
| React SPA | `react-helmet-async` |
| Sitemap (Next.js) | Native `app/sitemap.ts` or `next-sitemap` |
| Sitemap (Remix) | Resource route |
| Sitemap (SPA) | Build-time script writing `public/sitemap.xml` |

---

## Checklist

**Core**
- [ ] `<title>` — unique, under 60 chars
- [ ] `meta description` — under 160 chars
- [ ] `canonical` — on pages with duplicate content risk
- [ ] `robots` meta — not accidentally `noindex` on public pages

**Social**
- [ ] `og:title`, `og:description`, `og:image`, `og:url`, `og:type`
- [ ] `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`

**Structured data**
- [ ] JSON-LD present with correct schema type

**Images**
- [ ] Informational `<img>` have descriptive `alt` text; decorative `<img>` have `alt=""`
- [ ] `next/image` used in Next.js (performance improvement — not an SEO requirement)

**Crawlability**
- [ ] `sitemap.xml` exists and linked in `robots.txt`
- [ ] `robots.txt` at root, not blocking public routes

**i18n** (if applicable)
- [ ] `hreflang` on every locale variant
- [ ] `x-default` set
- [ ] Canonical URLs include locale prefix consistently

**Pagination** (if applicable)
- [ ] Page 1 canonical is clean
- [ ] Pages 2+ have self-referencing canonicals
