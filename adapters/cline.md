# SEO Enhancer — Cline Rules

> Copy this file contents into your project's `.clinerules` file.

## SEO rules for Next.js, Remix, and React SPAs

Apply these rules when editing page, route, or layout files in a web project, or when the user mentions SEO, meta tags, rankings, sitemaps, Open Graph, Twitter cards, structured data, hreflang, or social previews.

### Detect framework

Check `package.json` deps:
- `next` → Next.js. `app/` directory = app router, `pages/` = pages router
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

## Patterns

### Next.js app router

```tsx
// Static
export const metadata: Metadata = {
  title: 'Page | Site',
  description: 'Under 160 chars.',
  alternates: { canonical: 'https://acme.com/page' },
  openGraph: { title: 'Page | Site', images: [{ url: 'https://acme.com/og.png', width: 1200, height: 630 }] },
  twitter: { card: 'summary_large_image' },
}

// Dynamic — reuse the same fetch as the page (Next.js deduplicates)
export async function generateMetadata({ params }): Promise<Metadata> {
  const item = await getData(params.slug)
  return {
    title: item.title,
    alternates: { canonical: `https://acme.com/${params.slug}` },
    openGraph: { title: item.title, images: [{ url: item.image }] },
  }
}

// Root layout — always set metadataBase
export const metadata: Metadata = {
  metadataBase: new URL('https://acme.com'),
  title: { default: 'Site', template: '%s | Site' },
}
```

Sitemap: `app/sitemap.ts` returning `MetadataRoute.Sitemap`.
robots.txt: `app/robots.ts` returning `MetadataRoute.Robots`.

### Remix

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

Sitemap via resource route at `app/routes/sitemap[.xml].tsx`.

### React SPA

```tsx
import { Helmet } from 'react-helmet-async'
// Wrap root: <HelmetProvider><App /></HelmetProvider>

<Helmet>
  <title>{title} | Site</title>
  <meta name="description" content={description} />
  <link rel="canonical" href={url} />
  <meta property="og:title" content={title} />
  <meta property="og:image" content={image} />
  <meta name="twitter:card" content="summary_large_image" />
</Helmet>
```

**Rendering risk on SEO-critical SPAs:** Google renders JavaScript but with deferred timing, limited resources, and weaker guarantees for dynamic meta tags. Next.js or Remix provide reliable server-rendered metadata. react-helmet-async helps with social previews but does not eliminate the rendering reliability gap.

### JSON-LD

JSON-LD belongs in the **page component** as a `<script>` tag. It cannot be emitted from `generateMetadata` — that function only controls `<head>` meta elements.

```tsx
const schema = {
  '@context': 'https://schema.org',
  '@type': 'Article', // Article | Product | FAQPage | Organization | Event | Recipe | WebPage
  headline: title,
  description: excerpt,
  author: { '@type': 'Person', name: authorName },
  datePublished: publishedAt,
  image: coverImage,
}
<script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }} />
```

### i18n hreflang (Next.js)

```tsx
alternates: {
  canonical: `https://acme.com/${locale}/page`,
  languages: {
    'x-default': 'https://acme.com/en/page',
    en: 'https://acme.com/en/page',
    fr: 'https://acme.com/fr/page',
  },
},
```

Every locale variant must link to all others (reciprocal). Always include `x-default`. Missing hreflang causes incorrect locale disambiguation and wrong-region serving — not a generic duplicate-content penalty.

---

## Checklist

- [ ] `<title>` unique, under 60 chars, includes primary keyword
- [ ] `meta description` under 160 chars
- [ ] `canonical` on pages with duplicate content risk
- [ ] `robots` meta not accidentally `noindex`
- [ ] `og:title`, `og:description`, `og:image`, `og:url`, `og:type`
- [ ] `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- [ ] JSON-LD in page component with schema type matching content
- [ ] Informational `<img>` have descriptive `alt`; decorative `<img>` use `alt=""` (empty, not missing)
- [ ] `sitemap.xml` exists and linked in `robots.txt`
- [ ] `robots.txt` not blocking public routes
- [ ] hreflang on every locale variant (if i18n)
- [ ] Page 1 canonical clean, pages 2+ self-referencing (if paginated)
