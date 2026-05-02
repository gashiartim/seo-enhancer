# React SPA SEO Patterns

## The Fundamental Problem

Client-side rendered SPAs (React + Vite, CRA, etc.) render content via JavaScript. Googlebot may not execute JS before indexing, meaning meta tags set dynamically via JS can be invisible to crawlers.

**Always surface this as a Critical finding** for SEO-critical public sites. Add the crawlability warning from the main skill. Skip for internal tools, dashboards, or auth-gated apps.

## react-helmet-async Setup

Prefer `react-helmet-async` over `react-helmet` — it's thread-safe and actively maintained.

```bash
npm install react-helmet-async
```

Wrap the app root:

```tsx
// main.tsx or App.tsx
import { HelmetProvider } from 'react-helmet-async'

function App() {
  return (
    <HelmetProvider>
      <Router>
        <Routes />
      </Router>
    </HelmetProvider>
  )
}
```

## Static Page Meta Tags

```tsx
import { Helmet } from 'react-helmet-async'

export function AboutPage() {
  return (
    <>
      <Helmet>
        <title>About Us | Acme Corp</title>
        <meta name="description" content="Learn about our mission and team." />
        <link rel="canonical" href="https://acme.com/about" />
        <meta property="og:title" content="About Us | Acme Corp" />
        <meta property="og:description" content="Learn about our mission and team." />
        <meta property="og:image" content="https://acme.com/og-about.png" />
        <meta property="og:url" content="https://acme.com/about" />
        <meta property="og:type" content="website" />
        <meta name="twitter:card" content="summary_large_image" />
        <meta name="twitter:title" content="About Us | Acme Corp" />
        <meta name="twitter:description" content="Learn about our mission and team." />
        <meta name="twitter:image" content="https://acme.com/og-about.png" />
      </Helmet>
      {/* page content */}
    </>
  )
}
```

## Dynamic Page Meta Tags

```tsx
import { Helmet } from 'react-helmet-async'
import { useQuery } from '@tanstack/react-query'

export function BlogPost({ slug }: { slug: string }) {
  const { data: post } = useQuery({
    queryKey: ['post', slug],
    queryFn: () => fetchPost(slug),
  })

  if (!post) return <LoadingSkeleton />

  return (
    <>
      <Helmet>
        <title>{post.title} | Blog</title>
        <meta name="description" content={post.excerpt} />
        <link rel="canonical" href={`https://acme.com/blog/${slug}`} />
        <meta property="og:title" content={post.title} />
        <meta property="og:description" content={post.excerpt} />
        <meta property="og:image" content={post.coverImage} />
        <meta property="og:url" content={`https://acme.com/blog/${slug}`} />
        <meta property="og:type" content="article" />
        <meta name="twitter:card" content="summary_large_image" />
        <meta name="twitter:image" content={post.coverImage} />
      </Helmet>
      {/* content */}
    </>
  )
}
```

## Default/Fallback Meta Tags

Add a default `<Helmet>` in the root layout — child `<Helmet>` tags override it per page:

```tsx
// App.tsx or RootLayout.tsx
import { Helmet } from 'react-helmet-async'

export function RootLayout({ children }) {
  return (
    <>
      <Helmet defaultTitle="Acme Corp" titleTemplate="%s | Acme Corp">
        <meta name="description" content="Default site description." />
        <meta property="og:site_name" content="Acme Corp" />
        <meta name="twitter:card" content="summary_large_image" />
      </Helmet>
      {children}
    </>
  )
}
```

## JSON-LD in React SPA

```tsx
import { Helmet } from 'react-helmet-async'

export function BlogPost({ post }) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: post.title,
    description: post.excerpt,
    author: { '@type': 'Person', name: post.author.name },
    datePublished: post.publishedAt,
    image: post.coverImage,
    url: `https://acme.com/blog/${post.slug}`,
  }

  return (
    <>
      <Helmet>
        <script type="application/ld+json">{JSON.stringify(jsonLd)}</script>
      </Helmet>
      {/* content */}
    </>
  )
}
```

## Static Sitemap for SPA

SPAs don't have a server to generate sitemaps dynamically. Options:

**Option 1 — Static XML** (for small sites with known routes):
```xml
<!-- public/sitemap.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://acme.com/</loc>
    <lastmod>2024-01-01</lastmod>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://acme.com/about</loc>
    <changefreq>monthly</changefreq>
    <priority>0.5</priority>
  </url>
</urlset>
```

**Option 2 — Build-time generation script** (for dynamic routes fetched from an API):
```js
// scripts/generate-sitemap.mjs
import { writeFileSync } from 'fs'

const BASE_URL = 'https://acme.com'

async function generateSitemap() {
  const posts = await fetch(`${BASE_URL}/api/posts`).then(r => r.json())

  const urls = [
    `<url><loc>${BASE_URL}/</loc><priority>1.0</priority></url>`,
    `<url><loc>${BASE_URL}/about</loc><priority>0.5</priority></url>`,
    ...posts.map(p => `<url><loc>${BASE_URL}/blog/${p.slug}</loc><lastmod>${p.updatedAt}</lastmod></url>`),
  ]

  const xml = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${urls.join('\n')}
</urlset>`

  writeFileSync('public/sitemap.xml', xml)
  console.log('Sitemap generated')
}

generateSitemap()
```

Add to `package.json`:
```json
"scripts": {
  "build": "vite build && node scripts/generate-sitemap.mjs"
}
```

## robots.txt for SPA

```
# public/robots.txt
User-agent: *
Allow: /
Disallow: /admin/

Sitemap: https://acme.com/sitemap.xml
```

## Crawlability Warning (when to show)

Show this warning when ALL of the following are true:
1. Framework is React SPA (Vite, CRA, no SSR)
2. Site has public-facing pages that should be indexed
3. No prerendering service detected in the codebase

```
⚠️  Crawlability Risk Detected
This is a client-side rendered React SPA. Meta tags added via react-helmet-async
will work for human visitors, but may NOT be indexed by Googlebot if it doesn't
execute JavaScript before crawling.

To fully solve this, consider:
  1. Migrating to Next.js (recommended — full SSR/SSG, native metadata API)
  2. Adding a prerendering service (prerender.io, rendertron)
  3. Vite SSR (advanced, gives you SSR without changing frameworks)

react-helmet-async has been added for meta tag management. This improves
social sharing previews (Slack, Twitter, etc.) which do execute JS, but
does not guarantee Google indexing of dynamic meta tags.
```
