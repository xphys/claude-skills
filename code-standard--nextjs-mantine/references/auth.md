# Authentication (Clerk)

## Library

This project uses **Clerk** (`@clerk/nextjs`) for authentication. Clerk handles user management, sign-in/sign-up, session management, and OAuth providers as a managed service.

---

## Route Protection Strategy

### Middleware (Primary — Protects All Routes)

Use Clerk middleware as the **first line of defense**. All routes are protected by default; public routes are explicitly listed.

```tsx
// middleware.ts (project root)
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isPublicRoute = createRouteMatcher([
  "/",
  "/sign-in(.*)",
  "/sign-up(.*)",
  "/api/webhooks(.*)",
]);

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) {
    await auth.protect();
  }
});

export const config = {
  matcher: [
    "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
    "/(api|trpc)(.*)",
  ],
};
```

### Route Groups

Align route groups with auth state:

```
src/app/
├── (public)/               # No auth required (landing, marketing, docs)
│   ├── page.tsx            # Home page
│   ├── pricing/page.tsx
│   ├── sign-in/[[...sign-in]]/page.tsx   # Clerk sign-in page
│   └── sign-up/[[...sign-up]]/page.tsx   # Clerk sign-up page
├── (auth)/                 # Requires authentication
│   ├── layout.tsx          # Auth layout (can check auth here too)
│   ├── dashboard/page.tsx
│   └── settings/page.tsx
```

---

## Accessing Auth State

### Server Components & Server Actions (use `auth()`)

```tsx
// Server Component
import { auth, currentUser } from "@clerk/nextjs/server";

export default async function DashboardPage() {
  // Get session claims (lightweight — no API call)
  const { userId, sessionClaims } = await auth();

  if (!userId) {
    redirect("/sign-in");
  }

  // Get full user object (API call — use only when you need profile data)
  const user = await currentUser();

  return <Title order={1}>Welcome, {user?.firstName}</Title>;
}
```

```tsx
// Server Action
"use server";

import { auth } from "@clerk/nextjs/server";

export async function updateProfile(formData: FormData): Promise<IActionResult> {
  const { userId } = await auth();

  if (!userId) {
    return { success: false, error: "Unauthorized" };
  }

  // ... update logic using userId
}
```

### Client Components (use `useAuth()` / `useUser()`)

```tsx
"use client";

import { useAuth, useUser } from "@clerk/nextjs";

export function UserGreeting() {
  const { isSignedIn, isLoaded } = useAuth();
  const { user } = useUser();

  if (!isLoaded) return <Skeleton height={20} />;
  if (!isSignedIn) return null;

  return <Text>Hello, {user?.firstName}</Text>;
}
```

### Decision Table

| Context | What to use | Returns |
|---------|------------|---------|
| Server Component | `auth()` | `{ userId, sessionClaims, orgId }` |
| Server Action | `auth()` | Same as above |
| API Route Handler | `auth()` | Same as above |
| Client Component (session check) | `useAuth()` | `{ isSignedIn, userId, isLoaded }` |
| Client Component (user profile) | `useUser()` | `{ user, isSignedIn, isLoaded }` |
| Client Component (org data) | `useOrganization()` | `{ organization, membership }` |

---

## Role-Based Access Control (RBAC)

### Define Roles

Use Clerk's session claims or organization roles for RBAC. Define allowed roles as a constant:

```tsx
// lib/constants.ts
export const ROLES = {
  ADMIN: "org:admin",
  EDITOR: "org:editor",
  VIEWER: "org:viewer",
} as const;

export type TRole = (typeof ROLES)[keyof typeof ROLES];
```

### Extend Clerk Session Types

Clerk's `sessionClaims.metadata` is typed as `{}` by default. To access custom metadata (like `role`), create a type declaration file:

```tsx
// types/clerk.d.ts
import type { TRole } from "@/lib/constants";

declare global {
  interface CustomJwtSessionClaims {
    metadata?: {
      role?: TRole;
    };
  }
}
```

Without this, `sessionClaims?.metadata?.role` will cause a TypeScript error (`Property 'role' does not exist on type '{}'`).

### Server-Side Role Check

```tsx
// lib/auth.ts
import { auth } from "@clerk/nextjs/server";
import type { TRole } from "./constants";

export async function checkRole(allowedRoles: TRole[]): Promise<boolean> {
  const { sessionClaims } = await auth();
  const userRole = sessionClaims?.metadata?.role as TRole | undefined;
  return userRole ? allowedRoles.includes(userRole) : false;
}
```

```tsx
// Server Action with role check
"use server";

import { auth } from "@clerk/nextjs/server";
import { ROLES } from "@/lib/constants";

export async function deleteUser(id: string): Promise<IActionResult> {
  const { userId, sessionClaims } = await auth();

  if (!userId) {
    return { success: false, error: "Unauthorized" };
  }

  const userRole = sessionClaims?.metadata?.role;
  if (userRole !== ROLES.ADMIN) {
    return { success: false, error: "Forbidden: admin access required" };
  }

  // ... delete logic
}
```

### Client-Side Role Check (UI only — never trust for security)

```tsx
"use client";

import { useAuth } from "@clerk/nextjs";
import { ROLES } from "@/lib/constants";

export function AdminPanel() {
  const { sessionClaims } = useAuth();
  const userRole = sessionClaims?.metadata?.role;

  // Hide UI only — actual protection is in Server Actions / middleware
  if (userRole !== ROLES.ADMIN) return null;

  return <Stack>{ /* admin content */ }</Stack>;
}
```

---

## Clerk Components

### Sign-In / Sign-Up Pages

Use Clerk's pre-built components. Style them with Clerk's `appearance` prop to match your Mantine theme:

```tsx
// app/sign-in/[[...sign-in]]/page.tsx
import { SignIn } from "@clerk/nextjs";

export default function SignInPage() {
  return (
    <Center h="100vh">
      <SignIn
        appearance={{
          elements: {
            rootBox: { width: "100%" },
            card: { boxShadow: "none" },
          },
        }}
      />
    </Center>
  );
}
```

### User Button

```tsx
// components/layouts/Header.tsx
import { UserButton } from "@clerk/nextjs";

export function Header() {
  return (
    <Group justify="space-between">
      <Title order={3}>My App</Title>
      <UserButton afterSignOutUrl="/" />
    </Group>
  );
}
```

### Rule: Clerk Components vs Custom UI

| Use case | Approach |
|----------|---------|
| Sign-in / Sign-up pages | **Clerk `<SignIn>` / `<SignUp>`** — don't rebuild auth forms |
| User avatar / menu | **Clerk `<UserButton>`** — handles session, sign-out, profile |
| Organization switcher | **Clerk `<OrganizationSwitcher>`** — if using Clerk organizations |
| Custom auth-aware UI | **`useAuth()` / `useUser()`** + Mantine components |

---

## Provider Setup

```tsx
// app/layout.tsx
import { ClerkProvider } from "@clerk/nextjs";
import { MantineProvider } from "@mantine/core";
import { theme } from "@/theme";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body>
          <MantineProvider theme={theme}>
            {children}
          </MantineProvider>
        </body>
      </html>
    </ClerkProvider>
  );
}
```

`ClerkProvider` wraps **outside** `MantineProvider` — auth context must be available before UI renders.

---

## Webhooks

Use Clerk webhooks to sync user data to your database. Webhook endpoints are API Route Handlers (not Server Actions):

```tsx
// app/api/webhooks/clerk/route.ts
import { Webhook } from "svix";
import { headers } from "next/headers";
import type { WebhookEvent } from "@clerk/nextjs/server";
import { env } from "@/lib/env";

export async function POST(req: Request) {
  const headerPayload = await headers();
  const svixId = headerPayload.get("svix-id");
  const svixTimestamp = headerPayload.get("svix-timestamp");
  const svixSignature = headerPayload.get("svix-signature");

  if (!svixId || !svixTimestamp || !svixSignature) {
    return new Response("Missing svix headers", { status: 400 });
  }

  const payload = await req.json();
  const body = JSON.stringify(payload);

  const wh = new Webhook(env.CLERK_WEBHOOK_SECRET);
  let event: WebhookEvent;

  try {
    event = wh.verify(body, {
      "svix-id": svixId,
      "svix-timestamp": svixTimestamp,
      "svix-signature": svixSignature,
    }) as WebhookEvent;
  } catch {
    return new Response("Invalid signature", { status: 400 });
  }

  switch (event.type) {
    case "user.created":
      await db.user.create({
        data: {
          clerkId: event.data.id,
          email: event.data.email_addresses[0]?.email_address ?? "",
          displayName: `${event.data.first_name} ${event.data.last_name}`.trim(),
        },
      });
      break;
    case "user.updated":
      // ... sync updates
      break;
    case "user.deleted":
      // ... handle deletion
      break;
  }

  return new Response("OK", { status: 200 });
}
```

---

## Environment Variables

Add Clerk env vars to your Zod env validation:

```tsx
// lib/env.ts (extend existing schema)
const envSchema = z.object({
  // ... existing vars
  NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: z.string().startsWith("pk_"),
  CLERK_SECRET_KEY: z.string().startsWith("sk_"),
  CLERK_WEBHOOK_SECRET: z.string().startsWith("whsec_"),
  NEXT_PUBLIC_CLERK_SIGN_IN_URL: z.string().default("/sign-in"),
  NEXT_PUBLIC_CLERK_SIGN_UP_URL: z.string().default("/sign-up"),
});
```

---

## Rules

- **Middleware is the primary gate** — never rely solely on layout/page-level checks. The middleware protects all routes by default.
- **Always check `auth()` in Server Actions** — even though middleware protects routes, Server Actions can be called directly. Always verify `userId` exists.
- **RBAC on the server** — client-side role checks are for UI visibility only. Actual authorization must happen in Server Actions, middleware, or API routes.
- **Don't store auth state in Zustand** — use Clerk's `useAuth()` / `useUser()` hooks. Clerk manages the session.
- **Don't rebuild auth UI** — use Clerk's `<SignIn>`, `<SignUp>`, `<UserButton>` components. Customize via `appearance` prop.
- **Sync via webhooks** — use Clerk webhooks to keep your database in sync with Clerk user data. Never trust client-provided user data for database writes.
- **`auth()` over `currentUser()`** — prefer `auth()` (lightweight, no API call) unless you need the full user profile. Use `currentUser()` sparingly.
