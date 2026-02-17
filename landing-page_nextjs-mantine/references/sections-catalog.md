# Section Catalog Reference

The following section archetypes are available. Each archetype defines a layout pattern, not a fixed topic — the user decides the content and naming. A section can combine archetypes (e.g., a "How It Works" section using the `steps` archetype).

When the user requests a landing page, present this catalog and ask which sections to include and in what order. Each chosen section becomes a component file in `components/sections/`.

## `grid-cards` — Grid of Cards

A responsive grid of identical cards. Used for: features, benefits, services, team members, integrations, use cases.

- Layout: `SimpleGrid cols={{ base: 1, sm: 2, lg: 3 }}`
- Each card contains: icon/image (optional), title (`Title order={3}`), description (`Text size="sm" c="dimmed"`)
- Use `Card`, `Paper`, or just a `Stack` per item depending on visual weight
- Icon: `ThemeIcon` wrapping a `@tabler/icons-react` icon
- 3–6 items typical; 8 max before the section feels overwhelming
- Stagger animation on scroll

**Responsive:**
| Viewport | Columns |
|----------|---------|
| `< sm` | 1 |
| `sm` – `lg` | 2 |
| `>= lg` | 3 |

## `split-content` — Content + Image Side-by-Side

A two-column layout with text on one side and an image/illustration on the other. Used for: product showcase, "how it works" detail, feature deep-dive, about us.

- Layout: `Grid` with `span={{ base: 12, md: 6 }}` or `span={{ base: 12, md: 7 }}` / `span={{ base: 12, md: 5 }}`
- Text side: heading (`Title order={2}`), description paragraphs, optional bullet list or CTA button
- Image side: `next/image` with responsive `sizes` prop
- Alternate image left/right between consecutive `split-content` sections for visual rhythm
- On mobile: stacks vertically — text first, image below

**Responsive:**
| Viewport | Layout |
|----------|--------|
| `< md` | Single column, stacked (text → image) |
| `>= md` | Two columns side by side |

## `logo-bar` — Trust / Logo Strip

A horizontal row of logos or trust badges. Used for: client logos, partner logos, "as seen in" press, certification badges, review platform ratings.

- Layout: `Group justify="center" gap="xl"` or horizontal scroll on mobile
- Logos: grayscale + `opacity: 0.6`, full color on hover (desktop only)
- Keep concise — this is a trust accelerator, not a content section
- Optional heading: "Trusted by" or "Featured in" (`Text size="sm" c="dimmed" ta="center"`)

**Responsive:**
| Viewport | Layout |
|----------|--------|
| `< sm` | Wrapping grid or horizontal scroll |
| `>= sm` | Single row, centered |

## `stats-bar` — Key Metrics

A horizontal row of large numbers with labels. Used for: company stats, impact metrics, social proof numbers.

- Layout: `SimpleGrid cols={{ base: 2, sm: 3, lg: 4 }}` or `Group justify="center"`
- Each stat: large number (`Title order={2}` or `Text size="xl" fw={900}`), label below (`Text size="sm" c="dimmed"`)
- 3–4 stats typical; use animated counters (optional, Framer Motion)
- Can be combined with `logo-bar` in the same visual block

**Responsive:**
| Viewport | Columns |
|----------|---------|
| `< sm` | 2 (2×2 grid) |
| `sm` – `lg` | 3 or row |
| `>= lg` | Row or 4 columns |

## `testimonials` — Quotes / Social Proof

Attributed quotes from real users. Used for: customer testimonials, case study excerpts, team quotes.

- Each testimonial: quoted text, person's full name, role/title, company, photo (`Avatar`)
- Anonymous quotes have low credibility — always attribute
- Use `Card` or `Paper` with quotation styling
- 2–3 displayed; use a carousel (`@mantine/carousel`) if more than 3
- Include quantified results when possible: "Reduced deployment time by 40%"

**Responsive:**
| Viewport | Layout |
|----------|--------|
| `< sm` | 1 column stacked, or carousel with swipe |
| `sm` – `lg` | 2 columns |
| `>= lg` | 3 columns |

## `pricing` — Pricing Tiers

Side-by-side pricing plan cards. Used for: subscription plans, service tiers, product editions.

- Lead with value before showing price
- Highlight recommended plan: larger card, accent border, "Most Popular" badge
- Each card: plan name, price, billing period, feature list with check icons, CTA button
- Money-back guarantee or free trial notice prominently displayed
- `SimpleGrid cols={{ base: 1, sm: Math.min(plans.length, 2), lg: plans.length }}`

**Responsive:**
| Viewport | Layout |
|----------|--------|
| `< sm` | 1 column, recommended plan first |
| `sm` – `lg` | 2 columns (if 2 plans) or stacked |
| `>= lg` | All plans side by side |

## `accordion-list` — Expandable Q&A / Details

Collapsible content sections using Mantine `Accordion`. Used for: FAQ, feature details, specification lists, troubleshooting.

- 5–8 items typical
- If used for FAQ: add `FAQPage` JSON-LD structured data (generates rich results in Google)
- On `>= md`, optionally split into two-column layout: heading + description left, accordion right

**Responsive:**
| Viewport | Layout |
|----------|--------|
| All | Single column, full container width |
| `>= md` (optional) | Two-column: intro left, accordion right |

## `cta-block` — Call-to-Action Band

A full-width conversion block. Used for: final CTA, newsletter signup, demo request, download prompt.

- Full-width background (gradient or brand color) to visually separate from content sections
- Contains: headline restating value prop, optional description, primary CTA button, optional secondary CTA
- Keep copy brief — this is a decision point, not an information section
- At least one `cta-block` should appear somewhere after the main content sections (before Footer)

**Responsive:**
| Viewport | Layout |
|----------|--------|
| `< sm` | Stacked, CTA button full width |
| `>= sm` | Centered text, inline CTA button |

## `steps` — Sequential Process

Numbered or ordered steps showing a process flow. Used for: "how it works", onboarding flow, setup guide, methodology.

- 3–5 steps typical
- Each step: number/icon, title, description
- Layout options:
  - Horizontal timeline with connectors on `>= md`, vertical stack on mobile
  - Alternating left/right with `split-content` style
  - Simple numbered `Stack`
- Use `Stepper` component from Mantine or custom layout with `SimpleGrid`

**Responsive:**
| Viewport | Layout |
|----------|--------|
| `< md` | Vertical stack |
| `>= md` | Horizontal timeline or alternating layout |

## `form-capture` — Lead Capture Form

An inline form for collecting user information. Used for: newsletter signup, contact form, demo request, waitlist.

- Use `@mantine/form` for form state management
- Required: email field at minimum. Optional: name, company, message
- Include clear submit button with benefit-driven label ("Get Early Access", not "Submit")
- Success state: show confirmation message via `@mantine/notifications` or inline text
- Error state: inline field validation messages
- Link to Privacy Policy near the submit button

**Responsive:**
| Viewport | Layout |
|----------|--------|
| `< sm` | Stacked fields, full-width button |
| `>= sm` | Inline fields if few (email + button in a row), stacked if many |

## `video-embed` — Video / Demo Section

An embedded video or interactive demo. Used for: product demo, explainer video, customer story.

- Use a responsive video container with 16:9 aspect ratio
- Lazy-load the video iframe — do not load on page init (performance)
- Show a thumbnail/poster image with a play button overlay; load the actual player on click
- Use `AspectRatio` component from Mantine

**Responsive:**
| Viewport | Layout |
|----------|--------|
| All | Full container width, 16:9 aspect ratio maintained |
