# JSON-LD Schema Templates

Reference templates for common schema types. Adapt field values from the actual page data — never hardcode placeholder text.

## Article (blog posts, news, guides)

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Post title here",
  "description": "Post excerpt or summary",
  "author": {
    "@type": "Person",
    "name": "Author Name",
    "url": "https://acme.com/authors/name"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Acme Corp",
    "logo": {
      "@type": "ImageObject",
      "url": "https://acme.com/logo.png"
    }
  },
  "datePublished": "2024-01-15T08:00:00Z",
  "dateModified": "2024-01-20T10:00:00Z",
  "image": {
    "@type": "ImageObject",
    "url": "https://acme.com/blog/cover.jpg",
    "width": 1200,
    "height": 630
  },
  "url": "https://acme.com/blog/post-slug",
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://acme.com/blog/post-slug"
  }
}
```

## Product (e-commerce, product pages)

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "description": "Product description",
  "image": ["https://acme.com/products/img1.jpg", "https://acme.com/products/img2.jpg"],
  "sku": "PROD-001",
  "brand": {
    "@type": "Brand",
    "name": "Acme"
  },
  "offers": {
    "@type": "Offer",
    "url": "https://acme.com/products/prod-001",
    "priceCurrency": "USD",
    "price": "29.99",
    "priceValidUntil": "2025-12-31",
    "availability": "https://schema.org/InStock",
    "itemCondition": "https://schema.org/NewCondition"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.5",
    "reviewCount": "128"
  }
}
```

## FAQPage (FAQ sections, accordion content)

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is your return policy?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "We offer a 30-day return policy on all items."
      }
    },
    {
      "@type": "Question",
      "name": "How long does shipping take?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Standard shipping takes 5-7 business days."
      }
    }
  ]
}
```

## Organization (homepage, about page)

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Acme Corp",
  "url": "https://acme.com",
  "logo": "https://acme.com/logo.png",
  "description": "What the company does.",
  "foundingDate": "2020",
  "contactPoint": {
    "@type": "ContactPoint",
    "contactType": "customer support",
    "email": "support@acme.com"
  },
  "sameAs": [
    "https://twitter.com/acmecorp",
    "https://linkedin.com/company/acmecorp"
  ]
}
```

## WebSite (homepage — enables sitelinks search)

```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "Acme Corp",
  "url": "https://acme.com",
  "potentialAction": {
    "@type": "SearchAction",
    "target": {
      "@type": "EntryPoint",
      "urlTemplate": "https://acme.com/search?q={search_term_string}"
    },
    "query-input": "required name=search_term_string"
  }
}
```

## BreadcrumbList (any page with breadcrumbs)

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://acme.com"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Blog",
      "item": "https://acme.com/blog"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "Post Title",
      "item": "https://acme.com/blog/post-slug"
    }
  ]
}
```

## Recipe

```json
{
  "@context": "https://schema.org",
  "@type": "Recipe",
  "name": "Recipe Name",
  "description": "Brief description",
  "image": "https://acme.com/recipes/img.jpg",
  "author": { "@type": "Person", "name": "Chef Name" },
  "datePublished": "2024-01-01",
  "prepTime": "PT15M",
  "cookTime": "PT30M",
  "totalTime": "PT45M",
  "recipeYield": "4 servings",
  "recipeIngredient": ["2 cups flour", "1 cup sugar"],
  "recipeInstructions": [
    { "@type": "HowToStep", "text": "Preheat oven to 350°F." },
    { "@type": "HowToStep", "text": "Mix dry ingredients." }
  ],
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.8",
    "ratingCount": "45"
  }
}
```

## Event

```json
{
  "@context": "https://schema.org",
  "@type": "Event",
  "name": "Event Name",
  "description": "Event description",
  "startDate": "2024-06-15T18:00:00Z",
  "endDate": "2024-06-15T21:00:00Z",
  "eventStatus": "https://schema.org/EventScheduled",
  "eventAttendanceMode": "https://schema.org/OfflineEventAttendanceMode",
  "location": {
    "@type": "Place",
    "name": "Venue Name",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "123 Main St",
      "addressLocality": "San Francisco",
      "addressRegion": "CA",
      "postalCode": "94105",
      "addressCountry": "US"
    }
  },
  "organizer": {
    "@type": "Organization",
    "name": "Acme Corp",
    "url": "https://acme.com"
  },
  "offers": {
    "@type": "Offer",
    "url": "https://acme.com/events/event-slug",
    "price": "50",
    "priceCurrency": "USD",
    "availability": "https://schema.org/InStock"
  }
}
```

## WebPage (generic pages — use when no specific type fits)

```json
{
  "@context": "https://schema.org",
  "@type": "WebPage",
  "name": "Page Title",
  "description": "Page description",
  "url": "https://acme.com/page-slug",
  "breadcrumb": {
    "@type": "BreadcrumbList",
    "itemListElement": [
      { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://acme.com" },
      { "@type": "ListItem", "position": 2, "name": "Page Title", "item": "https://acme.com/page-slug" }
    ]
  }
}
```

## Multiple schemas on one page

Wrap in an array when a page needs more than one schema type (e.g., Article + BreadcrumbList):

```json
[
  {
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "..."
  },
  {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [...]
  }
]
```
