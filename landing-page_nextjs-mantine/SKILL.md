---
name: landing-page
description: >
  Build production-ready landing pages with Next.js App Router and Mantine v7+.
  Activates when building, creating, or developing a landing page, marketing page,
  or public-facing product page. Covers scaffolding, section composition, SEO,
  performance, accessibility, responsive design, animations, testing, and deployment.
---

## Tech Stack

- **Framework**: Next.js (App Router)
- **UI Library**: Mantine v7+ (`@mantine/core`, `@mantine/hooks`)
- **Styling**: Mantine theme system + CSS Modules with PostCSS (`postcss-preset-mantine`, `postcss-simple-vars`)
- **Animation**: Framer Motion (`framer-motion`) with `LazyMotion` + `MotionConfig reducedMotion="user"`
- **Icons**: `@tabler/icons-react`
- **Language**: TypeScript (strict mode)
- **Formatter**: `deno fmt` (line width 120)
- **Package Manager**: pnpm

## Project Structure

```
app/
  layout.tsx              # Root layout: MantineProvider, ColorSchemeScript, global metadata
  page.tsx                # Landing page entry (or per-route if multi-page)
  globals.css             # Global style overrides (minimal)
  robots.ts               # Dynamic robots.txt generation
  sitemap.ts              # Dynamic sitemap generation
  favicon.ico
  opengraph-image.png     # Default OG image (1200x630)
components/
  layout/
    Header.tsx            # Sticky navigation bar (required)
    Footer.tsx            # Footer with legal links (required)
  sections/
    Hero.tsx              # Hero section (required)
    [SectionName].tsx     # One file per section chosen from the catalog
  ui/
    MotionProvider.tsx     # 'use client' — LazyMotion + MotionConfig wrapper
lib/
  theme.ts                # Mantine createTheme configuration
  motion.ts               # Framer Motion shared variants and transitions (no 'use client' — pure constants)
  metadata.ts             # Shared metadata helpers
  structured-data.ts      # JSON-LD generators
public/
  fonts/                  # Self-hosted WOFF2 font files
  images/                 # Static images (prefer next/image optimization)
postcss.config.cjs        # PostCSS config for Mantine
next.config.mjs           # Next.js config with optimizePackageImports
```

## Setup Requirements

### Dependencies

```bash
pnpm add @mantine/core @mantine/hooks @tabler/icons-react framer-motion
pnpm add -D postcss postcss-preset-mantine postcss-simple-vars
```

Install additional Mantine packages only when needed:
- `@mantine/form` — for contact or lead capture forms
- `@mantine/notifications` — for toast feedback on form submission
- `@mantine/carousel` — for testimonial carousels (requires `embla-carousel-react`)

### PostCSS Configuration

`postcss.config.cjs`:
```js
module.exports = {
  plugins: {
    'postcss-preset-mantine': {},
    'postcss-simple-vars': {
      variables: {
        'mantine-breakpoint-xs': '36em',
        'mantine-breakpoint-sm': '48em',
        'mantine-breakpoint-md': '62em',
        'mantine-breakpoint-lg': '75em',
        'mantine-breakpoint-xl': '88em',
      },
    },
  },
};
```

### Next.js Configuration

`next.config.mjs`:
```js
export default {
  experimental: {
    optimizePackageImports: ['@mantine/core', '@mantine/hooks'],
  },
};
```

### Root Layout

`app/layout.tsx` must include:
- `import '@mantine/core/styles.layer.css'` — use layer variant for easy style overriding
- `mantineHtmlProps` spread onto `<html>` tag
- `<ColorSchemeScript defaultColorScheme="auto" />` in `<head>` — prevents flash of unstyled content
- `<MantineProvider theme={theme} defaultColorScheme="auto">` wrapping `{children}` in `<body>`
- `lang` attribute on `<html>` matching the page language

## Theme Configuration

Define the theme in `lib/theme.ts` using `createTheme()`. The theme is the single source of truth for visual design.

```ts
import { createTheme } from '@mantine/core';

export const theme = createTheme({
  primaryColor: 'brand',          // use a custom color tuple
  colors: {
    brand: [/* 10 shades from lightest to darkest */],
  },
  fontFamily: 'Inter, system-ui, sans-serif',
  headings: {
    fontFamily: 'Inter, system-ui, sans-serif',
    fontWeight: '700',
  },
  defaultRadius: 'md',
  cursorType: 'pointer',
  respectReducedMotion: true,
  autoContrast: true,
});
```

### Theme Rules

- All colors must come from the theme palette — never use hardcoded hex/rgb in components
- Use Mantine's responsive prop syntax for breakpoint-dependent values: `{{ base: 1, sm: 2, lg: 3 }}`
- Spacing, font sizes, and radii must reference theme tokens, not arbitrary pixel values
- Dark mode must be supported via `defaultColorScheme="auto"` and tested for both schemes
- Use CSS module `@mixin dark { ... }` or `@mixin light { ... }` for scheme-specific overrides

## Page Composition

A landing page is composed from **required scaffolding** (Header, Hero, Footer) plus **catalog sections** chosen per project. Before scaffolding, the agent must ask the user which catalog sections to include and in what order.

### Required Scaffolding

These are always present on every landing page. They are not optional.

#### Header (Sticky Navigation)

- Sticky at the top of the viewport
- Contains: logo (links to `/`), navigation links, primary CTA button
- Mobile: collapse navigation into a hamburger menu using Mantine `Burger` + `Drawer`
- Use `AppShell.Header` or a simple `Box` with `pos="sticky"` and `top={0}`
- Must have `z-index` above all page content
- Background should include a subtle backdrop blur or solid background so content beneath doesn't bleed through
- Keep navigation minimal — a landing page should minimize exit paths

#### Hero Section

- Full-width or contained; occupies the majority of the first viewport
- Contains:
  - **Badge** (optional): short category label above the headline (e.g., "Now in Beta")
  - **Headline** (`<h1>`): outcome-focused, not feature-focused. Use `Title order={1}` with large `size`
  - **Subheadline**: 1-2 sentences elaborating the value proposition. Use `Text size="xl" c="dimmed"`
  - **Primary CTA button**: action verb + benefit (e.g., "Start Free Trial"). Use `Button size="xl"`
  - **Secondary CTA** (optional): lower-commitment action (e.g., "Watch Demo"). Use `Button variant="outline"`
  - **Hero image or visual**: product screenshot, illustration, or short video
- The hero image is the LCP element — it must:
  - Use `next/image` with `priority={true}` (sets `fetchpriority="high"` and disables lazy loading)
  - Have explicit `width` and `height` to prevent CLS
- Layout: typically two-column on desktop (text left, image right) collapsing to stacked on mobile
- Optional trust signal below CTAs: "Trusted by 10,000+ teams" or a small logo strip

#### Footer

- Contains:
  - Company logo and brief tagline
  - Navigation columns (e.g., Product, Company, Resources, Legal)
  - Legal links (always required): Privacy Policy, Terms of Service, Cookie Policy / Manage Preferences
  - Social media icon links (open in new tab with `target="_blank" rel="noopener noreferrer"`)
  - Copyright notice: `© {currentYear} {CompanyName}. All rights reserved.`
- Keep footer navigation minimal on single-page landing pages
- Accessibility statement link is best practice for public-facing pages

### Section Catalog

The following archetypes are available. For full specs and responsive tables, see [references/sections-catalog.md](references/sections-catalog.md).

| Archetype | Purpose |
|-----------|---------|
| `grid-cards` | Responsive grid of identical cards (features, benefits, services) |
| `split-content` | Two-column text + image layout (product showcase, about us) |
| `logo-bar` | Horizontal row of trust logos/badges |
| `stats-bar` | Row of large numbers with labels (metrics, social proof) |
| `testimonials` | Attributed quotes from real users |
| `pricing` | Side-by-side pricing plan cards |
| `accordion-list` | Expandable Q&A using Mantine Accordion (FAQ) |
| `cta-block` | Full-width call-to-action band |
| `steps` | Numbered sequential process flow |
| `form-capture` | Inline lead capture form |
| `video-embed` | Embedded video with lazy-loaded player |

### Composition Rules

- **Required order**: Header → Hero → [catalog sections in user-chosen order] → Footer
- **At least one `cta-block`** should appear somewhere after the main content sections (before Footer)
- Each section component gets its own file: `components/sections/{SectionName}.tsx`
- Each section wraps content in a `<section>` element with `data-section="{kebab-name}"` and `aria-labelledby` referencing its heading
- Section headings use `<h2>` (Hero already uses the page's only `<h1>`)

### Default Templates

When the user's request doesn't specify sections, suggest a default composition:

- **SaaS product**: Hero → logo-bar → grid-cards (features) → split-content (product detail) → testimonials → pricing → accordion-list (FAQ) → cta-block
- **Lead generation**: Hero → logo-bar → grid-cards (benefits) → split-content → form-capture → testimonials → accordion-list (FAQ)
- **Event / launch**: Hero → stats-bar → split-content → steps → video-embed → cta-block
- **Simple / minimal**: Hero → grid-cards → cta-block

## Performance Requirements

### Core Web Vitals Targets

| Metric | Target |
|--------|--------|
| LCP (Largest Contentful Paint) | < 2.5s |
| INP (Interaction to Next Paint) | < 200ms |
| CLS (Cumulative Layout Shift) | < 0.1 |

### Image Optimization

- Always use `next/image` (`<Image>`) — it handles format conversion, responsive sizing, and lazy loading
- Hero image: set `priority={true}` — this is the LCP element. Never lazy-load it
- All other images: `next/image` applies `loading="lazy"` by default
- Always provide explicit `width` and `height` props to prevent CLS
- Store large images in `public/images/`; prefer WebP or AVIF source files when available

### Font Loading

- Self-host fonts in `public/fonts/` using WOFF2 format
- Use `next/font/local` or `next/font/google` for automatic optimization
- Use `font-display: swap` (Next.js does this by default with `next/font`)
- Preload only the font weights used above the fold

### JavaScript

- Minimize client-side JavaScript — landing pages should be mostly static
- Mark components as `'use client'` only when they need interactivity (accordion, mobile menu, theme toggle)
- Defer analytics and third-party scripts: load only after consent is granted
- Use Next.js `<Script strategy="lazyOnload">` for non-critical third-party scripts

### CSS

- Use `@mantine/core/styles.layer.css` so Mantine styles sit in a CSS layer and custom styles override without specificity battles
- Keep CSS Modules scoped to their components — avoid global styles except for truly global resets

### Budget

| Resource | Budget |
|----------|--------|
| Total JS (transferred) | < 250KB |
| Total CSS (transferred) | < 80KB |
| Total Fonts (transferred) | < 150KB |

## Accessibility Requirements (WCAG 2.1 AA)

### Color and Contrast

- Normal text (< 18px): minimum **4.5:1** contrast ratio against background
- Large text (>= 18px regular / >= 14px bold): minimum **3:1** contrast ratio
- UI components and interactive elements: minimum **3:1** contrast ratio
- Color alone must never convey information — always pair with text, icon, or pattern
- Mantine's `autoContrast: true` in theme helps but must be verified

### Keyboard Navigation

- All interactive elements must be operable by keyboard alone
- Tab order must match visual reading order
- No keyboard traps — users must be able to Tab away from any component
- Visible focus indicators on all focusable elements (Mantine provides this by default with `focusRing: 'auto'`)
- Add a skip navigation link as the first focusable element: `<a href="#main-content">Skip to main content</a>`
- Modal and drawer must trap focus within them when open and return focus when closed

### Semantic HTML and ARIA

- Use semantic landmarks: `<header>`, `<nav>`, `<main>`, `<section>`, `<footer>`
- Add `aria-label` to `<nav>` elements when there are multiple navs (e.g., "Main navigation", "Footer navigation")
- All images must have `alt` attributes (meaningful or `alt=""` for decorative)
- All form inputs must have explicit `<label>` elements — placeholder-only is insufficient
- Use `<button>` for actions and `<a>` for navigation — do not interchange
- Links opening in new tabs must indicate this: add `aria-label` or visible icon

### Interactive Elements

- Minimum touch/click target size: **44x44px** (WCAG 2.5.5)
- Recommended target size: **48x48px** on mobile
- Minimum spacing between adjacent interactive elements: **8px**
- No content that flashes more than 3 times per second

## Code Formatting

All code is formatted with `deno fmt` at line width 120. Format before committing.

```bash
# Format all files
pnpm fmt

# Format in watch mode during development
pnpm fmt:watch
```

Rules:
- Line width: 120 characters
- `deno fmt` handles indentation (2 spaces), semicolons, quote style, and trailing commas automatically
- Do not add Prettier or ESLint formatting rules — `deno fmt` is the sole formatter
- Run `pnpm fmt` before committing or after generating new files

## Component Conventions

### File Naming

- Components: PascalCase (`Hero.tsx`, `SocialProof.tsx`)
- CSS Modules: match component name (`Hero.module.css`)
- Utilities/helpers: kebab-case (`structured-data.ts`, `metadata.ts`)

### Component Structure

```tsx
// components/sections/Hero.tsx
import { Container, Title, Text, Button, Group, Stack } from '@mantine/core';
import Image from 'next/image';
import classes from './Hero.module.css';

export function Hero() {
  return (
    <section className={classes.root} data-section="hero" aria-labelledby="hero-heading">
      <Container size="lg">
        {/* content */}
      </Container>
    </section>
  );
}
```

### Rules

- Every section component wraps its content in a `<section>` element with an `aria-labelledby` referencing its heading
- Use Mantine components for all UI elements — do not write raw HTML for things Mantine provides
- Use `next/image` for all images, `next/link` for all internal links
- Mantine's `Button` with `component={Link}` for CTA links that navigate
- Mark files `'use client'` only when they use browser APIs, hooks, or interactive Mantine components (Accordion, Burger, Drawer, etc.)
- Keep section components focused — they should compose Mantine primitives, not contain business logic
- Use CSS Modules for custom styles beyond what Mantine props provide
- Use `@mixin dark { ... }` and `@mixin light { ... }` in CSS Modules for color-scheme-specific styles
- All text content should be easy to replace — avoid deeply nesting strings in logic

## Scaffolding Completion Checklist

After generating all project files, the agent **must** complete these steps in order before presenting the project to the user:

1. **Install all dependencies** — `pnpm add` production deps, then `pnpm add -D` all dev deps including testing packages
2. **Install test dependencies** — testing is not optional; install vitest, Playwright, and all test-related packages listed in the [Testing reference](references/testing.md)
3. **Install Playwright browsers** — `pnpm exec playwright install --with-deps chromium`
4. **Create test infrastructure** — vitest.config.ts, playwright.config.ts, `__tests__/setup.ts`, `__tests__/test-utils.tsx`, and test files for every component
5. **Create e2e test files** — `e2e/responsive.spec.ts`, `e2e/accessibility.spec.ts`, `e2e/performance.spec.ts`, `e2e/visual.spec.ts`
6. **Add all scripts to package.json** — including all test scripts listed in the [Testing reference](references/testing.md)
7. **Run `pnpm fmt`** — format all generated files with deno fmt before presenting the project
8. **Run `pnpm build`** — verify the production build succeeds with zero errors
9. **Run `pnpm test`** — verify all unit tests pass

Do not skip any of these steps. Do not defer test setup to "later". A project is not complete without working tests and clean formatting.

## References

Detailed specifications are split into focused reference files. Load these as needed during implementation:

- **[Section Catalog](references/sections-catalog.md)** — Full specs for all 10 section archetypes with responsive tables and layout patterns
- **[Responsive Design](references/responsive-design.md)** — Breakpoints, fluid typography, fluid spacing, per-section responsive specs, responsive images, touch targets, testing checklist
- **[Animation & Visual Effects](references/animation-and-effects.md)** — Framer Motion setup, lib/motion.ts variants, scroll-triggered animations, per-archetype animation map, visual effects catalog (parallax, mesh-gradient, glassmorphism, text-reveal, bento-grid, and more)
- **[SEO, Legal & Security](references/seo-legal-security.md)** — Metadata exports, JSON-LD structured data, sitemap/robots, legal compliance (GDPR cookie consent, legal pages), analytics integration, security headers
- **[Testing](references/testing.md)** — Vitest + React Testing Library setup, Playwright config (8 viewport projects), unit/component test specs per archetype, responsive/visual/a11y/performance E2E tests, Lighthouse local audit, package.json scripts
- **[Pre-Launch Checklist](references/pre-launch-checklist.md)** — Complete verification checklist covering content, SEO, performance, accessibility, responsive, legal, analytics, animation, visual effects, testing, and cross-browser
