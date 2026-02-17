# Animation & Visual Effects Reference

## Animation Guidelines

Animation library: **Framer Motion**. All scroll-triggered entrances, staggered reveals, and interactive transitions use Framer Motion. Every animated component must be in a `'use client'` file.

### Setup

Add a shared animation utilities file at `lib/motion.ts`:

```ts
import { Variants } from 'framer-motion';

// Shared fade-up entrance used by most sections
export const fadeUp: Variants = {
  hidden: { opacity: 0, y: 24 },
  visible: { opacity: 1, y: 0 },
};

// Fade in without vertical movement
export const fadeIn: Variants = {
  hidden: { opacity: 0 },
  visible: { opacity: 1 },
};

// Scale up from slightly smaller
export const scaleUp: Variants = {
  hidden: { opacity: 0, scale: 0.95 },
  visible: { opacity: 1, scale: 1 },
};

// Slide in from left
export const slideInLeft: Variants = {
  hidden: { opacity: 0, x: -32 },
  visible: { opacity: 1, x: 0 },
};

// Slide in from right
export const slideInRight: Variants = {
  hidden: { opacity: 0, x: 32 },
  visible: { opacity: 1, x: 0 },
};

// Stagger container — wrap children that each have their own variants
export const staggerContainer: Variants = {
  hidden: {},
  visible: {
    transition: {
      staggerChildren: 0.1,
      delayChildren: 0.1,
    },
  },
};

// Default transition config
export const defaultTransition = {
  duration: 0.5,
  ease: [0.25, 0.1, 0.25, 1], // cubic-bezier easeOut
};

// Spring transition for interactive elements
export const springTransition = {
  type: 'spring' as const,
  stiffness: 300,
  damping: 25,
};
```

### Scroll-Triggered Entrances (`whileInView`)

Use `motion.div` with `whileInView` for all section entrances. This replaces manual `useIntersection` logic.

```tsx
'use client';
import { motion } from 'framer-motion';
import { fadeUp, defaultTransition } from '@/lib/motion';

export function FeatureCard({ title, description, icon }: Props) {
  return (
    <motion.div
      variants={fadeUp}
      initial="hidden"
      whileInView="visible"
      viewport={{ once: true, margin: '-80px' }}
      transition={defaultTransition}
    >
      {/* card content */}
    </motion.div>
  );
}
```

Rules:
- **Always set `viewport={{ once: true }}`** — elements animate in once and stay visible. Re-triggering on scroll-out feels janky on landing pages.
- Use `margin: '-80px'` (or similar negative value) on viewport so animation triggers slightly before the element enters the viewport — feels more natural.
- Never use `whileInView` on elements above the fold (hero) — they should animate on mount with `initial` + `animate` instead.

### Staggered Group Animations

For grids of cards (features, pricing, testimonials), use a parent `staggerContainer` variant with child `fadeUp` variants:

```tsx
'use client';
import { motion } from 'framer-motion';
import { staggerContainer, fadeUp, defaultTransition } from '@/lib/motion';

export function FeaturesGrid({ features }: Props) {
  return (
    <motion.div
      variants={staggerContainer}
      initial="hidden"
      whileInView="visible"
      viewport={{ once: true, margin: '-80px' }}
    >
      <SimpleGrid cols={{ base: 1, sm: 2, lg: 3 }}>
        {features.map((feature) => (
          <motion.div key={feature.id} variants={fadeUp} transition={defaultTransition}>
            <FeatureCard {...feature} />
          </motion.div>
        ))}
      </SimpleGrid>
    </motion.div>
  );
}
```

Rules:
- `staggerChildren: 0.1` — 100ms between each child appearing. Range: 0.08–0.15 for landing pages. Slower than 0.15 feels sluggish.
- Only stagger groups of 3-8 items. For larger lists, animate the entire grid as one unit.
- Children inherit `initial` and `whileInView` from the parent when they have `variants` — do not set these on children.

### Hero Animation (Mount Animation)

The hero animates on page load, not on scroll. Use `initial` + `animate` (not `whileInView`):

```tsx
'use client';
import { motion } from 'framer-motion';

export function Hero() {
  return (
    <section>
      <motion.div
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6, ease: [0.25, 0.1, 0.25, 1] }}
      >
        <Title>Headline</Title>
      </motion.div>

      <motion.div
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6, delay: 0.15, ease: [0.25, 0.1, 0.25, 1] }}
      >
        <Text>Subheadline</Text>
      </motion.div>

      <motion.div
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6, delay: 0.3, ease: [0.25, 0.1, 0.25, 1] }}
      >
        <Button>CTA</Button>
      </motion.div>
    </section>
  );
}
```

Rules:
- Stagger hero elements manually with increasing `delay`: 0 → 0.15 → 0.3 → 0.45
- Total hero animation sequence should complete within **1 second** — users should not wait for content
- Hero image can animate separately (fade-in or scale-up from 0.95) with a slightly longer duration

### Per-Archetype Animation Map

| Archetype | Animation | Trigger |
|-----------|-----------|---------|
| **Header** | No entrance animation. Sticky header is always present. | — |
| **Hero** | Sequential fade-up: badge → headline → subheadline → CTAs → image | Page mount (`animate`) |
| **`logo-bar`** | Fade-in as a single unit | `whileInView`, `once: true` |
| **`stats-bar`** | Fade-in, optional animated number counters | `whileInView`, `once: true` |
| **`grid-cards`** | Staggered fade-up per card (0.1s stagger) | `whileInView`, `once: true` |
| **`split-content`** | Slide-in from left (text) + slide-in from right (image) or fade-up | `whileInView`, `once: true` |
| **`testimonials`** | Staggered fade-up per card or slide-in from alternating sides | `whileInView`, `once: true` |
| **`pricing`** | Staggered scale-up per card (recommended plan animates last with a slight pop) | `whileInView`, `once: true` |
| **`accordion-list`** | Fade-up for the section heading. Accordion open/close uses Mantine's built-in transition. | `whileInView`, `once: true` |
| **`cta-block`** | Fade-up for headline + CTA button | `whileInView`, `once: true` |
| **`steps`** | Staggered fade-up per step (0.12s stagger) | `whileInView`, `once: true` |
| **`form-capture`** | Fade-up for the form container as a single unit | `whileInView`, `once: true` |
| **`video-embed`** | Scale-up from 0.95 + fade-in | `whileInView`, `once: true` |
| **Footer** | Fade-in as a single unit (subtle, fast) | `whileInView`, `once: true` |

### Interactive Micro-Animations

For hover and tap effects on interactive elements, use Framer Motion's `whileHover` and `whileTap`:

```tsx
<motion.div
  whileHover={{ y: -4, boxShadow: '0 12px 24px rgba(0,0,0,0.1)' }}
  whileTap={{ scale: 0.98 }}
  transition={springTransition}
>
  <Card>...</Card>
</motion.div>
```

Rules:
- Use `whileHover` only for cards and larger interactive surfaces — not for text links or small buttons (Mantine handles those)
- Hover lift: `y: -2` to `y: -6` range. Anything more feels exaggerated.
- Tap scale: `scale: 0.97` to `scale: 0.99`. Subtle press-in feedback.
- Always use `springTransition` for interactive animations — springs feel more natural than eased curves for user-initiated motion
- Hover effects are desktop-only — on touch devices they have no effect (Framer Motion handles this)

### Reduced Motion

Respect the user's `prefers-reduced-motion` system setting via the `MotionProvider` in `components/ui/MotionProvider.tsx`:

```tsx
// components/ui/MotionProvider.tsx
'use client';
import { LazyMotion, domAnimation, MotionConfig } from 'framer-motion';

export function MotionProvider({ children }: { children: React.ReactNode }) {
  return (
    <LazyMotion features={domAnimation}>
      <MotionConfig reducedMotion="user">
        {children}
      </MotionConfig>
    </LazyMotion>
  );
}
```

For components that need to conditionally adjust animation logic (not just disable), use the `useReducedMotion` hook directly in the component:

```tsx
// Inside any 'use client' component
import { useReducedMotion } from 'framer-motion';

const shouldReduceMotion = useReducedMotion();
// Pass to transition: { duration: shouldReduceMotion ? 0 : 0.5 }
```

**Important:** Do not add `'use client'` to `lib/motion.ts` — it contains only constants (variants, transitions) that must be importable by both server and client components.

Rules:
- **Always use `<MotionConfig reducedMotion="user">`** in the root layout — this globally respects the OS-level reduced motion preference and disables all Framer Motion animations for users who have it enabled
- Wrap the app with `<LazyMotion features={domAnimation}>` to reduce initial bundle size (loads animation features lazily)
- Place `MotionProvider` inside `MantineProvider` in the root layout
- Mantine's `respectReducedMotion: true` in the theme handles Mantine's own transitions (Accordion, Drawer, etc.)

### Performance Rules

- **Never animate `width`, `height`, `top`, `left`, or `margin`** — these trigger layout recalculation. Only animate `transform` (x, y, scale, rotate) and `opacity`.
- Framer Motion uses `transform` under the hood for `x`, `y`, `scale`, `rotate` — these are GPU-composited and performant.
- Use `will-change: transform` sparingly and only on elements that are actively animating — do not blanket-apply it.
- Do not animate elements that are not visible or are far off-screen.
- Keep `staggerChildren` groups to 8 items or fewer — staggering 20+ cards causes noticeable frame drops.
- Entrance animations should complete within **0.5s per element**. The full viewport's animation sequence should complete within **1.2s**.
- Test animation smoothness on a throttled CPU (Chrome DevTools → Performance → 4x slowdown).

### What NOT to Animate

- Text content that users need to read immediately (body paragraphs in main content)
- Navigation links in the header
- Footer content (keep it minimal — a fast fade-in at most)
- Form inputs and labels
- Legal text / cookie banners
- Anything that causes layout shift (CLS) when it animates in

## Visual Effects Catalog

Advanced visual patterns that elevate a landing page beyond conventional layouts. These are **optional enhancements** — apply them intentionally based on the design goal, not by default. Every effect here must degrade gracefully on low-powered devices and respect `prefers-reduced-motion`.

All effects use the existing stack: CSS, Framer Motion, and Mantine. No additional animation libraries unless explicitly noted.

### Scroll-Linked Animations

These are animations driven by scroll position (continuous), not scroll intersection (binary trigger). They use Framer Motion's `useScroll` + `useTransform`.

#### `parallax` — Parallax Layers

Background elements move at a different speed than foreground content, creating depth.

```tsx
'use client';
import { motion, useScroll, useTransform } from 'framer-motion';
import { useRef } from 'react';

export function ParallaxSection({ children }: { children: React.ReactNode }) {
  const ref = useRef(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start end', 'end start'], // track from section entering to leaving viewport
  });

  // Background moves at 50% of scroll speed (parallax factor)
  const y = useTransform(scrollYProgress, [0, 1], ['-20%', '20%']);

  return (
    <section ref={ref} style={{ position: 'relative', overflow: 'hidden' }}>
      <motion.div
        style={{ y, position: 'absolute', inset: 0, zIndex: -1 }}
      >
        {/* background image or decorative element */}
      </motion.div>
      {children}
    </section>
  );
}
```

**When to use:** Hero backgrounds, decorative section backgrounds, floating decorative elements beside content.

**Performance:** Only transform `y` (GPU-composited). Use `overflow: hidden` on the container to prevent layout overflow. Disable on `prefers-reduced-motion`.

#### `scroll-progress` — Progress-Driven Transforms

Element properties change continuously as the user scrolls through a section — opacity, scale, rotation, position.

```tsx
'use client';
import { motion, useScroll, useTransform } from 'framer-motion';
import { useRef } from 'react';

export function ScrollRevealImage() {
  const ref = useRef(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start 80%', 'end 40%'], // animation range within viewport
  });

  const scale = useTransform(scrollYProgress, [0, 1], [0.8, 1]);
  const opacity = useTransform(scrollYProgress, [0, 0.5], [0, 1]);
  const rotate = useTransform(scrollYProgress, [0, 1], [-5, 0]);

  return (
    <motion.div ref={ref} style={{ scale, opacity, rotate }}>
      {/* content */}
    </motion.div>
  );
}
```

**When to use:** Product screenshots that grow into view, feature images that rotate into place, elements that fade and scale as user scrolls.

**Tip:** `offset` controls when the animation starts and ends relative to the viewport. `['start 80%', 'end 40%']` means: start when element top hits 80% of viewport, end when element bottom hits 40%.

#### `sticky-scroll` — Pinned Section with Content Swaps

Section stays pinned while content inside transitions — text, images, or entire layouts change as the user scrolls. Common on high-end SaaS landing pages.

```tsx
'use client';
import { motion, useScroll, useTransform } from 'framer-motion';
import { useRef } from 'react';

const steps = [
  { title: 'Step 1', description: '...', image: '/step1.png' },
  { title: 'Step 2', description: '...', image: '/step2.png' },
  { title: 'Step 3', description: '...', image: '/step3.png' },
];

export function StickyScrollSection() {
  const containerRef = useRef(null);
  const { scrollYProgress } = useScroll({
    target: containerRef,
    offset: ['start start', 'end end'],
  });

  // Map scroll progress to active step index
  const activeIndex = useTransform(scrollYProgress, [0, 1], [0, steps.length - 1]);

  return (
    // Container height = number of steps × viewport height
    <div ref={containerRef} style={{ height: `${steps.length * 100}vh`, position: 'relative' }}>
      <div style={{ position: 'sticky', top: 0, height: '100vh', display: 'flex', alignItems: 'center' }}>
        <Grid>
          <Grid.Col span={{ base: 12, md: 6 }}>
            {/* Text side — crossfade between steps */}
            {steps.map((step, i) => (
              <StepContent key={i} index={i} activeIndex={activeIndex} step={step} />
            ))}
          </Grid.Col>
          <Grid.Col span={{ base: 12, md: 6 }}>
            {/* Image side — crossfade between images */}
            {steps.map((step, i) => (
              <StepImage key={i} index={i} activeIndex={activeIndex} src={step.image} />
            ))}
          </Grid.Col>
        </Grid>
      </div>
    </div>
  );
}
```

**When to use:** "How it works" sections, feature walkthroughs, storytelling sequences. High impact but takes significant vertical space.

**Rules:**
- Container height = `(number of steps) × 100vh` — gives each step a full viewport of scroll travel
- Inner content uses `position: sticky; top: 0; height: 100vh`
- Individual step content crossfades using `opacity` driven by `scrollYProgress`
- Mobile: fall back to a vertical stack (no sticky) — sticky scroll on small screens is disorienting
- Minimum 3 steps, maximum 5 — more feels endless

#### `horizontal-scroll` — Horizontal Scroll Section

Content scrolls horizontally while the user scrolls vertically. Used for project showcases, portfolio items, or feature cards.

```tsx
'use client';
import { motion, useScroll, useTransform } from 'framer-motion';
import { useRef } from 'react';

export function HorizontalScrollSection({ items }: { items: React.ReactNode[] }) {
  const containerRef = useRef(null);
  const { scrollYProgress } = useScroll({
    target: containerRef,
    offset: ['start start', 'end end'],
  });

  // Move container from 0% to -(100% - one viewport width)
  const x = useTransform(scrollYProgress, [0, 1], ['0%', `-${(items.length - 1) * 100}%`]);

  return (
    <div ref={containerRef} style={{ height: `${items.length * 100}vh` }}>
      <div style={{ position: 'sticky', top: 0, height: '100vh', overflow: 'hidden' }}>
        <motion.div
          style={{ x, display: 'flex', width: `${items.length * 100}vw` }}
        >
          {items.map((item, i) => (
            <div key={i} style={{ width: '100vw', flexShrink: 0, padding: '0 2rem' }}>
              {item}
            </div>
          ))}
        </motion.div>
      </div>
    </div>
  );
}
```

**When to use:** Portfolio/case study showcases, feature highlights with large visuals, timeline/history sections.

**Rules:**
- Mobile: fall back to a vertical stack or swipeable carousel — horizontal scroll via vertical scrolling is confusing on small screens
- 3-6 items max
- Each item takes approximately one viewport width

### Background & Section Effects

Visual treatments for section backgrounds and transitions between sections.

#### `mesh-gradient` — Animated Mesh Gradient Background

A soft, animated gradient background with multiple color stops that slowly shift.

```css
/* CSS Module approach — zero JS, GPU-composited */
.meshGradient {
  background: linear-gradient(135deg, var(--color-1), var(--color-2), var(--color-3));
  background-size: 400% 400%;
  animation: gradientShift 15s ease infinite;
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

@media (prefers-reduced-motion: reduce) {
  .meshGradient { animation: none; }
}
```

For more complex mesh gradients with multiple blurred blobs:

```css
.meshGradientAdvanced {
  position: relative;
  overflow: hidden;
}

.meshGradientAdvanced::before,
.meshGradientAdvanced::after {
  content: '';
  position: absolute;
  width: 60%;
  aspect-ratio: 1;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.5;
  animation: float 20s ease-in-out infinite;
}

.meshGradientAdvanced::before {
  background: var(--mantine-color-brand-3);
  top: -20%;
  left: -10%;
}

.meshGradientAdvanced::after {
  background: var(--mantine-color-brand-6);
  bottom: -20%;
  right: -10%;
  animation-delay: -10s;
}

@keyframes float {
  0%, 100% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(30px, -30px) scale(1.05); }
  66% { transform: translate(-20px, 20px) scale(0.95); }
}

@media (prefers-reduced-motion: reduce) {
  .meshGradientAdvanced::before,
  .meshGradientAdvanced::after { animation: none; }
}
```

**When to use:** Hero backgrounds, CTA sections, full-page backgrounds. Creates a premium, modern feel.

**Performance:** Pure CSS, GPU-composited (`transform` + `opacity`). Use `filter: blur()` sparingly — large blur radii on large elements can cause frame drops on mobile. Test on real devices.

#### `glow` — Glow & Neon Effects

Colored glow around cards, buttons, or text to draw attention.

```css
.glowCard {
  box-shadow:
    0 0 20px rgba(var(--mantine-color-brand-5-rgb), 0.15),
    0 0 60px rgba(var(--mantine-color-brand-5-rgb), 0.1);
  border: 1px solid rgba(var(--mantine-color-brand-5-rgb), 0.2);
}

/* Animated glow pulse */
.glowPulse {
  animation: pulse 3s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { box-shadow: 0 0 20px rgba(var(--mantine-color-brand-5-rgb), 0.15); }
  50% { box-shadow: 0 0 40px rgba(var(--mantine-color-brand-5-rgb), 0.3); }
}

/* Glow text */
.glowText {
  text-shadow: 0 0 20px rgba(var(--mantine-color-brand-5-rgb), 0.5);
}

@media (prefers-reduced-motion: reduce) {
  .glowPulse { animation: none; }
}
```

**When to use:** Dark-themed landing pages, highlighting a primary CTA or featured card, hero headlines. Works best on dark backgrounds.

**Rules:** Glow effects are barely visible on light backgrounds — pair with dark mode or dark section backgrounds. Use theme colors for consistency.

#### `glassmorphism` — Frosted Glass

Translucent background with blur, creating a frosted glass effect over content or gradients.

```css
.glass {
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: var(--mantine-radius-lg);
}

@mixin light {
  .glass {
    background: rgba(255, 255, 255, 0.6);
    border: 1px solid rgba(255, 255, 255, 0.3);
  }
}
```

**When to use:** Cards floating over gradient backgrounds, sticky header over hero, modal/overlay styling, pricing cards over a colorful background.

**Performance:** `backdrop-filter: blur()` is GPU-composited on modern browsers but can be expensive on mobile Safari with many stacked glass layers. Limit to 2-3 glass elements visible simultaneously.

#### `noise-texture` — Subtle Grain Overlay

A subtle noise texture overlay that adds warmth and prevents banding in gradients.

```css
.noiseOverlay {
  position: relative;
}

.noiseOverlay::after {
  content: '';
  position: absolute;
  inset: 0;
  background-image: url('/images/noise.svg'); /* tiny SVG noise pattern */
  opacity: 0.03;
  pointer-events: none;
  mix-blend-mode: overlay;
}
```

Alternatively, generate noise with CSS (no image required):

```css
.noiseOverlay::after {
  content: '';
  position: absolute;
  inset: 0;
  background: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
  pointer-events: none;
}
```

**When to use:** Over gradient backgrounds (hero, CTA sections) to prevent color banding. Over solid-color sections for a premium textured feel.

#### `dot-grid` — Dot Pattern Background

A repeating dot grid pattern as a subtle background texture.

```css
.dotGrid {
  background-image: radial-gradient(circle, var(--mantine-color-dimmed) 1px, transparent 1px);
  background-size: 24px 24px;
}
```

**When to use:** Light section backgrounds, behind feature grids, full-page subtle texture. Works well in both light and dark modes.

#### `section-divider` — Shaped Section Transitions

Non-flat transitions between sections — waves, curves, diagonal cuts, or slants.

```css
/* Wave divider — place on the section that comes AFTER */
.waveDividerTop {
  position: relative;
  margin-top: -1px; /* prevent subpixel gap */
}

.waveDividerTop::before {
  content: '';
  position: absolute;
  top: -60px;
  left: 0;
  width: 100%;
  height: 60px;
  background: inherit;
  clip-path: ellipse(55% 100% at 50% 100%);
}

/* Diagonal cut */
.diagonalTop {
  clip-path: polygon(0 8%, 100% 0, 100% 100%, 0 100%);
  margin-top: -4vw; /* overlap previous section */
  padding-top: calc(4vw + 3rem); /* compensate for the clip */
}

/* Curved separator using SVG */
.curvedTop::before {
  content: '';
  position: absolute;
  top: -1px;
  left: 0;
  width: 100%;
  height: 80px;
  background: url("data:image/svg+xml,%3Csvg viewBox='0 0 1440 80' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath fill='%23ffffff' d='M0,40 C360,80 1080,0 1440,40 L1440,0 L0,0 Z'/%3E%3C/svg%3E");
  background-size: cover;
}
```

**When to use:** Between sections with different background colors. Adds visual interest without JavaScript. Use sparingly — 1-2 dividers per page max.

**Rules:**
- Use `clip-path` or SVG — they are GPU-composited and performant
- Match the divider fill color to the previous section's background
- On mobile, reduce divider height (use `clamp()` or responsive values)
- Dividers work best between high-contrast sections (e.g., white → brand-colored)

#### `aurora` — Aurora / Northern Lights Effect

Soft, moving color blobs that mimic aurora borealis. Premium hero background effect.

```css
.aurora {
  position: relative;
  overflow: hidden;
  background: var(--mantine-color-dark-9);
}

.aurora::before {
  content: '';
  position: absolute;
  width: 200%;
  height: 200%;
  top: -50%;
  left: -50%;
  background:
    conic-gradient(from 180deg at 50% 50%,
      var(--mantine-color-blue-4) 0deg,
      var(--mantine-color-violet-4) 60deg,
      var(--mantine-color-cyan-4) 120deg,
      transparent 180deg,
      transparent 360deg
    );
  filter: blur(100px);
  opacity: 0.3;
  animation: auroraRotate 30s linear infinite;
}

@keyframes auroraRotate {
  to { transform: rotate(360deg); }
}

@media (prefers-reduced-motion: reduce) {
  .aurora::before { animation: none; }
}
```

**When to use:** Dark-themed hero sections only. Very high visual impact. Use as the background behind hero text with glassmorphism cards.

### Text & Micro-Interaction Effects

#### `text-reveal` — Character-by-Character Reveal

Text animates in one character or word at a time.

```tsx
'use client';
import { motion } from 'framer-motion';

interface TextRevealProps {
  text: string;
  /** 'word' splits by space, 'char' splits by character */
  splitBy?: 'word' | 'char';
}

export function TextReveal({ text, splitBy = 'word' }: TextRevealProps) {
  const units = splitBy === 'word' ? text.split(' ') : text.split('');

  return (
    <motion.span
      initial="hidden"
      whileInView="visible"
      viewport={{ once: true }}
      variants={{
        visible: { transition: { staggerChildren: splitBy === 'char' ? 0.03 : 0.08 } },
      }}
      aria-label={text}
    >
      {units.map((unit, i) => (
        <motion.span
          key={i}
          variants={{
            hidden: { opacity: 0, y: 20 },
            visible: { opacity: 1, y: 0 },
          }}
          transition={{ duration: 0.4, ease: [0.25, 0.1, 0.25, 1] }}
          style={{ display: 'inline-block', whiteSpace: 'pre' }}
        >
          {unit}{splitBy === 'word' && i < units.length - 1 ? '\u00A0' : ''}
        </motion.span>
      ))}
    </motion.span>
  );
}
```

**When to use:** Hero headlines, section headings that need dramatic entrance. Use `word` split for readability, `char` split for short dramatic text only.

**Rules:**
- Add `aria-label={text}` on the container so screen readers read the full text, not individual characters
- `word` split: stagger 0.06–0.1s. `char` split: stagger 0.02–0.04s
- Only use on 1-2 headings per page — overuse dilutes impact
- Total reveal should complete within 1.5s

#### `gradient-text` — Animated Gradient Text

Text with an animated gradient fill.

```css
.gradientText {
  background: linear-gradient(
    135deg,
    var(--mantine-color-brand-4),
    var(--mantine-color-violet-4),
    var(--mantine-color-brand-4)
  );
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: shimmer 4s linear infinite;
}

@keyframes shimmer {
  to { background-position: 200% center; }
}

@media (prefers-reduced-motion: reduce) {
  .gradientText {
    animation: none;
    background-size: 100% auto;
  }
}
```

**When to use:** Hero headlines, badge text, section accents. Static gradient text (without animation) is also effective.

**Alternative:** Mantine's built-in `variant="gradient"` on `Text` and `Title` provides static gradient text without custom CSS:
```tsx
<Title variant="gradient" gradient={{ from: 'brand', to: 'violet', deg: 135 }}>
```

#### `typewriter` — Typewriter Effect

Text appears as if typed, optionally with a blinking cursor.

```tsx
'use client';
import { motion, useMotionValue, useTransform, animate } from 'framer-motion';
import { useEffect } from 'react';

export function Typewriter({ text, duration = 2 }: { text: string; duration?: number }) {
  const count = useMotionValue(0);
  const displayText = useTransform(count, (v) => text.slice(0, Math.round(v)));

  useEffect(() => {
    const controls = animate(count, text.length, {
      duration,
      ease: 'linear',
    });
    return controls.stop;
  }, [count, text.length, duration]);

  return (
    <span aria-label={text}>
      <motion.span>{displayText}</motion.span>
      <span className="cursor" aria-hidden="true">|</span>
    </span>
  );
}
```

```css
.cursor {
  animation: blink 0.8s step-end infinite;
}

@keyframes blink {
  50% { opacity: 0; }
}
```

**When to use:** Hero subheadlines, taglines that cycle through different value props, command-line styled text.

#### `counter` — Animated Number Counter

Numbers count up from 0 to their target value when scrolled into view.

```tsx
'use client';
import { motion, useMotionValue, useTransform, animate, useInView } from 'framer-motion';
import { useEffect, useRef } from 'react';

export function AnimatedCounter({ target, suffix = '' }: { target: number; suffix?: string }) {
  const ref = useRef(null);
  const isInView = useInView(ref, { once: true });
  const count = useMotionValue(0);
  const rounded = useTransform(count, (v) => `${Math.round(v).toLocaleString()}${suffix}`);

  useEffect(() => {
    if (isInView) {
      animate(count, target, { duration: 2, ease: 'easeOut' });
    }
  }, [isInView, count, target]);

  return <motion.span ref={ref}>{rounded}</motion.span>;
}

// Usage: <AnimatedCounter target={10000} suffix="+" />
```

**When to use:** Stats bar sections, social proof numbers, impact metrics.

#### `magnetic-button` — Magnetic Cursor Effect

Button subtly follows the cursor when hovered, creating a magnetic pull effect.

```tsx
'use client';
import { motion, useMotionValue, useSpring } from 'framer-motion';
import { useRef } from 'react';

export function MagneticButton({ children, ...props }: React.ComponentProps<typeof motion.button>) {
  const ref = useRef<HTMLButtonElement>(null);
  const x = useMotionValue(0);
  const y = useMotionValue(0);
  const springX = useSpring(x, { stiffness: 300, damping: 20 });
  const springY = useSpring(y, { stiffness: 300, damping: 20 });

  const handleMouse = (e: React.MouseEvent) => {
    if (!ref.current) return;
    const rect = ref.current.getBoundingClientRect();
    const centerX = rect.left + rect.width / 2;
    const centerY = rect.top + rect.height / 2;
    x.set((e.clientX - centerX) * 0.3); // 0.3 = magnetic strength
    y.set((e.clientY - centerY) * 0.3);
  };

  const handleLeave = () => {
    x.set(0);
    y.set(0);
  };

  return (
    <motion.button
      ref={ref}
      style={{ x: springX, y: springY }}
      onMouseMove={handleMouse}
      onMouseLeave={handleLeave}
      {...props}
    >
      {children}
    </motion.button>
  );
}
```

**When to use:** Primary CTA buttons, hero buttons. Desktop only — has no effect on touch devices.

**Rules:**
- Magnetic strength: `0.2` – `0.4` range. Higher = more dramatic.
- Spring damping: `15` – `25`. Lower = bouncier. Higher = snappier.
- Only apply to 1-2 buttons per page — the primary CTA in hero and/or final CTA.

#### `tilt-card` — 3D Perspective Tilt on Hover

Cards tilt toward the cursor with a 3D perspective effect.

```tsx
'use client';
import { motion, useMotionValue, useSpring, useTransform } from 'framer-motion';

export function TiltCard({ children }: { children: React.ReactNode }) {
  const x = useMotionValue(0.5);
  const y = useMotionValue(0.5);

  const rotateX = useSpring(useTransform(y, [0, 1], [8, -8]), { stiffness: 300, damping: 30 });
  const rotateY = useSpring(useTransform(x, [0, 1], [-8, 8]), { stiffness: 300, damping: 30 });

  const handleMouse = (e: React.MouseEvent<HTMLDivElement>) => {
    const rect = e.currentTarget.getBoundingClientRect();
    x.set((e.clientX - rect.left) / rect.width);
    y.set((e.clientY - rect.top) / rect.height);
  };

  const handleLeave = () => {
    x.set(0.5);
    y.set(0.5);
  };

  return (
    <motion.div
      style={{ rotateX, rotateY, transformPerspective: 800 }}
      onMouseMove={handleMouse}
      onMouseLeave={handleLeave}
    >
      {children}
    </motion.div>
  );
}
```

**When to use:** Feature cards, pricing cards, testimonial cards. Adds depth and interactivity.

**Rules:**
- Tilt range: `6` – `10` degrees. More than 12 feels disorienting.
- `transformPerspective`: `600` – `1000`. Lower = more dramatic. Higher = subtler.
- Desktop only — falls back to flat on touch devices.
- Can combine with a subtle glow that follows the cursor for extra depth.

#### `marquee` — Infinite Scroll Ticker

Content scrolls horizontally in an infinite loop. No interaction needed.

```css
.marquee {
  display: flex;
  overflow: hidden;
  white-space: nowrap;
}

.marqueeTrack {
  display: flex;
  gap: var(--mantine-spacing-xl);
  animation: marquee 30s linear infinite;
}

/* Duplicate content for seamless loop */
.marqueeTrack[data-direction="reverse"] {
  animation-direction: reverse;
}

@keyframes marquee {
  from { transform: translateX(0); }
  to { transform: translateX(-50%); }
}

@media (prefers-reduced-motion: reduce) {
  .marqueeTrack { animation-play-state: paused; }
}
```

```tsx
// Render content twice for seamless loop
<div className={classes.marquee}>
  <div className={classes.marqueeTrack}>
    {logos.map((logo, i) => <LogoItem key={`a-${i}`} logo={logo} />)}
    {logos.map((logo, i) => <LogoItem key={`b-${i}`} logo={logo} />)}
  </div>
</div>
```

**When to use:** Logo bars (replaces static `logo-bar` archetype), testimonial snippets, announcement ribbons, tech stack icons.

**Rules:**
- Duplicate the content array so the loop is seamless (scroll distance = 50% = one set of items)
- Speed: `20s` – `40s` for logos, `40s` – `60s` for text-heavy content. Faster feels frantic.
- Pause on hover (`animation-play-state: paused` on `.marquee:hover .marqueeTrack`) for accessibility
- `aria-hidden="true"` on the duplicate set so screen readers don't read content twice
- Optional: add a gradient mask on edges to fade content in/out: `mask-image: linear-gradient(to right, transparent, black 10%, black 90%, transparent)`

#### `cursor-spotlight` — Cursor-Following Spotlight

A radial gradient follows the cursor, creating a spotlight effect on a section background.

```tsx
'use client';
import { useMotionValue, motion, useMotionTemplate } from 'framer-motion';

export function SpotlightSection({ children }: { children: React.ReactNode }) {
  const mouseX = useMotionValue(0);
  const mouseY = useMotionValue(0);

  const background = useMotionTemplate`
    radial-gradient(600px circle at ${mouseX}px ${mouseY}px,
      rgba(var(--mantine-color-brand-5-rgb), 0.08),
      transparent 80%)
  `;

  return (
    <motion.section
      style={{ background, position: 'relative' }}
      onMouseMove={(e) => {
        const rect = e.currentTarget.getBoundingClientRect();
        mouseX.set(e.clientX - rect.left);
        mouseY.set(e.clientY - rect.top);
      }}
    >
      {children}
    </motion.section>
  );
}
```

**When to use:** Feature grid sections, pricing sections on dark backgrounds. Subtle interactive depth.

**Performance:** `useMotionTemplate` re-renders only the CSS property, not React components. GPU-composited via `background`. Desktop only.

### Advanced Layouts

#### `bento-grid` — Bento Box Layout

An asymmetric grid where cards have different sizes and spans, like Apple's product pages.

```tsx
<Grid>
  {/* Large featured card */}
  <Grid.Col span={{ base: 12, md: 8 }}>
    <Card h="100%">{/* featured content */}</Card>
  </Grid.Col>
  {/* Two stacked smaller cards */}
  <Grid.Col span={{ base: 12, md: 4 }}>
    <Stack h="100%">
      <Card flex={1}>{/* small card 1 */}</Card>
      <Card flex={1}>{/* small card 2 */}</Card>
    </Stack>
  </Grid.Col>
  {/* Three equal cards on next row */}
  <Grid.Col span={{ base: 12, sm: 6, md: 4 }}><Card>{/* card 3 */}</Card></Grid.Col>
  <Grid.Col span={{ base: 12, sm: 6, md: 4 }}><Card>{/* card 4 */}</Card></Grid.Col>
  <Grid.Col span={{ base: 12, md: 4 }}><Card>{/* card 5 */}</Card></Grid.Col>
</Grid>
```

Or using CSS Grid for more control:

```css
.bentoGrid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-auto-rows: minmax(180px, auto);
  gap: var(--mantine-spacing-md);
}

.bentoGrid > :nth-child(1) { grid-column: span 2; grid-row: span 2; }
.bentoGrid > :nth-child(2) { grid-column: span 2; }
.bentoGrid > :nth-child(3) { grid-column: span 2; }

@media (max-width: 48em) {
  .bentoGrid {
    grid-template-columns: 1fr;
  }
  .bentoGrid > * {
    grid-column: span 1 !important;
    grid-row: span 1 !important;
  }
}
```

**When to use:** Replaces standard `grid-cards` archetype when a more visual, editorial layout is desired. Feature showcases, dashboard previews, mixed-media content displays.

**Rules:**
- On mobile: all cards collapse to single column, equal height
- Featured (large) card should contain the most important content
- All cards must have the same border radius and gap for visual cohesion
- Cards can contain: text, images, mini charts, code snippets, embedded UI elements

#### `masonry` — Masonry / Pinterest Layout

Cards of varying heights arranged in columns without fixed rows.

```css
.masonry {
  columns: 3;
  column-gap: var(--mantine-spacing-md);
}

.masonry > * {
  break-inside: avoid;
  margin-bottom: var(--mantine-spacing-md);
}

@media (max-width: 62em) { .masonry { columns: 2; } }
@media (max-width: 48em) { .masonry { columns: 1; } }
```

**When to use:** Testimonials with varying quote lengths, portfolio items, blog previews, image galleries.

**Rules:**
- CSS `columns` is the simplest implementation — no JS needed
- Items render top-to-bottom per column, not left-to-right across rows. If order matters, use CSS Grid with `grid-auto-rows: 1px` and dynamic `grid-row: span N` per item (requires JS to measure heights).
- Mobile: single column — masonry has no effect

#### `floating-elements` — Decorative Floating Elements

Small decorative elements (shapes, icons, gradients) that float around a section, adding depth.

```tsx
'use client';
import { motion } from 'framer-motion';

const floatingElements = [
  { size: 60, x: '10%', y: '20%', delay: 0, duration: 6 },
  { size: 40, x: '80%', y: '30%', delay: 2, duration: 8 },
  { size: 80, x: '60%', y: '70%', delay: 1, duration: 7 },
];

export function FloatingDecorations() {
  return (
    <div style={{ position: 'absolute', inset: 0, overflow: 'hidden', pointerEvents: 'none' }}>
      {floatingElements.map((el, i) => (
        <motion.div
          key={i}
          style={{
            position: 'absolute',
            width: el.size,
            height: el.size,
            left: el.x,
            top: el.y,
            borderRadius: '50%',
            background: 'var(--mantine-color-brand-2)',
            opacity: 0.15,
          }}
          animate={{
            y: [0, -20, 0],
            x: [0, 10, 0],
            scale: [1, 1.05, 1],
          }}
          transition={{
            duration: el.duration,
            repeat: Infinity,
            delay: el.delay,
            ease: 'easeInOut',
          }}
        />
      ))}
    </div>
  );
}
```

**When to use:** Behind hero sections, around CTA blocks. Pure decoration. Never let floating elements obstruct text or interactive elements.

**Rules:**
- `pointer-events: none` — decorations must not capture clicks
- `overflow: hidden` on parent — prevent decorations from causing horizontal scroll
- Keep opacity low (0.1–0.2) — they should be noticed subconsciously, not stared at
- 3–5 elements max — more feels cluttered
- Disable entirely on `prefers-reduced-motion`

### Visual Effects Rules

1. **Less is more** — apply 2-3 effects per page maximum. Every effect on the same page competes for attention. A single well-executed `sticky-scroll` section is more impactful than parallax + glow + marquee + floating elements all on one page.

2. **Progressive enhancement** — the page must work perfectly without any visual effects. Effects are layered on top, not load-bearing. Content must be readable and navigable with `prefers-reduced-motion: reduce`.

3. **Mobile fallback is mandatory** — every effect must specify its mobile behavior:
   - `sticky-scroll` → vertical stack
   - `horizontal-scroll` → vertical stack or carousel
   - cursor effects (`magnetic-button`, `tilt-card`, `cursor-spotlight`) → disabled (no cursor on touch)
   - `parallax` → static positioned background
   - `marquee` → static row or auto-paused

4. **Performance budget** — effects must not push Core Web Vitals beyond targets:
   - No `box-shadow` animations (use `filter: drop-shadow` or pseudo-elements instead)
   - `filter: blur()` on elements > 500px: limit to 2 simultaneous instances
   - `backdrop-filter`: limit to 3 simultaneous glass elements
   - Continuous animations (`marquee`, `float`, `aurora`): use CSS animations, not Framer Motion — CSS animations don't trigger React re-renders
   - Test every effect at 4× CPU throttle in Chrome DevTools

5. **Combine effects with archetypes** — effects enhance catalog sections:
   - `logo-bar` → can be upgraded to `marquee`
   - `grid-cards` → can use `bento-grid` layout, `tilt-card` hover, `cursor-spotlight` background
   - `stats-bar` → can use `counter` animations
   - `cta-block` → can use `mesh-gradient` or `aurora` background, `glow` on button
   - `split-content` → can use `parallax` on the image side
   - `steps` → can use `sticky-scroll` for dramatic reveal
   - Hero → can use `text-reveal` headline, `magnetic-button` CTA, `mesh-gradient` background
