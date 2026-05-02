# SEO Enhancer — GitHub Copilot Instructions

> Copy this file to `.github/copilot-instructions.md` in your project root.

When working on Next.js, Remix, or React SPA projects, apply these SEO rules automatically.

## Trigger on

- Files in `app/`, `pages/`, `app/routes/`, or `src/pages/` (page, layout, and route files — not generic UI components)
- User mentions: SEO, meta tags, Open Graph, sitemap, robots.txt, structured data, hreflang, ranking, discoverability, social previews
- Page/layout/route files missing metadata exports with no inherited coverage

## Framework detection

Read `package.json`:
- `next` → Next.js (`app/` = app router, `pages/` = pages router)
- `@remix-run/react` → Remix
- `react` + `vite`/`react-scripts` → React SPA (no SSR)

## Critical issues — fix without asking

These represent genuine indexing risk:

- Public page missing `<title>` or `meta description`, with no coverage from a parent layout
- Next.js page/layout missing `metadata` or `generateMetadata` AND no ancestor layout providing coverage
- Remix route missing `export const meta` AND no root defaults covering it
- Informational `<img>` missing `alt` (decorative images with `alt=""` are correct — do not flag)
- No canonical on pages with real duplicate content risk
- React SPA on SEO-critical public site without acknowledging rendering risk
- i18n site missing `hreflang` alternate links

## Enhancements — show example, ask before applying

- Missing Open Graph / Twitter card tags
- Missing JSON-LD structured data
- No sitemap
- Missing `robots.txt` (crawl is allowed by default; absence is not a blocker)
- `next/head` in app router (native metadata API is preferred, not required)
- `next/image` instead of raw `<img>` (performance improvement, not an SEO requirement)
- `robots.txt` missing sitemap pointer

---

## Next.js — app router

```tsx
// Static
export const metadata: Metadata = {
  title: 'Page | Site',
  description: 'Under 160 chars.',
  alternates: { canonical: 'https://acme.com/page' },
  openGraph: {
    title: 'Page | Site',
    images: [{ url: 'https://acme.com/og.png', width: 1200, height: 630 }],
  },
  twitter: { card: 'summary_large_image' },
}

// Dynamic — reuse the same fetch as the page component (Next.js deduplicates)
export async function generateMetadata({ params }): Promise<Metadata> {
  const data = await getData(params.slug)
  return {
    title: `${data.title} | Site`,
    description: data.excerpt,
    alternates: { canonical: `https://acme.com/${params.slug}` },
    openGraph: { title: data.title, images: [{ url: data.image }] },
  }
}

// Root layout — set metadataBase and title template
export const metadata: Metadata = {
  metadataBase: new URL('https://acme.com'),
  title: { default: 'Site Name', template: '%s | Site Name' },
  robots: { index: true, follow: true },
}
```

Sitemap: `app/sitemap.ts` returning `MetadataRoute.Sitemap`.
robots.txt: `app/robots.ts` returning `MetadataRoute.Robots`.

## Remix

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

## React SPA

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

## JSON-LD

JSON-LD belongs in the **page component** as a `<script>` tag. It cannot be emitted from `generateMetadata` — that function only controls `<head>` meta elements.

```tsx
<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{ __html: JSON.stringify({
    '@context': 'https://schema.org',
    '@type': 'Article', // or Product, FAQPage, Organization+WebSite, Event, Recipe
    headline: title,
    description: excerpt,
    author: { '@type': 'Person', name: authorName },
    datePublished: publishedAt,
    image: coverImage,
  })}}
/>
```

Schema types: `/blog/[slug]` → Article, `/products/[id]` → Product, `/faq` → FAQPage, `/` → Organization + WebSite.

## i18n hreflang

```tsx
// Next.js generateMetadata — for alternates/hreflang (not JSON-LD)
alternates: {
  canonical: `https://acme.com/${locale}/page`,
  languages: {
    'x-default': 'https://acme.com/en/page',
    en: 'https://acme.com/en/page',
    fr: 'https://acme.com/fr/page',
  },
},
```

Every locale variant must link to all others (reciprocal). Always include `x-default`. Missing hreflang causes incorrect locale disambiguation and wrong-region serving — not flagged as a duplicate-content penalty by Google.
