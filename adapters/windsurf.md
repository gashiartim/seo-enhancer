# SEO Enhancer — Windsurf Rules

> Copy this file contents into your project's `.windsurfules` file, or add to your global `~/.codeium/windsurf/memories/global_rules.md`.

## SEO rules for Next.js, Remix, and React SPAs

When working in a web project, apply these SEO rules automatically when:
- Editing page components, route files, or layout files
- The user mentions SEO, meta tags, rankings, sitemaps, Open Graph, or social previews
- You see a page/layout file missing metadata exports

### Detect framework from `package.json`

- `next` → Next.js (check `app/` vs `pages/` for router version)
- `@remix-run/react` → Remix
- `react` + `vite`/`react-scripts` → React SPA (client-side only)
- `next-intl` / `[locale]` folder segments → i18n project, hreflang required

---

### Critical — fix immediately

- Missing `<title>` or `meta description` on any public page
- Next.js page/layout without `metadata` export or `generateMetadata`
- Remix route without `export const meta`
- `<img>` tags without `alt` attribute
- No `robots.txt` file in production
- No canonical URL on pages with duplicate content risk
- React SPA on an SEO-critical site — add crawlability warning
- i18n site with no hreflang alternate links

### Enhancements — recommend + show code, ask before applying

- Open Graph and Twitter card tags missing
- JSON-LD structured data missing
- No sitemap
- `next/head` in app router (should be metadata API)
- Raw `<img>` instead of `next/image`
- `robots.txt` missing sitemap pointer

---

### Implementations

**Next.js app router — static page:**
```tsx
export const metadata: Metadata = {
  title: 'Page | Site',
  description: 'Description.',
  alternates: { canonical: 'https://acme.com/page' },
  openGraph: { title: 'Page | Site', images: [{ url: 'https://acme.com/og.png', width: 1200, height: 630 }] },
  twitter: { card: 'summary_large_image' },
}
```

**Next.js app router — dynamic page:**
```tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const data = await getData(params.slug) // deduplicates with page fetch
  return {
    title: data.title,
    alternates: { canonical: `https://acme.com/${params.slug}` },
    openGraph: { title: data.title, images: [{ url: data.image }] },
  }
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
  { name: 'description', content: data?.excerpt },
  { tagName: 'link', rel: 'canonical', href: `https://acme.com/${params.slug}` },
  { property: 'og:title', content: data?.title },
  { property: 'og:image', content: data?.image },
  { name: 'twitter:card', content: 'summary_large_image' },
]
```

**React SPA:**
```tsx
import { Helmet } from 'react-helmet-async'
// Wrap app root with <HelmetProvider>
<Helmet>
  <title>{title} | Site</title>
  <meta name="description" content={description} />
  <link rel="canonical" href={canonicalUrl} />
  <meta property="og:title" content={title} />
  <meta property="og:image" content={ogImage} />
  <meta name="twitter:card" content="summary_large_image" />
</Helmet>
```

**JSON-LD:**
```tsx
<script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }} />
```
Schema types: `Article` (blog), `Product` (ecommerce), `FAQPage` (FAQ), `Organization`+`WebSite` (homepage).

**i18n hreflang:**
```tsx
alternates: {
  canonical: `https://acme.com/${locale}/page`,
  languages: { 'x-default': 'https://acme.com/en/page', en: '...', fr: '...' },
},
```

---

### Checklist

- [ ] `<title>` unique, under 60 chars
- [ ] `meta description` under 160 chars
- [ ] `canonical` on duplicate-content pages
- [ ] OG: title, description, image, url, type
- [ ] Twitter card tags present
- [ ] JSON-LD with correct schema
- [ ] All `<img>` have meaningful `alt`
- [ ] `sitemap.xml` linked in `robots.txt`
- [ ] `robots.txt` not blocking public routes
- [ ] hreflang on every locale (if i18n project)
