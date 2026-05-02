# Remix SEO Patterns

## How Remix Metadata Works

Remix uses a `meta` export function per route. It receives loader data, params, and parent matches — so dynamic metadata is first-class with no extra fetches.

```tsx
// app/routes/blog.$slug.tsx
import type { MetaFunction, LoaderFunctionArgs } from "@remix-run/node"
import { json } from "@remix-run/node"
import { useLoaderData } from "@remix-run/react"

export async function loader({ params }: LoaderFunctionArgs) {
  const post = await getPost(params.slug!)
  if (!post) throw new Response("Not Found", { status: 404 })
  return json({ post })
}

export const meta: MetaFunction<typeof loader> = ({ data, params }) => {
  if (!data) return [{ title: "Post Not Found" }]
  const { post } = data
  return [
    { title: `${post.title} | Blog` },
    { name: "description", content: post.excerpt },
    { tagName: "link", rel: "canonical", href: `https://acme.com/blog/${params.slug}` },
    { property: "og:title", content: post.title },
    { property: "og:description", content: post.excerpt },
    { property: "og:image", content: post.coverImage },
    { property: "og:url", content: `https://acme.com/blog/${params.slug}` },
    { property: "og:type", content: "article" },
    { name: "twitter:card", content: "summary_large_image" },
    { name: "twitter:title", content: post.title },
    { name: "twitter:description", content: post.excerpt },
    { name: "twitter:image", content: post.coverImage },
  ]
}
```

## Root Route — Default / Fallback Meta

```tsx
// app/root.tsx
export const meta: MetaFunction = () => [
  { title: "Acme Corp" },
  { name: "description", content: "Default site description." },
  { property: "og:site_name", content: "Acme Corp" },
  { name: "twitter:card", content: "summary_large_image" },
]
```

Child route meta arrays **replace** the parent's by default. To merge with parent meta (e.g. keep root defaults and add page-specific tags):

```tsx
export const meta: MetaFunction<typeof loader> = ({ data, matches }) => {
  const rootMeta = matches.find(m => m.id === "root")?.meta ?? []
  return [
    ...rootMeta,
    { title: `${data?.post.title} | Blog` },
    { name: "description", content: data?.post.excerpt },
  ]
}
```

## Static Page Meta

```tsx
// app/routes/about.tsx
export const meta: MetaFunction = () => [
  { title: "About Us | Acme Corp" },
  { name: "description", content: "Learn about our mission and team." },
  { tagName: "link", rel: "canonical", href: "https://acme.com/about" },
  { property: "og:title", content: "About Us | Acme Corp" },
  { property: "og:description", content: "Learn about our mission and team." },
  { property: "og:image", content: "https://acme.com/og-about.png" },
  { property: "og:url", content: "https://acme.com/about" },
  { property: "og:type", content: "website" },
]
```

## JSON-LD in Remix

Inject via a `<script>` tag in the component — Remix renders it server-side so Google sees it:

```tsx
export default function BlogPost() {
  const { post } = useLoaderData<typeof loader>()

  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "Article",
    headline: post.title,
    description: post.excerpt,
    author: { "@type": "Person", name: post.author.name },
    datePublished: post.publishedAt,
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

## Sitemap — Resource Route

```tsx
// app/routes/sitemap[.xml].tsx
import type { LoaderFunctionArgs } from "@remix-run/node"

export async function loader({ request }: LoaderFunctionArgs) {
  const posts = await getAllPosts()
  const baseUrl = new URL(request.url).origin

  const urls = [
    `<url><loc>${baseUrl}/</loc><priority>1.0</priority></url>`,
    `<url><loc>${baseUrl}/about</loc><priority>0.5</priority></url>`,
    ...posts.map(p =>
      `<url><loc>${baseUrl}/blog/${p.slug}</loc><lastmod>${p.updatedAt}</lastmod><priority>0.8</priority></url>`
    ),
  ]

  const xml = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${urls.join("\n")}
</urlset>`

  return new Response(xml, {
    headers: {
      "Content-Type": "application/xml",
      "Cache-Control": "public, max-age=3600",
    },
  })
}
```

## robots.txt — Resource Route

```tsx
// app/routes/robots[.txt].tsx
export async function loader() {
  const content = `User-agent: *
Allow: /
Disallow: /admin/
Disallow: /api/

Sitemap: https://acme.com/sitemap.xml`

  return new Response(content, {
    headers: { "Content-Type": "text/plain" },
  })
}
```

## Canonical URLs

Remix doesn't have a built-in canonical helper. Add via `tagName: "link"` in the meta export:

```tsx
export const meta: MetaFunction = ({ request }) => [
  { tagName: "link", rel: "canonical", href: "https://acme.com/page" },
]
```

For dynamic canonicals, build from loader data or params:

```tsx
export const meta: MetaFunction<typeof loader> = ({ data, params }) => [
  { tagName: "link", rel: "canonical", href: `https://acme.com/blog/${params.slug}` },
]
```

## robots Meta Tag

```tsx
// Prevent indexing on specific pages (e.g. admin, preview)
export const meta: MetaFunction = () => [
  { name: "robots", content: "noindex, nofollow" },
]
```

## Checking for Missing `meta` Exports

When auditing a Remix project, look for route files under `app/routes/` that:
- Render public-facing content (not `_index`, not `api.`, not `$`)
- Are missing an `export const meta` function

These are Critical findings — add `meta` exports for each.
