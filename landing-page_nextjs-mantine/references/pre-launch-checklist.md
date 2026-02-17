# Pre-Launch Checklist

Before considering the landing page complete, verify:

## Content
- [ ] All placeholder/lorem ipsum text replaced with real copy
- [ ] All images are final assets (not placeholders)
- [ ] Links point to correct destinations
- [ ] Contact information is accurate

## SEO
- [ ] `<title>` and `<meta name="description">` set with keywords
- [ ] Open Graph and Twitter Card meta tags present
- [ ] JSON-LD structured data validates (Google Rich Results Test)
- [ ] `sitemap.ts` and `robots.ts` configured
- [ ] Canonical URL set
- [ ] Single `<h1>`, heading hierarchy correct

## Performance
- [ ] Lighthouse Performance score >= 90 (run `pnpm test:lighthouse`)
- [ ] LCP < 2.5s
- [ ] CLS < 0.1 (all images have dimensions, no layout shifts)
- [ ] Hero image uses `priority={true}` in `next/image`
- [ ] Fonts loaded via `next/font` with swap display
- [ ] No render-blocking third-party scripts
- [ ] JS bundle < 250KB, CSS < 80KB, Fonts < 150KB

## Accessibility
- [ ] Passes axe DevTools automated scan with zero violations
- [ ] Full keyboard navigation works (Tab through entire page)
- [ ] Skip navigation link present and functional
- [ ] Color contrast meets WCAG AA (4.5:1 normal text, 3:1 large text)
- [ ] All images have appropriate `alt` attributes
- [ ] All form inputs have `<label>` elements
- [ ] Focus indicators visible on all interactive elements

## Responsive
- [ ] Looks correct at 320px, 768px, 1024px, 1440px widths
- [ ] Mobile navigation (hamburger + drawer) works correctly
- [ ] Touch targets are at least 44x44px on mobile
- [ ] No horizontal scrolling at any viewport size

## Legal & Security
- [ ] Cookie consent banner present and functional (if using analytics/tracking)
- [ ] Privacy Policy and Terms of Service pages linked from footer
- [ ] Security headers configured (HSTS, CSP, X-Frame-Options, etc.)
- [ ] No sensitive data exposed in client-side code

## Analytics
- [ ] Analytics fires only after consent is granted
- [ ] Key conversion events tracked (CTA clicks, form submissions)
- [ ] UTM parameter handling tested

## Animation
- [ ] `MotionProvider` with `reducedMotion="user"` wrapping the app
- [ ] `LazyMotion features={domAnimation}` in place for bundle optimization
- [ ] All scroll-triggered animations use `viewport={{ once: true }}`
- [ ] Hero animation completes within 1s total
- [ ] No animations cause layout shift (only `opacity` and `transform` animated)
- [ ] Animations disabled gracefully with `prefers-reduced-motion: reduce`
- [ ] Stagger groups limited to 8 items or fewer
- [ ] Smooth at 4x CPU throttle in Chrome DevTools

## Visual Effects
- [ ] Max 2-3 visual effects per page (less is more)
- [ ] All effects degrade gracefully on mobile (simplified or disabled)
- [ ] `prefers-reduced-motion: reduce` disables all scroll-linked and background effects
- [ ] No effect causes layout shift or reflow (only `transform` and `opacity`)
- [ ] Scroll-linked effects use `useScroll` + `useTransform` (no scroll event listeners)
- [ ] Sticky-scroll and horizontal-scroll sections have correct height spacers
- [ ] Background effects (mesh-gradient, aurora, glow) use `will-change: transform` or `contain: paint`
- [ ] Marquee pauses on hover and respects `prefers-reduced-motion`
- [ ] Counter animations trigger only once via `whileInView` with `once: true`
- [ ] Effects maintain 60fps at 4x CPU throttle in Chrome DevTools
- [ ] All effects tested at 320px, 768px, and 1440px viewports

## Testing
- [ ] `pnpm test` passes — all unit/component tests green
- [ ] `pnpm test:e2e` passes — all Playwright viewport projects green
- [ ] `pnpm test:a11y` passes — axe audit zero violations at all viewports
- [ ] `pnpm test:perf` passes — CWV budgets met
- [ ] Visual snapshots up to date (`pnpm test:visual`)
- [ ] Lighthouse report reviewed (run `pnpm test:lighthouse` with production server running)

## Cross-Browser
- [ ] Tested in Chrome, Firefox, Safari (minimum)
- [ ] Tested on at least one real mobile device or BrowserStack
- [ ] Dark mode renders correctly
