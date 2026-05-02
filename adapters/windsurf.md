# SEO Enhancer — Windsurf Rules

> Copy this file contents into your project's `.windsurfrules` file, or add to your global `~/.codeium/windsurf/memories/global_rules.md`.

## SEO rules for Next.js, Remix, and React SPAs

Apply these rules when editing page, route, or layout files in a web project, or when the user mentions SEO, meta tags, rankings, sitemaps, Open Graph, Twitter cards, structured data, hreflang, or social previews.

### Detect framework

Check `package.json` deps:
- `next` → Next.js. `app/` = app router, `pages/` = pages router
- `@remix-run/react` → Remix
- `react` + `vite` or `react-scripts` → React SPA (no SSR)
- `next-intl` / `next-i18next` / `[locale]` path segments → i18n project

### Audit — two tiers

**Critical (fix immediately — genuine indexing risk):**
- Public page with no `<title>` or `meta description`, with no coverage from a parent layout
- Next.js page/layout missing `metadata` or `generateMetadata` AND no ancestor layout providing coverage
- Remix route missing `export const meta` AND no inherited root defaults covering it
- Informational `<img>` without `alt` — decorative images with `alt=""` are correct, do not flag
- No `canonical` on pages with real duplicate content risk
- React SPA on an SEO-critical public site with no acknowledgment of rendering risk
- i18n site missing `hreflang` alternate links

**Enhancement (show code example, ask before applying):**
- Missing Open Graph / Twitter card tags
- Missing JSON-LD structured data
- No sitemap
- Missing `robots.txt` (crawl is allowed by default; absence is not a blocker)
- `next/head` in app router (native metadata API is preferred, not required)
- `next/image` instead of raw `<img>` (performance improvement, not an SEO requirement)
- `robots.txt` missing sitemap pointer

---

## Implementations

**Next.js app router — static:**
```tsx
export const metadata: Metadata = {
  title: 'Page | Site',
  description: 'Under 160 chars.',
  alternates: { canonical: 'https://acme.com/page' },
  openGraph: { title: 'Page | Site', images: [{ url: 'https://acme.com/og.png', width: 1200, height: 630 }] },
  twitter: { card: 'summary_large_image' },
}
```

**Next.js app router — dynamic (reuse same fetch as page, Next.js deduplicates):**
```tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const item = await getData(params.slug)
  return {
    title: item.title,
    alternates: { canonical: `https://acme.com/${params.slug}` },
    openGraph: { title: item.title, images: [{ url: item.image }] },
  }
}
```

**Next.js root layout:**
```tsx
export const metadata: Metadata = {
  metadataBase: new URL('https://acme.com'),
  title: { default: 'Site Name', template: '%s | Site Name' },
}
```

**Next.js sitemap:**
```ts
// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getAllPosts()
  return [
    { url: 'https://acme.com', changeFrequency: 'daily', priority: 1 },
    ...posts.map(p => ({ url: `https://acme.com/blog/${p.slug}`, lastModified: p.updatedAt })),
  ]
}
```

**Remix:**
```tsx
export const meta: MetaFunction<typeof loader> = ({ data, params }) => [
  { title: `${data?.title} | Site` },
  { name: 'description', content: data?.description },
  { tagName: 'link', rel: 'canonical', href: `https://acme.com/${params.slug}` },
  { property: 'og:title', content: data?.title },
  { property: 'og:image', content: data?.image },
  { name: 'twitter:card', content: 'summary_large_image' },
]
```

**React SPA:**
```tsx
import { Helmet } from 'react-helmet-async'
// Wrap root: <HelmetProvider><App /></HelmetProvider>
<Helmet>
  <title>{title} | Site</title>
  <meta name="description" content={description} />
  <link rel="canonical" href={canonicalUrl} />
  <meta property="og:title" content={title} />
  <meta property="og:image" content={ogImage} />
  <meta name="twitter:card" content="summary_large_image" />
</Helmet>
```

**Rendering risk on SEO-critical SPAs:** Google renders JavaScript but with deferred timing, limited resources, and weaker guarantees for dynamic meta tags. Next.js or Remix provide reliable server-rendered metadata.

**JSON-LD — inject in the page component, not in `generateMetadata`:**
```tsx
<script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }} />
```
Schema types: `Article` (blog), `Product` (ecommerce), `FAQPage`, `Organization`+`WebSite` (homepage).

**i18n hreflang:**
```tsx
alternates: {
  canonical: `https://acme.com/${locale}/page`,
  languages: { 'x-default': 'https://acme.com/en/page', en: '...', fr: '...' },
},
```
Missing hreflang causes incorrect locale disambiguation and wrong-region serving.

---

## Checklist

- [ ] `<title>` unique, under 60 chars
- [ ] `meta description` under 160 chars
- [ ] `canonical` on duplicate-content pages
- [ ] OG: title, description, image, url, type
- [ ] Twitter card tags present
- [ ] JSON-LD in page component with correct schema type
- [ ] Informational `<img>` have descriptive `alt`; decorative `<img>` have `alt=""`
- [ ] `sitemap.xml` linked in `robots.txt`
- [ ] `robots.txt` not blocking public routes
- [ ] hreflang on every locale (if i18n project)
