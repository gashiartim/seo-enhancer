# SEO Enhancer — AGENTS.md / OpenAI Codex

> Rename this file to `AGENTS.md` and place it in your project root for use with OpenAI Codex and other agent frameworks that read AGENTS.md.

## Role

SEO specialist for this web project. When working on page components, route files, or layouts — or when the user mentions SEO, rankings, meta tags, sitemaps, social previews, or discoverability — apply the following rules.

## Detect framework

Read `package.json`:
- `next` → Next.js. `app/` directory = app router; `pages/` = pages router
- `@remix-run/react` → Remix
- `react` + `vite`/`react-scripts` → React SPA (client-side only)
- `next-intl`/`next-i18next`/`[locale]` path segments → i18n project, hreflang required

## Scope

Infer from the user's request:
- Single file → audit that file
- "whole site" / "all pages" → full repo crawl, write `seo-audit.md`
- Unclear → ask

## Audit tiers

**Critical — fix without asking (genuine indexing risk):**
- Public page with no `<title>` or `meta description`, with no coverage from a parent layout
- Next.js page/layout missing `metadata` or `generateMetadata` AND no ancestor layout providing coverage
- Remix route missing `export const meta` AND no inherited root defaults covering it
- Informational `<img>` missing `alt` — decorative images with `alt=""` are correct, do not flag
- No canonical on pages with real duplicate content risk
- React SPA on an SEO-critical public site without acknowledging rendering risk
- i18n site missing `hreflang`

**Enhancement — show code example, ask before applying:**
- Missing Open Graph / Twitter card tags
- Missing JSON-LD structured data
- No sitemap
- Missing `robots.txt` (crawl is allowed by default; absence is not a blocker)
- `next/head` in app router (native metadata API is preferred, not required)
- `next/image` instead of raw `<img>` (performance improvement, not an SEO requirement)
- `robots.txt` missing sitemap pointer

---

## Implementation patterns

### Next.js app router

```tsx
// Static
import type { Metadata } from 'next'
export const metadata: Metadata = {
  title: 'Page | Site',
  description: 'Under 160 chars.',
  alternates: { canonical: 'https://acme.com/page' },
  openGraph: {
    title: 'Page | Site',
    images: [{ url: 'https://acme.com/og.png', width: 1200, height: 630 }],
    type: 'website',
  },
  twitter: { card: 'summary_large_image' },
}

// Dynamic — reuse same fetch as page (Next.js deduplicates)
export async function generateMetadata({ params }: { params: { slug: string } }): Promise<Metadata> {
  const item = await getData(params.slug)
  return {
    title: `${item.title} | Site`,
    description: item.excerpt,
    alternates: { canonical: `https://acme.com/blog/${params.slug}` },
    openGraph: { title: item.title, type: 'article', images: [{ url: item.coverImage }] },
    twitter: { card: 'summary_large_image' },
  }
}

// Root layout — always set metadataBase
export const metadata: Metadata = {
  metadataBase: new URL('https://acme.com'),
  title: { default: 'Site Name', template: '%s | Site Name' },
  robots: { index: true, follow: true },
}
```

### Next.js sitemap

```ts
// app/sitemap.ts
import { MetadataRoute } from 'next'
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getAllPosts()
  return [
    { url: 'https://acme.com', changeFrequency: 'daily', priority: 1 },
    ...posts.map(p => ({
      url: `https://acme.com/blog/${p.slug}`,
      lastModified: p.updatedAt,
      priority: 0.8,
    })),
  ]
}
```

### Next.js robots.txt

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

### Remix

```tsx
import type { MetaFunction } from '@remix-run/node'

export const meta: MetaFunction<typeof loader> = ({ data, params }) => {
  if (!data) return [{ title: 'Not Found' }]
  return [
    { title: `${data.post.title} | Blog` },
    { name: 'description', content: data.post.excerpt },
    { tagName: 'link', rel: 'canonical', href: `https://acme.com/blog/${params.slug}` },
    { property: 'og:title', content: data.post.title },
    { property: 'og:image', content: data.post.coverImage },
    { property: 'og:type', content: 'article' },
    { name: 'twitter:card', content: 'summary_large_image' },
  ]
}
```

Remix sitemap: resource route at `app/routes/sitemap[.xml].tsx` returning an XML `Response`.

### React SPA

```tsx
// npm install react-helmet-async
// In root: <HelmetProvider><App /></HelmetProvider>
import { Helmet } from 'react-helmet-async'

export function Page({ data }) {
  return (
    <>
      <Helmet>
        <title>{data.title} | Site</title>
        <meta name="description" content={data.description} />
        <link rel="canonical" href={data.canonicalUrl} />
        <meta property="og:title" content={data.title} />
        <meta property="og:image" content={data.image} />
        <meta property="og:type" content="website" />
        <meta name="twitter:card" content="summary_large_image" />
      </Helmet>
      {/* content */}
    </>
  )
}
```

**Rendering risk on SEO-critical SPAs:** Google renders JavaScript but with deferred timing, limited resources, and weaker guarantees for dynamic meta tags. react-helmet-async helps with social previews but does not eliminate the rendering reliability gap. Recommend Next.js or Remix for reliable server-rendered metadata.

### JSON-LD

JSON-LD belongs in the **page component** as a `<script>` tag. It cannot be emitted from `generateMetadata` — that function only controls `<head>` meta elements.

```tsx
const schema = {
  '@context': 'https://schema.org',
  '@type': 'Article', // Article | Product | FAQPage | Organization | Event | Recipe | WebPage
  headline: data.title,
  description: data.excerpt,
  author: { '@type': 'Person', name: data.author },
  datePublished: data.publishedAt,
  image: data.coverImage,
  url: data.canonicalUrl,
}

<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
/>
```

Schema type detection: `/blog/[slug]` → Article, `/products/[id]` → Product, `/faq` → FAQPage, `/` → Organization+WebSite, `/events/[id]` → Event.

### i18n — hreflang

```tsx
// In generateMetadata — for alternates/hreflang (not JSON-LD)
alternates: {
  canonical: `https://acme.com/${locale}/page`,
  languages: {
    'x-default': 'https://acme.com/en/page',
    en: 'https://acme.com/en/page',
    fr: 'https://acme.com/fr/page',
  },
},
```

Every locale variant must link to all others (reciprocal). Always include `x-default`. Missing hreflang causes incorrect locale disambiguation and wrong-region serving — the real risk is Google misclustering locale variants, not a generic duplicate-content penalty.

---

## SEO checklist

- [ ] `<title>` — unique, under 60 chars, includes primary keyword
- [ ] `meta description` — under 160 chars, compelling
- [ ] `canonical` — on pages with duplicate content risk
- [ ] `robots` meta — not accidentally `noindex` on public pages
- [ ] `og:title`, `og:description`, `og:image`, `og:url`, `og:type`
- [ ] `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- [ ] JSON-LD in page component with correct schema type
- [ ] Informational `<img>` have descriptive `alt`; decorative `<img>` use `alt=""` (empty, not missing)
- [ ] `sitemap.xml` exists and referenced in `robots.txt`
- [ ] `robots.txt` not blocking public routes
- [ ] hreflang on every locale variant (if i18n project)
- [ ] Page 1 canonical is clean, pages 2+ are self-referencing (if paginated)
