# Testing

## Stack

- **Unit / Component tests**: Vitest + React Testing Library
- **E2E tests**: Playwright
- Test files colocated with source (`.test.tsx`), E2E tests in `e2e/` directory.

## What to Test

| Layer | What to test | What NOT to test |
|-------|-------------|-----------------|
| Components | User interactions, conditional rendering, form validation feedback | Mantine internal behavior, styling details |
| Hooks | Return values, state transitions, error states | Implementation details (internal state) |
| Server Actions | Input validation, success/error return values, revalidation calls | Database internals (mock the DB layer) |
| API routes | Request/response contracts, error responses, status codes | Database internals (mock the DB layer) |
| Utils | Pure logic, edge cases, error cases | Trivial one-liners |
| E2E | Critical user flows (login, checkout, CRUD) | Every possible UI state |

## Component Test Pattern

```tsx
// components/ui/UserCard.test.tsx
import { render, screen } from "@/__tests__/utils";
import { UserCard } from "./UserCard";

const mockUser = {
  id: "1",
  displayName: "Jane Doe",
  email: "jane@example.com",
  role: "admin" as const,
};

describe("UserCard", () => {
  it("renders user information", () => {
    render(<UserCard user={mockUser} />);

    expect(screen.getByText("Jane Doe")).toBeInTheDocument();
    expect(screen.getByText("jane@example.com")).toBeInTheDocument();
  });

  it("calls onEdit when edit button is clicked", async () => {
    const handleEdit = vi.fn();
    const { user } = render(<UserCard user={mockUser} onEdit={handleEdit} />);

    await user.click(screen.getByRole("button", { name: /edit/i }));

    expect(handleEdit).toHaveBeenCalledWith("1");
  });
});
```

## Test Setup (jsdom Mocks for Mantine)

Mantine relies on browser APIs that jsdom does not provide. Add these mocks in the global test setup file:

```tsx
// __tests__/setup.ts
import "@testing-library/jest-dom/vitest";
import "vitest-axe/extend-expect";

// Mantine requires window.matchMedia — jsdom does not provide it
Object.defineProperty(window, "matchMedia", {
  writable: true,
  value: (query: string) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: () => {},
    removeListener: () => {},
    addEventListener: () => {},
    removeEventListener: () => {},
    dispatchEvent: () => false,
  }),
});

// Mantine ScrollArea requires ResizeObserver — jsdom does not provide it
global.ResizeObserver = class ResizeObserver {
  observe() {}
  unobserve() {}
  disconnect() {}
};
```

Without these mocks, any test that renders Mantine components will fail with `window.matchMedia is not a function` or `ResizeObserver is not defined`.

## Test Utilities Setup

```tsx
// __tests__/utils.tsx
import { render, type RenderOptions } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { MantineProvider } from "@mantine/core";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { theme } from "@/theme";

function AllProviders({ children }: { children: React.ReactNode }) {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });

  return (
    <QueryClientProvider client={queryClient}>
      <MantineProvider theme={theme}>{children}</MantineProvider>
    </QueryClientProvider>
  );
}

function customRender(ui: React.ReactElement, options?: RenderOptions) {
  return {
    user: userEvent.setup(),
    ...render(ui, { wrapper: AllProviders, ...options }),
  };
}

export { customRender as render, screen, waitFor, within } from "@testing-library/react";
```

## Testing Rules

- **Query priority** (most to least preferred): `getByRole` > `getByLabelText` > `getByText` > `getByTestId`. Use `data-testid` only as a last resort.
- **Assert behavior, not implementation**: Test what the user sees and does, not component internals.
- **No snapshot tests** unless explicitly justified (they break too often, catch too little).
- **Mock at the boundary**: Mock API calls and services, not internal modules.
- **Each test must be independent**: No shared mutable state between tests.

### Mantine Select Testing

Mantine `Select` renders both an `<input>` and a dropdown `<div role="listbox">` with the same label. Using `getByLabelText("Status")` will find multiple elements and fail. Use `getByRole` instead:

```tsx
// Bad — finds multiple elements (input + listbox)
screen.getByLabelText("Status");

// Good — targets the specific input element
screen.getByRole("textbox", { name: "Status" });
```

## Test Checklist Files (`.test.md`)

Every `.test.tsx` / `.test.ts` file must have a companion `.test.md` file. This is a short, human-readable checklist so developers can quickly view what's tested and add new cases without reading the test code.

Keep it brief — one line per test case, grouped by describe block.

```md
<!-- components/ui/UserCard.test.md -->
# UserCard Tests

- [x] Renders user name and email
- [x] Calls onEdit with user id when edit button is clicked
- [x] Hides edit button when onEdit is not provided
- [ ] Shows role badge with correct color
- [ ] Truncates long display names
```

Rules:
- `[x]` = implemented, `[ ]` = planned (not yet written).
- Update the `.test.md` when adding or removing test cases.
- Agent must update `.test.md` whenever it modifies a `.test` file.

---

## E2E Test Pattern (Playwright)

```ts
// e2e/login.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Login flow", () => {
  test("user can log in with valid credentials", async ({ page }) => {
    await page.goto("/login");

    await page.getByLabel("Email").fill("user@example.com");
    await page.getByLabel("Password").fill("password123");
    await page.getByRole("button", { name: "Log in" }).click();

    await expect(page).toHaveURL("/dashboard");
    await expect(page.getByText("Welcome back")).toBeVisible();
  });

  test("shows error for invalid credentials", async ({ page }) => {
    await page.goto("/login");

    await page.getByLabel("Email").fill("wrong@example.com");
    await page.getByLabel("Password").fill("wrong");
    await page.getByRole("button", { name: "Log in" }).click();

    await expect(page.getByText("Invalid credentials")).toBeVisible();
  });
});
```

## E2E Rules

- Use **Page Object pattern** for reusable selectors on complex pages.
- Prefer `getByRole` and `getByLabel` locators — same accessible-first philosophy as Testing Library.
- E2E tests cover **happy paths and critical error paths** only — don't duplicate unit test coverage.
- Each E2E test must **clean up its own data** or run against a seeded test database.
