# Next.js Conventions

## App Router

- Use **Server Components** by default. Add `"use client"` only when the component needs:
  - React hooks (`useState`, `useEffect`, etc.)
  - Browser APIs (`window`, `document`)
  - Event handlers (`onClick`, `onChange`)
  - Mantine interactive components (`useForm`, `useDisclosure`, etc.)
- Keep server components lean — fetch data at the page/layout level, pass down as props.

## Data Mutations & Fetching

- **Mutations**: Use **Server Actions** by default (see below). Only use API Route Handlers for external/public APIs and webhooks.
- **Server data**: Use `async` Server Components with direct database calls or Server Actions.
- **Client data**: Use **TanStack Query** with Server Actions as `queryFn` — never raw `useEffect` + `fetch`. See [state-management.md](./state-management.md) for patterns.
- API Route Handlers (`app/api/`) are reserved for external consumers, webhooks, and streaming. Always validate input with **Zod** (see [validation.md](./validation.md)).

## Metadata

- Use the `metadata` export or `generateMetadata()` for SEO — never raw `<head>` tags.

## Images

- Use `next/image` for all images — never raw `<img>`.

## Links

- Use `next/link` for all internal navigation — never raw `<a>` for internal routes.

---

# Server Actions

## Default Over API Routes

Use **Server Actions** as the default for all data mutations (create, update, delete) and simple data fetches from client components. Only use API Route Handlers (`app/api/`) when:

- You need a **public API** consumed by external clients or third-party services
- You need **webhook endpoints** (e.g. Stripe, GitHub)
- You need **streaming responses** or long-running HTTP connections
- The user **explicitly requests** an API route

| Use case | Solution |
|----------|----------|
| Form submission (create/update/delete) | **Server Action** |
| Client-side data fetch (TanStack Query) | **Server Action** |
| Revalidating cached data | **Server Action** with `revalidatePath` / `revalidateTag` |
| Public/external API | API Route Handler |
| Webhook receiver | API Route Handler |
| File upload with progress | API Route Handler |

## File Organization

Server Actions live in `actions/`, grouped by domain. Each file must start with `"use server"`.

```
src/actions/
├── userActions.ts          # createUser, updateUser, deleteUser
├── authActions.ts          # login, logout, register
├── projectActions.ts       # createProject, updateProject
└── uploadActions.ts        # uploadAvatar, uploadDocument
```

## Action Pattern

```tsx
// actions/userActions.ts
"use server";

import { revalidatePath } from "next/cache";
import { createUserSchema, updateUserSchema } from "@/lib/validations/user";

// Return type for consistent error handling
interface IActionResult<T = void> {
  success: boolean;
  data?: T;
  error?: string;
}

export async function createUser(formData: FormData): Promise<IActionResult<{ id: string }>> {
  const raw = Object.fromEntries(formData);
  const parsed = createUserSchema.safeParse(raw);

  if (!parsed.success) {
    return { success: false, error: parsed.error.issues[0].message };
  }

  try {
    const user = await db.user.create({ data: parsed.data });
    revalidatePath("/users");
    return { success: true, data: { id: user.id } };
  } catch {
    return { success: false, error: "Failed to create user" };
  }
}

export async function updateUser(id: string, formData: FormData): Promise<IActionResult> {
  const raw = Object.fromEntries(formData);
  const parsed = updateUserSchema.safeParse({ ...raw, id });

  if (!parsed.success) {
    return { success: false, error: parsed.error.issues[0].message };
  }

  try {
    await db.user.update({ where: { id }, data: parsed.data });
    revalidatePath("/users");
    revalidatePath(`/users/${id}`);
    return { success: true };
  } catch {
    return { success: false, error: "Failed to update user" };
  }
}

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

## Server Actions for Data Fetching (with TanStack Query)

Server Actions can also serve as the `queryFn` for TanStack Query, replacing API route calls:

```tsx
// actions/userActions.ts
"use server";

export async function getUsers(): Promise<IUser[]> {
  return db.user.findMany();
}

export async function getUserById(id: string): Promise<IUser | null> {
  return db.user.findUnique({ where: { id } });
}
```

```tsx
// hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { getUsers, createUser } from "@/actions/userActions";

export function useUsers() {
  return useQuery({
    queryKey: userKeys.lists(),
    queryFn: () => getUsers(),
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (formData: FormData) => createUser(formData),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
    },
  });
}
```

## Using with Mantine Forms

```tsx
"use client";

import { useForm, zodResolver } from "@mantine/form";
import { TextInput, Select, Button } from "@mantine/core";
import { notifications } from "@mantine/notifications";
import { createUser } from "@/actions/userActions";
import { createUserSchema } from "@/lib/validations/user";

export function CreateUserForm() {
  const form = useForm({
    initialValues: { email: "", displayName: "", role: "viewer" as const },
    validate: zodResolver(createUserSchema),
  });

  async function handleSubmit(values: typeof form.values) {
    const formData = new FormData();
    Object.entries(values).forEach(([key, value]) => formData.append(key, value));

    const result = await createUser(formData);

    if (result.success) {
      notifications.show({ title: "Success", message: "User created", color: "green" });
      form.reset();
    } else {
      notifications.show({ title: "Error", message: result.error, color: "red" });
    }
  }

  return (
    <form onSubmit={form.onSubmit(handleSubmit)}>
      <TextInput label="Email" {...form.getInputProps("email")} />
      <TextInput label="Name" {...form.getInputProps("displayName")} />
      <Select label="Role" data={["admin", "editor", "viewer"]} {...form.getInputProps("role")} />
      <Button type="submit" mt="md">Create</Button>
    </form>
  );
}
```

## Rules

- **Always add `"use server"` at the top of the file** — never inline `"use server"` inside individual functions.
- **Always validate input** with Zod before processing — Server Actions are public endpoints.
- **Return structured results** (`IActionResult<T>`) — never throw errors to the client; return `{ success: false, error }`.
- **Call `revalidatePath` or `revalidateTag`** after mutations to keep the UI in sync.
- **Keep actions thin** — validate, call the service layer, revalidate. Business logic belongs in `services/`.
- **Never return sensitive data** (passwords, tokens, internal errors) in action results.
- **One file per domain** — `userActions.ts`, `authActions.ts`, not one giant `actions.ts`.
