# SEO, Legal, Analytics & Security Reference

## SEO Requirements

### Metadata

Use Next.js `metadata` export in `app/layout.tsx` and `app/page.tsx`:

```ts
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Primary Keyword | Brand Name',
  description: '150-160 character compelling description with primary keyword',
  metadataBase: new URL('https://example.com'),
  alternates: { canonical: '/' },
  openGraph: {
    type: 'website',
    title: 'Primary Keyword | Brand Name',
    description: 'Same or similar to meta description',
    url: '/',
    siteName: 'Brand Name',
    images: [{ url: '/opengraph-image.png', width: 1200, height: 630, alt: 'Description' }],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Primary Keyword | Brand Name',
    description: 'Same or similar to meta description',
    images: ['/opengraph-image.png'],
  },
  robots: { index: true, follow: true },
};
```

### Structured Data (JSON-LD)

Add JSON-LD via `<script type="application/ld+json">` in the page or layout. Create helper functions in `lib/structured-data.ts`:

Required schemas:
- **WebSite** + **Organization**: on every page
- **FAQPage**: on any page that includes an `accordion-list` section used for FAQ content

Rules:
- Data must match visible page content exactly — no hidden or phantom markup
- Validate with Google's Rich Results Test before shipping

### Sitemap and Robots

Use Next.js file-based conventions:
- `app/sitemap.ts` — export a `sitemap()` function returning page entries
- `app/robots.ts` — export a `robots()` function; never block CSS or JS files

### Heading Hierarchy

- Exactly one `<h1>` per page (the hero headline)
- Heading levels must not skip: h1 → h2 → h3 (no jumping from h1 to h3)
- Each section title uses `<h2>`, sub-items within sections use `<h3>`

## Legal Compliance

### Cookie Consent (GDPR / ePrivacy)

- Block all non-essential scripts (analytics, marketing pixels) until user grants consent
- Cookie banner must include:
  - "Accept All" and "Reject All" buttons with **equal visual prominence** (same size, same color weight)
  - "Manage Preferences" button/link leading to granular category controls
  - Brief description of what cookies are used for
- No dark patterns: do not hide "Reject All" behind a submenu
- Banner itself must be accessible: keyboard navigable, screen reader compatible, meets contrast requirements
- Store consent state and honor it on subsequent visits
- Strictly necessary cookies (session, CSRF) do not require consent

### Required Legal Pages

Link from the footer on every page:
- **Privacy Policy**: what data is collected, why, how long it's retained, user rights (GDPR Art. 13/14)
- **Terms of Service**: governing terms for using the product/website
- **Cookie Policy**: detailed cookie inventory (can be part of Privacy Policy)

### Accessibility Statement

Best practice for public-facing pages. Include:
- Conformance target (WCAG 2.1 AA)
- Known limitations
- Contact method for accessibility issues

## Analytics Integration

### Implementation Rules

- Analytics scripts must only fire **after** user consents to analytics cookies
- Use Next.js `<Script strategy="lazyOnload">` for the analytics tag
- Implement GA4 Consent Mode v2 for EU traffic if using Google Analytics:
  - Set `analytics_storage` to `'denied'` by default
  - Update to `'granted'` when consent is given

### UTM Parameters

- All external campaign links must use UTM parameters
- Naming convention: always lowercase, use underscores for spaces
- Standard parameters: `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`
- Never use UTM parameters on internal links — they overwrite session attribution
- Optionally persist UTM values in `sessionStorage` on page load for form attribution

### Event Tracking

Track key conversion events:
- Primary CTA clicks
- Form submissions (lead capture)
- Scroll depth milestones (25%, 50%, 75%, 100%)
- Pricing plan selection
- External link clicks

## Security Headers

Configure these headers in `next.config.mjs` via the `headers()` function or at the hosting/CDN layer:

```js
// next.config.mjs — merge with existing config (optimizePackageImports, etc.)
export default {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload',
          },
          {
            key: 'Content-Security-Policy',
            value: "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self';",
          },
        ],
      },
    ];
  },
};
```

Adjust CSP directives when adding third-party services (analytics, fonts, CDN images). Start with `Content-Security-Policy-Report-Only` in staging.
