# Testing Reference

The landing page must be tested at three levels: **unit/component tests** (Vitest + React Testing Library), **visual responsive + E2E tests** (Playwright), and **performance tests** (Lighthouse local audit). All tests must pass before deployment.

## Test Stack & Dependencies

```bash
# Unit & component testing
pnpm add -D vitest @vitejs/plugin-react @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom

# E2E, visual responsive, and accessibility testing
pnpm add -D @playwright/test @axe-core/playwright

# Performance testing (Lighthouse runs via pnpm dlx — no install needed)

# Accessibility testing (unit tests)
pnpm add -D vitest-axe
```

## Project Test Structure

```
__tests__/
  components/
    sections/
      Hero.test.tsx             # Required — always present
      [SectionName].test.tsx    # One test file per chosen catalog section
    layout/
      Header.test.tsx           # Required — always present
      Footer.test.tsx           # Required — always present
  lib/
    structured-data.test.ts
    metadata.test.ts
e2e/
  landing-page.spec.ts         # Full page E2E flow
  responsive.spec.ts           # Multi-viewport visual tests
  accessibility.spec.ts        # Automated a11y audit per viewport
  performance.spec.ts          # CWV measurement via Playwright
vitest.config.ts               # Vitest config
playwright.config.ts           # Playwright config
```

## Vitest Configuration

`vitest.config.ts`:
```ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./__tests__/setup.ts'],
    include: ['__tests__/**/*.test.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      include: ['components/**', 'lib/**'],
      exclude: ['**/*.d.ts', '**/*.test.*'],
      thresholds: {
        statements: 80,
        branches: 75,
        functions: 80,
        lines: 80,
      },
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, '.'),
    },
  },
});
```

`__tests__/setup.ts`:
```ts
import '@testing-library/jest-dom/vitest';
import { vi } from 'vitest';

// Mock next/image — renders as a plain <img> in tests
vi.mock('next/image', () => ({
  default: (props: React.ImgHTMLAttributes<HTMLImageElement>) => {
    // eslint-disable-next-line @next/next/no-img-element
    return <img {...props} />;
  },
}));

// Mock next/link — renders as a plain <a> in tests
vi.mock('next/link', () => ({
  default: ({ children, href, ...props }: any) => (
    <a href={href} {...props}>{children}</a>
  ),
}));

// Mock framer-motion — renders children without animation in tests
// Strips motion-specific props so they don't leak onto DOM elements
const motionProps = new Set([
  'initial', 'animate', 'exit', 'variants', 'whileInView', 'whileHover',
  'whileTap', 'whileFocus', 'whileDrag', 'viewport', 'transition',
  'layout', 'layoutId', 'onAnimationStart', 'onAnimationComplete',
]);

vi.mock('framer-motion', async () => {
  const actual = await vi.importActual('framer-motion');
  return {
    ...actual,
    motion: new Proxy({}, {
      get: (_target, prop: string) => {
        return ({ children, ...props }: any) => {
          const htmlProps = Object.fromEntries(
            Object.entries(props).filter(([key]) => !motionProps.has(key))
          );
          const Element = prop as keyof React.JSX.IntrinsicElements;
          return <Element {...htmlProps}>{children}</Element>;
        };
      },
    }),
    AnimatePresence: ({ children }: any) => children,
    LazyMotion: ({ children }: any) => children,
    MotionConfig: ({ children }: any) => children,
    useReducedMotion: () => false,
    useInView: () => true,
    useScroll: () => ({ scrollYProgress: { get: () => 0 } }),
    useTransform: (_: any, __: any, output: any[]) => ({ get: () => output?.[0] }),
    useMotionValue: (v: any) => ({ get: () => v, set: () => {} }),
    useSpring: (v: any) => v,
    useMotionTemplate: (strings: TemplateStringsArray, ...values: any[]) => ({
      get: () => String.raw(strings, ...values.map((v) => v?.get?.() ?? v)),
    }),
  };
});

// Mock matchMedia for responsive hook testing
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation((query: string) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});
```

## Mantine Test Wrapper

All component tests must render within MantineProvider. Create a shared wrapper:

```ts
// __tests__/test-utils.tsx
import { MantineProvider } from '@mantine/core';
import { render, RenderOptions } from '@testing-library/react';
import { theme } from '@/lib/theme';

function AllProviders({ children }: { children: React.ReactNode }) {
  return (
    <MantineProvider theme={theme}>
      {children}
    </MantineProvider>
  );
}

export function renderWithProviders(
  ui: React.ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) {
  return render(ui, { wrapper: AllProviders, ...options });
}

// Helper to mock matchMedia for a specific breakpoint
export function mockViewport(width: number) {
  window.matchMedia = vi.fn().mockImplementation((query: string) => {
    // Parse min-width from query like "(min-width: 768px)"
    const match = query.match(/min-width:\s*(\d+)/);
    const minWidth = match ? parseInt(match[1], 10) : 0;
    return {
      matches: width >= minWidth,
      media: query,
      onchange: null,
      addListener: vi.fn(),
      removeListener: vi.fn(),
      addEventListener: vi.fn(),
      removeEventListener: vi.fn(),
      dispatchEvent: vi.fn(),
    };
  });
}

export * from '@testing-library/react';
export { default as userEvent } from '@testing-library/user-event';
```

## Unit & Component Test Requirements

### What to Test per Section Component

Every section component must have tests covering:

1. **Renders correctly** — component mounts without errors, key content is in the DOM
2. **Semantic HTML** — correct heading level, landmark elements, aria attributes
3. **Accessibility** — passes axe automated audit, images have alt text, buttons are labeled
4. **Responsive behavior** — mock different viewports and assert conditional rendering (mobile nav vs desktop nav, stacked vs grid layout)
5. **User interaction** — clicks, accordion toggles, form submissions

### Required Scaffolding Test Specs

**Header:**
```ts
// Must test:
- Renders logo linking to "/"
- Renders navigation links
- Renders CTA button
- At mobile viewport: hamburger button is visible, nav links are hidden
- At desktop viewport: nav links are visible, hamburger is hidden
- Hamburger click opens drawer with nav links
- Drawer closes on link click
- Has role="banner" or <header> landmark
- Has aria-label on <nav>
```

**Hero:**
```ts
// Must test:
- Renders h1 headline
- Renders subheadline text
- Renders primary CTA button
- CTA button has correct href or onClick
- Hero image renders with alt text and priority attribute
- Only one h1 exists in the component
```

**Footer:**
```ts
// Must test:
- Renders company logo/name
- Renders Privacy Policy link
- Renders Terms of Service link
- External links have target="_blank" and rel="noopener noreferrer"
- Social media links are present and accessible
- Copyright text includes current year
- Has role="contentinfo" or <footer> landmark
```

### Per-Archetype Test Specs

For each catalog section chosen, apply the test requirements for its archetype:

**`grid-cards`:**
```ts
- Renders all items passed via props
- Each item has a title and description
- Titles use correct heading level (h3)
- At mobile: renders in single column
- At desktop: renders in multi-column grid
- Passes axe accessibility audit
```

**`split-content`:**
```ts
- Renders heading, description text, and image
- Image has alt text
- At mobile: stacks vertically (text above image)
- At desktop: two-column layout
- Passes axe accessibility audit
```

**`logo-bar`:**
```ts
- Renders all logo images
- Each logo image has alt text
- Logos are visible
```

**`stats-bar`:**
```ts
- Renders all stat items
- Each stat has a number and label
- Numbers are formatted correctly
```

**`testimonials`:**
```ts
- Renders all testimonial items
- Each testimonial has quote text, name, and role
- Avatar images have alt text
- At mobile: single column or carousel is navigable
```

**`pricing`:**
```ts
- Renders all pricing tiers
- Each tier shows: name, price, feature list, CTA button
- Recommended plan has visual distinction (badge, elevated style)
- CTA buttons have correct labels and actions
- Price values are formatted correctly
```

**`accordion-list`:**
```ts
- Renders all items
- Items are collapsed by default
- Clicking an item expands its content
- Only one item is expanded at a time (if single-expand mode)
- Content is accessible to screen readers when expanded
- Section has correct heading level (h2)
```

**`cta-block`:**
```ts
- Renders headline and CTA button
- CTA button has correct href or onClick
- Button label is benefit-driven (not "Submit")
```

**`steps`:**
```ts
- Renders all steps in correct order
- Each step has a number/icon, title, and description
- Steps are visually connected (timeline/connector)
```

**`form-capture`:**
```ts
- Renders form with required fields
- Submit button is present and labeled
- Shows validation errors for empty required fields
- Successful submission shows confirmation
- Privacy Policy link is present near submit
```

**`video-embed`:**
```ts
- Renders thumbnail/poster image
- Play button is visible and keyboard-accessible
- Video does not auto-load on page init
- Maintains 16:9 aspect ratio
```

### Responsive Unit Test Pattern

**Important limitation:** `mockViewport()` only works for components that use `useMediaQuery` from `@mantine/hooks` for conditional rendering in JavaScript. Mantine's CSS-based responsive props (`visibleFrom`, `hiddenFrom`, responsive prop objects like `cols={{ base: 1, sm: 2 }}`) are implemented via CSS media queries that jsdom does not evaluate. For comprehensive responsive testing, use **Playwright E2E tests** (see below).

To make responsive behavior testable in unit tests, use `useMediaQuery` for conditional rendering logic (e.g., show/hide burger menu) instead of relying solely on Mantine's `visibleFrom`/`hiddenFrom` CSS props.

Use `mockViewport()` to test `useMediaQuery`-based responsive rendering:

```tsx
import { renderWithProviders, screen, mockViewport } from '../test-utils';
import { Header } from '@/components/layout/Header';

describe('Header', () => {
  it('shows hamburger menu on mobile', () => {
    mockViewport(375);
    renderWithProviders(<Header />);
    expect(screen.getByRole('button', { name: /menu/i })).toBeInTheDocument();
    expect(screen.queryByRole('navigation')).not.toBeVisible();
  });

  it('shows inline navigation on desktop', () => {
    mockViewport(1280);
    renderWithProviders(<Header />);
    expect(screen.queryByRole('button', { name: /menu/i })).not.toBeInTheDocument();
    expect(screen.getByRole('navigation')).toBeVisible();
  });
});
```

### Accessibility Unit Test Pattern

Run axe on every section component:

```tsx
import { renderWithProviders } from '../test-utils';
import { axe, toHaveNoViolations } from 'vitest-axe';
import { Hero } from '@/components/sections/Hero';

expect.extend(toHaveNoViolations);

describe('Hero accessibility', () => {
  it('has no axe violations', async () => {
    const { container } = renderWithProviders(<Hero />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

## Playwright Configuration

`playwright.config.ts`:
```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html', { open: 'never' }]],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    // Mobile viewports
    {
      name: 'Mobile (320px)',
      use: { viewport: { width: 320, height: 568 } },
    },
    {
      name: 'Mobile (375px)',
      use: { ...devices['iPhone 13'] },
    },
    {
      name: 'Mobile (414px)',
      use: { ...devices['iPhone 13 Pro Max'] },
    },
    // Tablet
    {
      name: 'Tablet (768px)',
      use: { ...devices['iPad (gen 7)'] },
    },
    {
      name: 'Tablet Landscape (1024px)',
      use: { ...devices['iPad Pro 11'], viewport: { width: 1024, height: 768 } },
    },
    // Desktop
    {
      name: 'Desktop (1280px)',
      use: { viewport: { width: 1280, height: 720 } },
    },
    {
      name: 'Desktop (1440px)',
      use: { viewport: { width: 1440, height: 900 } },
    },
    {
      name: 'Full HD (1920px)',
      use: { viewport: { width: 1920, height: 1080 } },
    },
  ],
  webServer: {
    command: 'pnpm build && pnpm start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});
```

## Playwright Responsive Tests

`e2e/responsive.spec.ts` — tests that every section renders correctly at each viewport:

```ts
import { test, expect } from '@playwright/test';

test.describe('Responsive layout', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('no horizontal scrollbar', async ({ page }) => {
    const scrollWidth = await page.evaluate(() => document.documentElement.scrollWidth);
    const clientWidth = await page.evaluate(() => document.documentElement.clientWidth);
    expect(scrollWidth).toBeLessThanOrEqual(clientWidth);
  });

  test('no content overflow', async ({ page }) => {
    // Find any element that overflows the viewport
    const overflows = await page.evaluate(() => {
      const viewportWidth = document.documentElement.clientWidth;
      const allElements = document.querySelectorAll('*');
      const overflowing: string[] = [];
      allElements.forEach((el) => {
        const rect = el.getBoundingClientRect();
        if (rect.right > viewportWidth + 1 || rect.left < -1) {
          overflowing.push(`${el.tagName}.${el.className} (left: ${rect.left}, right: ${rect.right})`);
        }
      });
      return overflowing;
    });
    expect(overflows).toEqual([]);
  });

  test('hero section is visible and properly structured', async ({ page }) => {
    const hero = page.locator('section').first();
    await expect(hero).toBeVisible();
    await expect(hero.locator('h1')).toBeVisible();
    await expect(hero.locator('h1')).toHaveCount(1);
  });

  test('all section headings are visible', async ({ page }) => {
    const h2s = page.locator('h2');
    const count = await h2s.count();
    expect(count).toBeGreaterThanOrEqual(2); // at least 2 content sections expected
    for (let i = 0; i < count; i++) {
      await expect(h2s.nth(i)).toBeVisible();
    }
  });

  test('CTA buttons meet minimum touch target size', async ({ page, viewport }) => {
    if (!viewport || viewport.width >= 1024) return; // only check mobile/tablet
    const buttons = page.locator('a[role="button"], button').filter({ hasText: /.+/ });
    const count = await buttons.count();
    for (let i = 0; i < count; i++) {
      const box = await buttons.nth(i).boundingBox();
      if (box) {
        expect(box.height, `Button ${i} height`).toBeGreaterThanOrEqual(44);
      }
    }
  });

  test('images are not stretched or broken', async ({ page }) => {
    const images = page.locator('img[src]');
    const count = await images.count();
    for (let i = 0; i < count; i++) {
      const img = images.nth(i);
      await expect(img).toBeVisible();
      const naturalWidth = await img.evaluate((el: HTMLImageElement) => el.naturalWidth);
      expect(naturalWidth, `Image ${i} has no natural width (broken)`).toBeGreaterThan(0);
    }
  });

  test('text is readable (minimum 12px rendered)', async ({ page }) => {
    const smallText = await page.evaluate(() => {
      const allText = document.querySelectorAll('p, span, a, li, td, th, label');
      const tooSmall: string[] = [];
      allText.forEach((el) => {
        const style = window.getComputedStyle(el);
        const fontSize = parseFloat(style.fontSize);
        if (fontSize < 12 && el.textContent?.trim()) {
          tooSmall.push(`${el.tagName}: "${el.textContent?.slice(0, 30)}" = ${fontSize}px`);
        }
      });
      return tooSmall;
    });
    expect(smallText).toEqual([]);
  });

  test('footer legal links are present', async ({ page }) => {
    const footer = page.locator('footer');
    await expect(footer.getByRole('link', { name: /privacy/i })).toBeVisible();
    await expect(footer.getByRole('link', { name: /terms/i })).toBeVisible();
  });
});

test.describe('Mobile navigation', () => {
  test('hamburger menu opens and closes', async ({ page, viewport }) => {
    if (!viewport || viewport.width >= 768) {
      test.skip();
      return;
    }
    await page.goto('/');

    const burger = page.getByRole('button', { name: /menu/i });
    await expect(burger).toBeVisible();

    // Open drawer
    await burger.click();
    const drawer = page.locator('[role="dialog"]');
    await expect(drawer).toBeVisible();

    // Close on nav link click
    const firstLink = drawer.getByRole('link').first();
    await firstLink.click();
    await expect(drawer).not.toBeVisible();
  });
});
```

## Playwright Visual Snapshot Tests

Capture and compare screenshots per viewport to catch visual regressions:

```ts
// e2e/visual.spec.ts
import { test, expect } from '@playwright/test';

// Adapt this list to match the sections chosen for this landing page.
// Each section component should have a data-section attribute.
const sections = [
  { name: 'hero', selector: 'section:first-of-type' },
  // Add one entry per catalog section used, e.g.:
  // { name: 'features', selector: '[data-section="features"]' },
  // { name: 'pricing', selector: '[data-section="pricing"]' },
  { name: 'footer', selector: 'footer' },
];

for (const { name, selector } of sections) {
  test(`${name} section visual snapshot`, async ({ page }) => {
    await page.goto('/');
    const section = page.locator(selector);
    await section.scrollIntoViewIfNeeded();
    // Wait for animations to complete
    await page.waitForTimeout(1000);
    await expect(section).toHaveScreenshot(`${name}.png`, {
      maxDiffPixelRatio: 0.01,
    });
  });
}
```

Rules:
- Add `data-section="<name>"` attributes to each section component for reliable test selectors
- Snapshot baseline images are committed to the repo in `e2e/visual.spec.ts-snapshots/`
- Update snapshots with `pnpm playwright test --update-snapshots` when intentional design changes are made
- Set `maxDiffPixelRatio: 0.01` (1%) to allow minor anti-aliasing differences

## Playwright Accessibility Audit

```ts
// e2e/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility audit', () => {
  test('full page has no a11y violations', async ({ page }) => {
    await page.goto('/');
    // Wait for animations to settle
    await page.waitForTimeout(1500);

    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('color contrast passes', async ({ page }) => {
    await page.goto('/');
    const results = await new AxeBuilder({ page })
      .withRules(['color-contrast'])
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('heading hierarchy is correct', async ({ page }) => {
    await page.goto('/');
    const headings = await page.evaluate(() => {
      const hs = document.querySelectorAll('h1, h2, h3, h4, h5, h6');
      return Array.from(hs).map((h) => ({
        level: parseInt(h.tagName[1]),
        text: h.textContent?.trim().slice(0, 50),
      }));
    });

    // Exactly one h1
    const h1s = headings.filter((h) => h.level === 1);
    expect(h1s).toHaveLength(1);

    // No skipped levels
    for (let i = 1; i < headings.length; i++) {
      const jump = headings[i].level - headings[i - 1].level;
      expect(jump, `Heading jump from "${headings[i - 1].text}" to "${headings[i].text}"`).toBeLessThanOrEqual(1);
    }
  });
});
```

## Lighthouse Performance Audit (Local, No CI)

Lighthouse runs directly via Chrome's built-in tooling or `pnpm dlx` — no CI pipeline, no config files, fully self-contained.

### Running Lighthouse Locally

```bash
# 1. Build and start the production server
pnpm build && pnpm start

# 2. In another terminal — run Lighthouse via Chrome's CLI
# Desktop audit:
pnpm dlx lighthouse http://localhost:3000 --output=html --output-path=./lighthouse-desktop.html --preset=desktop

# Mobile audit (default):
pnpm dlx lighthouse http://localhost:3000 --output=html --output-path=./lighthouse-mobile.html

# JSON output for programmatic checks:
pnpm dlx lighthouse http://localhost:3000 --output=json --output-path=./lighthouse-results.json
```

That's it. No config files, no server setup. Opens a Chrome instance, runs the audit, outputs a full HTML report you can open in your browser.

### What Lighthouse Measures

| Category | Target Score | Key Metrics |
|----------|-------------|-------------|
| Performance | >= 90 | LCP, TBT, CLS, FCP, Speed Index |
| Accessibility | >= 95 | ARIA, color contrast, labels, landmarks |
| Best Practices | >= 90 | HTTPS, no console errors, image aspect ratios |
| SEO | >= 95 | meta tags, crawlability, mobile-friendly |

### Reading the Report

The HTML report (`lighthouse-desktop.html`) shows:
- Scores per category with pass/fail coloring
- **Opportunities**: specific recommendations with estimated time savings (e.g., "Serve images in next-gen formats — potential savings 1.2s")
- **Diagnostics**: detailed metrics (DOM size, main thread work, resource transfer sizes)
- **Passed audits**: everything that's already good

### Quick Headless Check (No HTML Report)

For a fast pass/fail check without generating reports:
```bash
pnpm dlx lighthouse http://localhost:3000 --only-categories=performance,accessibility,seo --output=json --quiet | node -e "
  const r = JSON.parse(require('fs').readFileSync('/dev/stdin','utf8'));
  const cats = r.categories;
  console.log('Performance:', Math.round(cats.performance.score*100));
  console.log('Accessibility:', Math.round(cats.accessibility.score*100));
  console.log('SEO:', Math.round(cats.seo.score*100));
  const pass = cats.performance.score >= 0.9 && cats.accessibility.score >= 0.95 && cats.seo.score >= 0.95;
  console.log(pass ? 'PASS' : 'FAIL');
  process.exit(pass ? 0 : 1);
"
```

### Add to `.gitignore`

```
lighthouse-*.html
lighthouse-*.json
```

## Playwright Performance Measurement

For more granular CWV measurement during real user flows:

```ts
// e2e/performance.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Core Web Vitals', () => {
  test('LCP is under 2.5s', async ({ page }) => {
    await page.goto('/', { waitUntil: 'networkidle' });

    const lcp = await page.evaluate(() => {
      return new Promise<number>((resolve) => {
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          const lastEntry = entries[entries.length - 1];
          resolve(lastEntry.startTime);
        }).observe({ type: 'largest-contentful-paint', buffered: true });

        // Fallback timeout
        setTimeout(() => resolve(0), 10000);
      });
    });

    expect(lcp).toBeGreaterThan(0);
    expect(lcp).toBeLessThan(2500);
  });

  test('CLS is under 0.1', async ({ page }) => {
    await page.goto('/');
    // Scroll through the full page to trigger any layout shifts
    await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
    await page.waitForTimeout(2000);

    const cls = await page.evaluate(() => {
      return new Promise<number>((resolve) => {
        let clsValue = 0;
        new PerformanceObserver((list) => {
          for (const entry of list.getEntries() as any[]) {
            if (!entry.hadRecentInput) {
              clsValue += entry.value;
            }
          }
        }).observe({ type: 'layout-shift', buffered: true });

        setTimeout(() => resolve(clsValue), 3000);
      });
    });

    expect(cls).toBeLessThan(0.1);
  });

  test('total page weight is under budget', async ({ page }) => {
    const [response] = await Promise.all([
      page.goto('/', { waitUntil: 'networkidle' }),
    ]);

    const resourceSizes = await page.evaluate(() => {
      const entries = performance.getEntriesByType('resource') as PerformanceResourceTiming[];
      let totalJS = 0;
      let totalCSS = 0;
      let totalImages = 0;
      let totalFonts = 0;

      entries.forEach((entry) => {
        const size = entry.transferSize || 0;
        if (entry.name.match(/\.js($|\?)/)) totalJS += size;
        else if (entry.name.match(/\.css($|\?)/)) totalCSS += size;
        else if (entry.name.match(/\.(png|jpg|jpeg|webp|avif|svg|gif)($|\?)/)) totalImages += size;
        else if (entry.name.match(/\.(woff2?|ttf|otf)($|\?)/)) totalFonts += size;
      });

      return { totalJS, totalCSS, totalImages, totalFonts };
    });

    // Budget thresholds (in bytes)
    expect(resourceSizes.totalJS, 'JS budget').toBeLessThan(250 * 1024);       // 250KB
    expect(resourceSizes.totalCSS, 'CSS budget').toBeLessThan(80 * 1024);      // 80KB
    expect(resourceSizes.totalFonts, 'Font budget').toBeLessThan(150 * 1024);  // 150KB
  });
});
```

## Package.json Scripts

```json
{
  "scripts": {
    "fmt": "deno fmt --line-width 120",
    "fmt:watch": "deno fmt --line-width 120 --watch",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:visual": "playwright test e2e/visual.spec.ts",
    "test:visual:update": "playwright test e2e/visual.spec.ts --update-snapshots",
    "test:a11y": "playwright test e2e/accessibility.spec.ts",
    "test:perf": "playwright test e2e/performance.spec.ts",
    "test:lighthouse": "pnpm dlx lighthouse http://localhost:3000 --output=html --output-path=./lighthouse-report.html --preset=desktop",
    "test:all": "vitest run && playwright test"
  }
}
```

## Test Running Rules

- `pnpm test` — run before every commit. Must pass.
- `pnpm test:e2e` — run after significant layout or styling changes. Runs across all viewport projects defined in Playwright config.
- `pnpm test:visual:update` — run after intentional design changes to update snapshot baselines.
- `pnpm test:lighthouse` — run manually before deployment. Requires `pnpm build && pnpm start` running in another terminal.
- `pnpm test:all` — unit + E2E suite. Run before any release. Lighthouse is run separately since it requires a running production server.

## Performance Budget Summary

| Resource | Budget |
|----------|--------|
| Total JS (transferred) | < 250KB |
| Total CSS (transferred) | < 80KB |
| Total Fonts (transferred) | < 150KB |
| LCP | < 2.5s |
| CLS | < 0.1 |
| TBT (Total Blocking Time) | < 200ms |
| Lighthouse Performance Score | >= 90 |
| Lighthouse Accessibility Score | >= 95 |
| Lighthouse SEO Score | >= 95 |
