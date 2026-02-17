# Accessibility (a11y)

## Core Principles

- All interactive elements must be **keyboard accessible** (reachable via `Tab`, activatable via `Enter`/`Space`).
- All pages must be **navigable by screen reader** with a logical reading order.
- Color contrast must meet **WCAG AA** minimum (4.5:1 for normal text, 3:1 for large text).
- Never rely on **color alone** to convey information (add icons, text, or patterns).

## Component Rules

| Requirement | How |
|------------|-----|
| Form inputs | Always use Mantine's `label` prop — never a floating placeholder as the only label |
| Icon-only buttons | Must have `aria-label` — e.g. `<ActionIcon aria-label="Delete item">` |
| Images | Use `alt` text that describes the content. Use `alt=""` for purely decorative images |
| Loading states | Use `aria-busy="true"` on the loading container |
| Dynamic content | Use `aria-live="polite"` for status updates (e.g. toast notifications, form success) |
| Modals / drawers | Must trap focus and return focus to trigger on close (Mantine handles this by default) |
| Links vs buttons | `<Button>` for actions, `<Anchor>` / `next/link` for navigation — never misuse |
| Skip navigation | Include a "Skip to main content" link for keyboard users |
| Page titles | Every page must have a unique, descriptive `<title>` via Next.js `metadata` |

## Semantic Structure

```tsx
// Good — uses semantic landmark elements
<AppShell>
  <AppShell.Header>          {/* <header> */}
    <nav aria-label="Main">   {/* navigation landmark */}
      ...
    </nav>
  </AppShell.Header>
  <AppShell.Main>             {/* <main> */}
    <Title order={1}>Dashboard</Title>    {/* single h1 per page */}
    <section aria-labelledby="stats-heading">
      <Title order={2} id="stats-heading">Statistics</Title>
      ...
    </section>
  </AppShell.Main>
</AppShell>

// Bad — div soup with no semantics
<div className="header">
  <div className="nav">...</div>
</div>
<div className="content">
  <div className="title">Dashboard</div>
</div>
```

## Heading Hierarchy

- Each page has exactly **one `<Title order={1}>`** (h1).
- Headings must be **sequential** — no skipping from h1 to h3.
- Use Mantine's `<Title order={n}>` component, not raw heading tags.

## Testing a11y

- Run **axe-core** checks in component tests via `vitest-axe` or `jest-axe`.
- Include keyboard navigation in Playwright E2E tests for critical flows.

```tsx
// Component-level a11y check
import { axe } from "vitest-axe";

it("has no accessibility violations", async () => {
  const { container } = render(<LoginForm />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```
