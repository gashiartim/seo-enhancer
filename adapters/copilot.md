# SEO Enhancer — GitHub Copilot Instructions

> Copy this file to `.github/copilot-instructions.md` in your project root.

When working on Next.js, Remix, or React SPA projects, apply these SEO rules automatically.

## Trigger on

- Any file in `app/`, `pages/`, `app/routes/`, or `src/pages/` (page, layout, and route files — not generic components)
- User mentions: SEO, meta tags, Open Graph, sitemap, robots.txt, structured data, hreflang, ranking, discoverability, social previews
- Page/layout/route files missing metadata exports

## Framework detection

Read `package.json`:
- `next` → Next.js
- `@remix-run/react` → Remix
- `react` + `vite` or `react-scripts` → React SPA (no SSR)

## Critical issues — fix without asking

- Public page missing `<title>` or `meta description`
- Next.js page/layout missing `metadata` export or `generateMetadata`
- Remix route missing `export const meta`
- `<img>` without `alt`
- No `robots.txt` in production
- No canonical on duplicate-content pages
- React SPA (no SSR) on SEO-critical site without crawlability warning
- i18n site missing `hreflang` alternate links

## Enhancements — show example, ask before applying

- Missing Open Graph / Twitter card tags
- Missing JSON-LD structured data
- No sitemap
- `next/head` in app router (migrate to metadata API)
- Raw `<img>` instead of `next/image`
- `robots.txt` missing sitemap pointer

---

## Next.js — app router

```tsx
// Static metadata
export const metadata: Metadata = {
  title: 'Page Title | Site',
  description: 'Under 160 chars.',
  alternates: { canonical: 'https://acme.com/page' },
  openGraph: {
    title: 'Page Title | Site',
    description: 'Under 160 chars.',
    images: [{ url: 'https://acme.com/og.png', width: 1200, height: 630 }],
  },
  twitter: { card: 'summary_large_image' },
}

// Dynamic metadata — use same fetch as page component (Next.js deduplicates)
export async function generateMetadata({ params }): Promise<Metadata> {
  const data = await getData(params.slug)
  return {
    title: data.title,
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

Sitemap:
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

robots.txt:
```ts
// app/robots.ts
export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/', disallow: ['/admin/'] },
    sitemap: 'https://acme.com/sitemap.xml',
  }
}
```

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
// npm install react-helmet-async
import { Helmet } from 'react-helmet-async'

<Helmet>
  <title>{title} | Site</title>
  <meta name="description" content={description} />
  <link rel="canonical" href={url} />
  <meta property="og:title" content={title} />
  <meta property="og:image" content={image} />
  <meta name="twitter:card" content="summary_large_image" />
</Helmet>
```

Always warn on SEO-critical SPAs: Google renders JS but with deferred timing, limited resources, and weaker guarantees for dynamic meta tags. Next.js or Remix provide reliable server-rendered metadata.

## JSON-LD

```tsx
<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{ __html: JSON.stringify({
    '@context': 'https://schema.org',
    '@type': 'Article', // or Product, FAQPage, Organization, etc.
    headline: title,
    description: excerpt,
    author: { '@type': 'Person', name: authorName },
    datePublished: publishedAt,
    image: coverImage,
  })}}
/>
```

Schema type guide: `/blog/[slug]` → Article, `/products/[id]` → Product, `/faq` → FAQPage, `/` → Organization + WebSite.

## i18n — hreflang

```tsx
// Next.js generateMetadata
alternates: {
  canonical: `https://acme.com/${locale}/page`,
  languages: {
    'x-default': 'https://acme.com/en/page',
    en: 'https://acme.com/en/page',
    fr: 'https://acme.com/fr/page',
  },
},
```

Every locale variant must link to all others (reciprocal). Always include `x-default`.
