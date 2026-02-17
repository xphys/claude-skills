# Responsive Design Reference

This is a public-facing landing page — responsive quality is critical. Every section must look polished from 320px to 2560px. There is no "desktop version" that gets adapted to mobile. The mobile layout is the primary design; wider viewports enhance it.

## Approach

**Mobile-first, always.** Base styles target 320px. Wider layouts are layered on via `min-width` breakpoints using Mantine's responsive prop syntax. Never use `max-width` media queries.

## Viewport

- Next.js sets `<meta name="viewport" content="width=device-width, initial-scale=1.0" />` by default — do not override
- Never use `user-scalable=no` or `maximum-scale=1` — these violate WCAG 1.4.4 and block users who need to zoom
- Test that no horizontal scrollbar appears at any width from 320px to 2560px

## Breakpoints (Mantine defaults)

| Name | Value | Typical Use |
|------|-------|-------------|
| `xs` | 36em (576px) | Large phones |
| `sm` | 48em (768px) | Tablets |
| `md` | 62em (992px) | Small desktops |
| `lg` | 75em (1200px) | Desktops |
| `xl` | 88em (1408px) | Wide desktops |

Use Mantine's responsive prop objects everywhere instead of writing manual media queries:

```tsx
<SimpleGrid cols={{ base: 1, sm: 2, lg: 3 }} spacing={{ base: 'md', sm: 'lg', lg: 'xl' }}>
```

## Fluid Typography

Do not use fixed font sizes for headings. Use `clamp()` to scale smoothly between breakpoints:

```css
/* Hero headline */
.heroTitle { font-size: clamp(2rem, 5vw + 1rem, 4rem); }

/* Section headings */
.sectionTitle { font-size: clamp(1.5rem, 3vw + 0.5rem, 2.5rem); }

/* Body text stays fixed for readability */
.body { font-size: 1rem; } /* 16px — never smaller on mobile */
```

Rules:
- Body text must never be smaller than `1rem` (16px) on any viewport
- Headings scale fluidly — no jarring size jumps between breakpoints
- Line height for headings: `1.1` – `1.2`; for body text: `1.5` – `1.7`
- Maximum content line length: `65ch` – `75ch` for readability (use `maw` prop on `Text`)

## Fluid Spacing

Section vertical padding and component gaps should scale with viewport:

```css
/* Section padding */
.section { padding-block: clamp(3rem, 8vw, 6rem); }

/* Inner spacing */
.sectionInner { gap: clamp(1.5rem, 4vw, 3rem); }
```

Or use Mantine responsive props:
```tsx
<Stack gap={{ base: 'lg', sm: 'xl', lg: 48 }}>
```

Rules:
- Sections need generous vertical breathing room — at least `3rem` on mobile, `5rem`+ on desktop
- Never let content touch the screen edges — use `Container` with `px={{ base: 'md', sm: 'lg', lg: 'xl' }}`
- Consistent spacing rhythm: pick a scale (e.g., 8px base) and stick to multiples

## Per-Section Responsive Specs

Responsive breakpoint tables for each section archetype are defined inline in the [Section Catalog](sections-catalog.md). In addition, the required scaffolding has these responsive specs:

### Header / Navigation
| Viewport | Layout |
|----------|--------|
| `< sm` | Logo left, `Burger` right. Navigation in full-screen `Drawer`. CTA button inside drawer. |
| `sm` – `md` | Logo left, horizontal nav links center/right, CTA button right. Compact spacing. |
| `> md` | Logo left, horizontal nav links center, CTA button right. Comfortable spacing. |

- Sticky header height: `60px` mobile, `70px` desktop — account for this with `scroll-padding-top` on `<html>`
- Drawer opens from right, full viewport height, closes on link click, outside click, or Escape
- Burger button: minimum 44x44px touch target

### Hero Section
| Viewport | Layout |
|----------|--------|
| `< md` | Single column, stacked: badge → headline → subheadline → CTA buttons → hero image below. Text centered. |
| `>= md` | Two columns via `Grid`: text left (span 6-7), image right (span 5-6). Text left-aligned. |

- Hero headline: `clamp(2rem, 5vw + 1rem, 4rem)`, `fw={900}`
- CTA buttons: full width (`fullWidth`) on mobile, inline on desktop
- If two CTAs, stack them vertically on `< xs`, side by side on `>= xs`:
  ```tsx
  <Group gap="sm" grow={{ base: true }} wrap={{ base: 'wrap', xs: 'nowrap' }}>
  ```
- Hero image: maintain aspect ratio, use `sizes="(max-width: 992px) 100vw, 50vw"` for responsive loading

### Footer
| Viewport | Layout |
|----------|--------|
| `< sm` | Stacked: logo/tagline → nav columns (stacked, each full width) → legal links → copyright |
| `sm` – `md` | 2-column grid for nav columns |
| `>= md` | Logo/tagline left, nav columns in a row right. Legal links and copyright at bottom. |

```tsx
<SimpleGrid cols={{ base: 1, sm: 2, md: 4 }} spacing="lg">
```

- Social media icons: `Group gap="xs"` — always horizontal, never wrap
- Legal links: `Group gap="md"` wrapping on mobile, single row on desktop
- Footer padding: generous `py={{ base: 'xl', sm: 48, lg: 64 }}`

## Responsive Images

- Always use `next/image` with the `sizes` prop to serve correctly-sized images:
  ```tsx
  // Full-width image
  <Image src={src} alt={alt} sizes="100vw" />

  // Hero image (full width mobile, half on desktop)
  <Image src={src} alt={alt} sizes="(max-width: 992px) 100vw, 50vw" />

  // Grid card image (1 col mobile, 2 col tablet, 3 col desktop)
  <Image src={src} alt={alt} sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw" />
  ```
- Use `fill` + `style={{ objectFit: 'cover' }}` for images that must fill a container
- Decorative background images: use CSS `background-image` with responsive `image-set()` or hide on mobile entirely if non-essential

## Touch Targets

- All interactive elements: minimum **44x44px** touch target
- Recommended: **48x48px** on mobile
- Adjacent interactive elements: minimum **8px** gap to prevent mis-taps
- CTA buttons on mobile: full container width, minimum height `48px`
- Navigation links in mobile drawer: full width, minimum height `48px`, with visible separator between items
- Social media icons in footer: wrap each in a container that's at least 44x44px even if the icon itself is smaller

## Responsive Testing Checklist

Test every page at these exact widths:
- **320px** — smallest supported phone (iPhone SE)
- **375px** — standard phone (iPhone 12/13/14)
- **414px** — larger phone (iPhone Plus/Max)
- **768px** — tablet portrait (iPad)
- **1024px** — tablet landscape / small desktop
- **1280px** — standard desktop
- **1440px** — large desktop
- **1920px** — full HD
- **2560px** — ultra-wide (verify content doesn't stretch beyond readable width)

At each width verify:
- [ ] No horizontal scrollbar
- [ ] No text overflow or truncation (unless intentional with ellipsis)
- [ ] No overlapping elements
- [ ] All touch targets meet minimum size
- [ ] Images are sharp and correctly sized (not stretched or cropped incorrectly)
- [ ] Spacing feels proportional — neither too cramped nor too sparse
- [ ] Text is readable without zooming (minimum 16px body text)
- [ ] Interactive elements are reachable and usable
