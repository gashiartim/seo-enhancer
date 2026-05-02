# Next.js SEO Patterns

## App Router — Static Metadata

```tsx
// app/about/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'About Us | Acme Corp',
  description: 'Learn about our mission and team.',
  alternates: { canonical: 'https://acme.com/about' },
  openGraph: {
    title: 'About Us | Acme Corp',
    description: 'Learn about our mission and team.',
    url: 'https://acme.com/about',
    siteName: 'Acme Corp',
    images: [{ url: 'https://acme.com/og-about.png', width: 1200, height: 630 }],
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'About Us | Acme Corp',
    description: 'Learn about our mission and team.',
    images: ['https://acme.com/og-about.png'],
  },
}
```

## App Router — Dynamic Metadata

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next'

// Next.js deduplicates this fetch with the one in the component
async function getPost(slug: string) {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { revalidate: 3600 },
  })
  return res.json()
}

export async function generateMetadata({ params }: { params: { slug: string } }): Promise<Metadata> {
  const post = await getPost(params.slug)
  return {
    title: `${post.title} | Blog`,
    description: post.excerpt,
    alternates: { canonical: `https://acme.com/blog/${params.slug}` },
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      publishedTime: post.publishedAt,
      authors: [post.author.name],
      images: [{ url: post.coverImage, width: 1200, height: 630 }],
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  }
}
```

## App Router — Root Layout Defaults

```tsx
// app/layout.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  metadataBase: new URL('https://acme.com'), // required for relative OG image URLs
  title: {
    default: 'Acme Corp',
    template: '%s | Acme Corp', // child pages use: title: 'About' → 'About | Acme Corp'
  },
  description: 'Default site description.',
  robots: { index: true, follow: true },
  openGraph: {
    siteName: 'Acme Corp',
    type: 'website',
  },
}
```

## Pages Router — `next/head`

```tsx
// pages/about.tsx
import Head from 'next/head'

export default function About() {
  return (
    <>
      <Head>
        <title>About Us | Acme Corp</title>
        <meta name="description" content="Learn about our mission." />
        <link rel="canonical" href="https://acme.com/about" />
        <meta property="og:title" content="About Us | Acme Corp" />
        <meta property="og:description" content="Learn about our mission." />
        <meta property="og:image" content="https://acme.com/og-about.png" />
        <meta property="og:url" content="https://acme.com/about" />
        <meta name="twitter:card" content="summary_large_image" />
      </Head>
      {/* page content */}
    </>
  )
}
```

## Pages Router — Dynamic with `getServerSideProps`

```tsx
// pages/blog/[slug].tsx
import Head from 'next/head'

export async function getServerSideProps({ params }) {
  const post = await fetchPost(params.slug)
  return { props: { post } }
}

export default function BlogPost({ post }) {
  return (
    <>
      <Head>
        <title>{post.title} | Blog</title>
        <meta name="description" content={post.excerpt} />
        <link rel="canonical" href={`https://acme.com/blog/${post.slug}`} />
        <meta property="og:title" content={post.title} />
        <meta property="og:type" content="article" />
        <meta property="og:image" content={post.coverImage} />
      </Head>
      {/* content */}
    </>
  )
}
```

## Sitemap — App Router (dynamic)

```ts
// app/sitemap.ts
import { MetadataRoute } from 'next'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await fetchAllPosts()
  const postEntries = posts.map((post) => ({
    url: `https://acme.com/blog/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: 'weekly' as const,
    priority: 0.8,
  }))

  return [
    { url: 'https://acme.com', lastModified: new Date(), changeFrequency: 'daily', priority: 1 },
    { url: 'https://acme.com/about', lastModified: new Date(), changeFrequency: 'monthly', priority: 0.5 },
    ...postEntries,
  ]
}
```

## Sitemap — `next-sitemap`

```js
// next-sitemap.config.js
/** @type {import('next-sitemap').IConfig} */
module.exports = {
  siteUrl: process.env.SITE_URL || 'https://acme.com',
  generateRobotsTxt: true,
  robotsTxtOptions: {
    policies: [{ userAgent: '*', allow: '/' }],
  },
  exclude: ['/admin/*', '/api/*', '/404', '/500'],
  changefreq: 'weekly',
  priority: 0.7,
}
```

Add to `package.json`:
```json
"scripts": {
  "postbuild": "next-sitemap"
}
```

## robots.txt — App Router (generated)

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

## robots.txt — Static file

```
# public/robots.txt
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /api/

Sitemap: https://acme.com/sitemap.xml
```

## next/image for SEO

```tsx
// Always use next/image — it enforces alt, handles lazy loading, and serves optimized formats
import Image from 'next/image'

// Good
<Image src="/hero.jpg" alt="Hero image showing our product dashboard" width={1200} height={630} priority />

// Above-the-fold images: add `priority` to preload (improves LCP)
// Below-the-fold: omit `priority` (lazy loaded by default)
```

## JSON-LD in Next.js

```tsx
// Inject via script tag in the page component or generateMetadata
// App router — in the page component:
export default function BlogPost({ post }) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: post.title,
    description: post.excerpt,
    author: { '@type': 'Person', name: post.author.name },
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    image: post.coverImage,
    url: `https://acme.com/blog/${post.slug}`,
  }

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      {/* page content */}
    </>
  )
}
```
