# Internationalization (i18n) SEO Patterns

## Why i18n SEO Matters

Without hreflang, Google cannot reliably distinguish locale variants from one another. It may cluster `/en/about` and `/fr/about` together and serve the wrong regional page to users — or exclude variants from locale-specific search results. Hreflang tells Google: "these pages are translations of each other, serve the right one to the right user."

## hreflang Rules

1. Every locale variant must link to **all** other variants (including itself)
2. Always include `x-default` — points to the fallback/default locale
3. Use BCP 47 language tags: `en`, `en-US`, `fr`, `fr-FR`, `pt-BR`
4. Canonical and hreflang must be consistent — canonical must match one of the hreflang URLs

## Next.js App Router — with next-intl

```tsx
// app/[locale]/blog/[slug]/page.tsx
import { getTranslations } from "next-intl/server"
import type { Metadata } from "next"

const locales = ["en", "fr", "de", "es"]

export async function generateMetadata({
  params,
}: {
  params: { locale: string; slug: string }
}): Promise<Metadata> {
  const t = await getTranslations({ locale: params.locale, namespace: "Blog" })
  const post = await getPost(params.slug, params.locale)
  const baseUrl = "https://acme.com"

  return {
    title: post.title,
    description: post.excerpt,
    alternates: {
      canonical: `${baseUrl}/${params.locale}/blog/${params.slug}`,
      languages: Object.fromEntries(
        locales.map((locale) => [
          locale,
          `${baseUrl}/${locale}/blog/${params.slug}`,
        ])
      ),
    },
    openGraph: {
      title: post.title,
      description: post.excerpt,
      locale: params.locale,
      alternateLocale: locales.filter((l) => l !== params.locale),
    },
  }
}
```

The `alternates.languages` object in Next.js metadata API automatically renders as:
```html
<link rel="alternate" hreflang="en" href="https://acme.com/en/blog/slug" />
<link rel="alternate" hreflang="fr" href="https://acme.com/fr/blog/slug" />
<link rel="canonical" href="https://acme.com/en/blog/slug" />
```

## Next.js App Router — x-default

Always add `x-default` to the languages object:

```tsx
alternates: {
  canonical: `${baseUrl}/${params.locale}/blog/${params.slug}`,
  languages: {
    "x-default": `${baseUrl}/en/blog/${params.slug}`,
    en: `${baseUrl}/en/blog/${params.slug}`,
    fr: `${baseUrl}/fr/blog/${params.slug}`,
    de: `${baseUrl}/de/blog/${params.slug}`,
  },
},
```

## Next.js Root Layout — Site-Wide hreflang

For static routes that exist in all locales, set hreflang in the root layout:

```tsx
// app/[locale]/layout.tsx
export async function generateMetadata({
  params,
}: {
  params: { locale: string }
}): Promise<Metadata> {
  const baseUrl = "https://acme.com"
  return {
    metadataBase: new URL(baseUrl),
    alternates: {
      languages: {
        "x-default": `${baseUrl}/en`,
        en: `${baseUrl}/en`,
        fr: `${baseUrl}/fr`,
        de: `${baseUrl}/de`,
      },
    },
  }
}
```

## Next.js Pages Router — Manual hreflang

```tsx
// pages/[locale]/about.tsx
import Head from "next/head"

const locales = ["en", "fr", "de"]
const baseUrl = "https://acme.com"

export default function About({ locale }: { locale: string }) {
  return (
    <>
      <Head>
        <link rel="canonical" href={`${baseUrl}/${locale}/about`} />
        <link rel="alternate" hreflang="x-default" href={`${baseUrl}/en/about`} />
        {locales.map((l) => (
          <link key={l} rel="alternate" hreflang={l} href={`${baseUrl}/${l}/about`} />
        ))}
      </Head>
    </>
  )
}
```

## Remix — hreflang in meta export

```tsx
// app/routes/$locale.about.tsx
export const meta: MetaFunction<typeof loader> = ({ data, params }) => {
  const baseUrl = "https://acme.com"
  const locales = ["en", "fr", "de"]

  return [
    { title: data?.title },
    { tagName: "link", rel: "canonical", href: `${baseUrl}/${params.locale}/about` },
    { tagName: "link", rel: "alternate", hreflang: "x-default", href: `${baseUrl}/en/about` },
    ...locales.map((locale) => ({
      tagName: "link",
      rel: "alternate",
      hreflang: locale,
      href: `${baseUrl}/${locale}/about`,
    })),
  ]
}
```

## Sitemap with Locales

### Next.js app/sitemap.ts

```ts
import { MetadataRoute } from "next"

const locales = ["en", "fr", "de"]
const baseUrl = "https://acme.com"

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getAllPosts()

  // Static pages — all locales
  const staticPages = ["/", "/about", "/contact"].flatMap((path) =>
    locales.map((locale) => ({
      url: `${baseUrl}/${locale}${path}`,
      lastModified: new Date(),
      changeFrequency: "monthly" as const,
      alternates: {
        languages: Object.fromEntries(
          locales.map((l) => [l, `${baseUrl}/${l}${path}`])
        ),
      },
    }))
  )

  // Dynamic pages — blog posts per locale
  const blogPages = posts.flatMap((post) =>
    locales.map((locale) => ({
      url: `${baseUrl}/${locale}/blog/${post.slug}`,
      lastModified: post.updatedAt,
      alternates: {
        languages: Object.fromEntries(
          locales.map((l) => [l, `${baseUrl}/${l}/blog/${post.slug}`])
        ),
      },
    }))
  )

  return [...staticPages, ...blogPages]
}
```

## Common i18n SEO Mistakes to Flag

| Mistake | Why it's a problem | Fix |
|---------|-------------------|-----|
| No hreflang at all | Google cannot distinguish locale variants; may serve wrong region or exclude variants | Add hreflang to every locale variant |
| hreflang not reciprocal | Google ignores unreciprocated hreflang | Every locale must link to all others |
| Missing `x-default` | No fallback for users whose locale isn't covered | Point `x-default` to your default locale |
| Canonical points to different locale | Contradicts hreflang, Google drops it | Canonical must match the current page's locale URL |
| Using query params for locale (`?lang=fr`) | Less reliable for Google than path-based (`/fr/`) | Prefer path-based locale routing |
| Same content, different locale URL, no hreflang | Google may cluster variants incorrectly and serve the wrong locale | Add hreflang or consolidate |

## Detecting i18n Setup

When auditing, check for these signals that hreflang is needed:

```bash
# Path-based locale routing
grep -r "\[locale\]" app/ pages/
grep -r "i18n" next.config.*
# Library usage
grep -E "next-intl|next-i18next|i18next|react-i18next" package.json
# Locale config
ls | grep -E "i18n|intl"
```

If any match → check every public route for hreflang. Missing hreflang on an i18n site = **Critical** finding.
