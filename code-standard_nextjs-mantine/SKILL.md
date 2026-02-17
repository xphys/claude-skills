---
name: code-standard
description: Coding conventions and standards for Next.js + Mantine projects. Use when writing, reviewing, or refactoring code. Covers TypeScript, React components, Server Actions, Mantine styling, Clerk auth, Drizzle ORM, Zod validation, TanStack Query, Zustand, testing, and accessibility.
user-invocable: false
---

# Code Standard — Next.js + Mantine

> **Package manager**: Use `pnpm` exclusively. Never use `npm` or `yarn`.
>
> **Detailed references** are in `references/`. Read the relevant file before working on that topic.

## Reference Files

Read these **before** working on the related area:

| Topic | File | When to read |
|-------|------|-------------|
| Mantine components, styling, theme, forms | [references/mantine.md](references/mantine.md) | Building UI, styling, form components |
| Next.js App Router, Server Actions | [references/nextjs.md](references/nextjs.md) | Pages, layouts, data fetching, mutations, API routes |
| Testing (Vitest, Playwright) | [references/testing.md](references/testing.md) | Writing or reviewing tests |
| Accessibility (a11y) | [references/accessibility.md](references/accessibility.md) | Building UI, forms, navigation, modals |
| Validation (Zod) | [references/validation.md](references/validation.md) | Schemas, form validation, API input, env vars |
| State management (TanStack Query, Zustand) | [references/state-management.md](references/state-management.md) | Data fetching, global state, choosing state solution |
| Modules (complex features) | [references/modules.md](references/modules.md) | Building features with own sub-components, actions, types, hooks |
| Authentication (Clerk) | [references/auth.md](references/auth.md) | Auth, route protection, RBAC, webhooks, Clerk components |
| Database (Drizzle ORM + PostgreSQL) | [references/database.md](references/database.md) | Schema, queries, migrations, transactions, services layer |

---

## 1. Project Structure

```
.env.example                   # Template of all env vars (no real values — update when env changes)
src/
├── app/                    # Next.js App Router pages & layouts
│   ├── (auth)/             # Route group: authenticated pages
│   ├── (public)/           # Route group: public pages
│   │   ├── sign-in/        # Clerk sign-in page
│   │   └── sign-up/        # Clerk sign-up page
│   ├── api/                # Route handlers (external/webhooks only)
│   ├── layout.tsx          # Root layout (ClerkProvider + MantineProvider)
│   └── global.css          # Global styles (minimal — prefer Mantine)
├── components/
│   ├── ui/                 # Generic reusable UI components
│   ├── forms/              # Form-specific components
│   └── layouts/            # Layout shells, navbars, sidebars
├── modules/                # Complex feature modules (see references/modules.md)
│   └── [feature]/          # Self-contained: components/, hooks/, actions/, types/
├── hooks/                  # Shared custom React hooks
├── lib/                    # Utilities, helpers, constants
│   ├── api.ts              # API client / fetch wrappers
│   ├── utils.ts            # General utility functions
│   ├── constants.ts        # App-wide constants
│   ├── auth.ts             # Auth helpers (role checks, etc.)
│   ├── env.ts              # Zod-validated environment variables
│   └── validations/        # Zod schemas (shared between client & server)
├── db/                     # Drizzle ORM (see references/database.md)
│   ├── index.ts            # Database client (drizzle instance)
│   ├── schema/             # Table definitions (one file per domain)
│   └── migrations/         # Generated migration files
├── actions/                # Server Actions (grouped by domain)
├── services/               # Business logic & database queries
├── stores/                 # Zustand stores
├── types/                  # Shared TypeScript types & interfaces
├── theme/                  # Mantine theme overrides & custom components
│   ├── index.ts            # createTheme() export
│   └── components/         # Mantine component style overrides
├── __tests__/              # Integration tests & test utilities
│   ├── setup.ts            # Vitest global setup (jsdom mocks for Mantine)
│   └── utils.tsx           # Custom render, providers wrapper
└── e2e/                    # Playwright end-to-end tests
    ├── fixtures/           # Test fixtures & page objects
    └── *.spec.ts           # E2E test files
```

### File Naming

| Category       | Convention            | Example                        |
| -------------- | --------------------- | ------------------------------ |
| Components     | PascalCase            | `UserProfile.tsx`              |
| Hooks          | camelCase, `use`      | `useAuth.ts`                   |
| Utilities      | camelCase             | `formatDate.ts`                |
| Types          | camelCase             | `user.ts` (exports `IUser`)    |
| Constants      | camelCase file        | `apiRoutes.ts`                 |
| API routes     | `route.ts`            | `app/api/users/route.ts`       |
| Pages          | `page.tsx`            | `app/dashboard/page.tsx`       |
| Test files     | colocated `.test.*`   | `UserProfile.test.tsx`         |
| E2E tests      | `.spec.ts` in `e2e/`  | `e2e/login.spec.ts`            |
| Zod schemas    | camelCase             | `userSchema.ts`                |
| Zustand stores | camelCase, `use`      | `useAuthStore.ts`              |
| Server Actions | camelCase             | `userActions.ts`               |
| Module folders | kebab-case            | `modules/order-management/`    |
| DB schema      | camelCase, plural     | `db/schema/users.ts`           |

---

## 2. TypeScript Conventions

### General Rules

- **Strict mode** enabled (`"strict": true` in tsconfig).
- Prefer `interface` for object shapes; use `type` for unions, intersections, and mapped types.
- Never use `any`. Use `unknown` when the type is truly unknown, then narrow.
- Use `as const` for literal objects/arrays that should not be mutated.
- Prefer named exports over default exports (except for Next.js pages/layouts).

### Naming

| Symbol        | Convention  | Example                     |
| ------------- | ----------- | --------------------------- |
| Interface     | PascalCase  | `interface IUserProfile {}`  |
| Type alias    | PascalCase  | `type TApiResponse<T> = {}` |
| Enum          | PascalCase  | `enum Role { Admin, User }` |
| Constant      | UPPER_SNAKE | `const MAX_RETRIES = 3`     |
| Function      | camelCase   | `function getUser() {}`     |
| Component     | PascalCase  | `function UserCard() {}`    |
| Boolean       | `is/has/can/should` prefix | `isLoading`, `hasError` |

#### NOTE
- Interface starts prefix with `I` such as IUserProfile.
- Type starts with prefix `T` such as TApiResponse<T>

### Example

```tsx
// types/user.ts
export interface IUser {
  id: string;
  email: string;
  displayName: string;
  role: UserRole;
  createdAt: Date;
}

export type TUserRole = "admin" | "editor" | "viewer";

// Prefer string union over enum when values are simple strings
```

---

## 3. React & Component Conventions

### Component Structure

Write components in this order:

```tsx
// 1. Imports
import { useState } from "react";
import { Button, Stack, Text } from "@mantine/core";

// 2. Types (if component-specific; otherwise import from types/)
interface IUserCardProps {
  user: User;
  onEdit: (id: string) => void;
}

// 3. Component (named export, function declaration)
export function UserCard(props: IUserCardProps) {
  // a. Hooks
  const [isExpanded, setIsExpanded] = useState(false);

  // b. Derived state / computations
  const fullName = `${props.user.firstName} ${props.user.lastName}`;

  // c. Handlers
  function handleEdit() {
    props.onEdit(props.user.id);
  }

  // d. Early returns (loading, error, empty)
  if (!props.user) return null;

  // e. Render
  return (
    <Stack gap="sm">
      <Text fw={600}>{fullName}</Text>
      <Button onClick={handleEdit}>Edit</Button>
    </Stack>
  );
}
```

### Rules

- **Named exports** for all components: `export function Foo()` not `export default function Foo()`.
  - Exception: Next.js `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx` which require default exports.
- **No arrow function components** at the top level. Use `function` declarations.
- **Props interface** named `I[Component]Props` and defined directly above the component.
- **No prop spreading** (`{...props}`) unless building a wrapper around a Mantine component.
- **Colocate** small helper components in the same file. Extract to separate files when reused or > 50 lines.

### Hooks

- Custom hooks live in `hooks/` or colocated with the feature.
- Always prefix with `use`.
- Return objects (not arrays) when returning more than 2 values.

```tsx
// hooks/useUser.ts
export function useUser(id: string) {
  // ...
  return { user, isLoading, error, refetch };
}
```

---

## 4. Error Handling

- Use `error.tsx` boundary files for page-level errors.
- Use `try/catch` in Server Actions and API routes; return structured results.
- Display errors to users via Mantine notifications or inline form errors — never `console.error` alone.
- **Server Actions** must return `IActionResult` — never throw to the client (see [references/nextjs.md](references/nextjs.md)).

```tsx
// Server Action — return structured result, never throw
export async function deleteUser(id: string): Promise<IActionResult> {
  try {
    await db.user.delete({ where: { id } });
    revalidatePath("/users");
    return { success: true };
  } catch {
    return { success: false, error: "Failed to delete user" };
  }
}
```

```tsx
// Client — handle action result with Mantine notifications
async function handleDelete(id: string) {
  const result = await deleteUser(id);

  if (result.success) {
    notifications.show({ title: "Deleted", message: "User removed", color: "green" });
  } else {
    notifications.show({ title: "Error", message: result.error, color: "red" });
  }
}
```

---

## 5. Import Order

Organize imports in this order, separated by blank lines:

```tsx
// 1. React / Next.js
import { useState, useEffect } from "react";
import { useRouter } from "next/navigation";
import Image from "next/image";

// 2. Third-party libraries
import { Button, Stack, Text } from "@mantine/core";
import { useForm } from "@mantine/form";
import { notifications } from "@mantine/notifications";
import { useQuery } from "@tanstack/react-query";
import { z } from "zod";

// 3. Internal — absolute imports (use `@/` alias)
import { useAuth } from "@/hooks/useAuth";
import { createUser } from "@/actions/userActions";
import { UserCard } from "@/components/ui/UserCard";
import type { IUser } from "@/types/user";

// 4. Relative imports (colocated files only)
import { helpers } from "./helpers";
import classes from "./UserProfile.module.css";
```

- Always use the `@/` path alias for non-relative imports (configured in tsconfig `paths`).
- Group Mantine imports from the same package on one line.
- Use `import type` for type-only imports.

---

## 6. Git & Code Quality

- **Commits**: Follow conventional commits — `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`.
- **Branch names**: `feat/short-description`, `fix/short-description`.
- **No commented-out code** in commits.
- **No `console.log`** in committed code (use a proper logger if needed).
- **No magic numbers** — extract to named constants.
- **Keep `.env.example` in sync** — whenever a new environment variable is added (to `lib/env.ts`, `.env`, or any config), update `.env.example` with the new key and a placeholder/comment. Never put real secrets in `.env.example`.
- **Formatting**: Use `deno fmt` (120 char line width). Run `pnpm fmt` before committing. Never use Prettier or ESLint for formatting.

### Package Scripts (Formatting)

```json
{
  "scripts": {
    "fmt": "deno fmt --line-width 120",
    "fmt:watch": "deno fmt --line-width 120 --watch"
  }
}
```

| Script | Purpose |
|--------|---------|
| `pnpm fmt` | Format all files (120 char line width) |
| `pnpm fmt:watch` | Format on file change (dev mode) |

---

## 7. Quick Reference Checklist

When writing or reviewing code, verify:

**Architecture**
- [ ] Server Component unless `"use client"` is justified
- [ ] Mutations use Server Actions, not API routes (unless external/webhook)
- [ ] Client data fetching uses TanStack Query with Server Actions as `queryFn`
- [ ] Global client state uses Zustand with selectors
- [ ] URL-driven state uses searchParams, not Zustand

**Modules**
- [ ] Complex features (4+ files, own actions/types) live in `modules/`, not `components/`
- [ ] Barrel file (`index.ts`) only exports the public API
- [ ] External consumers import from barrel, never internal paths
- [ ] Tests colocated: `Component.tsx` + `Component.test.tsx`
- [ ] Module has `[module-name].md` summary (purpose, components, actions)

**Authentication**
- [ ] Middleware protects all routes; public routes explicitly listed
- [ ] Server Actions check `auth()` for `userId` before processing
- [ ] RBAC enforced on server (Server Actions / middleware), not just client
- [ ] Auth state from Clerk hooks (`useAuth`, `useUser`), not Zustand
- [ ] Clerk components used for sign-in/sign-up, not custom forms
- [ ] `types/clerk.d.ts` extends `CustomJwtSessionClaims` for custom metadata (e.g., role)

**Database**
- [ ] DB types from Drizzle `$inferSelect` / `$inferInsert`, not manual interfaces
- [ ] One schema file per domain (`users.ts`, `posts.ts`), not one giant `schema.ts`
- [ ] `db` never imported in client components — server only
- [ ] Multi-table mutations wrapped in `db.transaction()`
- [ ] Database-first: schema changes in SQL first, then `drizzle-kit pull`
- [ ] Relations added manually after `pull` (not auto-generated)

**Server Actions**
- [ ] File starts with `"use server"` (not inline per function)
- [ ] Input validated with Zod before processing
- [ ] Returns `IActionResult<T>` — never throws to client
- [ ] Calls `revalidatePath` / `revalidateTag` after mutations
- [ ] Actions are thin — business logic in `services/`

**Mantine & Styling**
- [ ] Mantine components used instead of raw HTML elements
- [ ] Spacing uses Mantine scale (`xs`–`xl`), no hardcoded pixels
- [ ] Forms use `@mantine/form` with `zodResolver`

**Validation**
- [ ] Server Action / API route inputs validated with Zod `.safeParse()`
- [ ] Types derived from Zod schemas via `z.infer<>`, not defined separately
- [ ] Environment variables validated at startup
- [ ] `.env.example` updated when new env vars are added

**Accessibility**
- [ ] All form inputs have a `label` (not just placeholder)
- [ ] Icon-only buttons have `aria-label`
- [ ] Images have meaningful `alt` text
- [ ] One `h1` per page, headings are sequential
- [ ] Interactive elements are keyboard accessible

**Code Quality**
- [ ] No `any` types
- [ ] Named exports (except Next.js pages)
- [ ] Imports sorted correctly with `@/` alias
- [ ] No `console.log` left in code
- [ ] Boolean variables use `is/has/can/should` prefix

**Testing**
- [ ] New components have tests (behavior, not snapshots)
- [ ] Queries use `getByRole` / `getByLabelText` first
- [ ] Critical user flows covered by E2E tests
- [ ] Test setup includes `window.matchMedia` and `ResizeObserver` mocks (required for Mantine in jsdom)
- [ ] Mantine `Select` tested with `getByRole("textbox", { name })`, not `getByLabelText`
- [ ] Every `.test` file has a companion `.test.md` checklist (updated when tests change)
