# Validation (Zod)

## Principles

- **Validate at system boundaries**: Server Action inputs, API route inputs, form submissions, environment variables, external API responses.
- **Single source of truth**: Define a Zod schema once, derive TypeScript types from it with `z.infer<>`.
- **Schemas in `lib/validations/`** for shared schemas. Colocate with the route/component if used only once.

## Schema Conventions

```tsx
// lib/validations/user.ts
import { z } from "zod";

// Schema definition
export const createUserSchema = z.object({
  email: z.string().email("Invalid email address"),
  displayName: z.string().min(2, "Name must be at least 2 characters"),
  role: z.enum(["admin", "editor", "viewer"]),
});

// Derive type from schema — never define separately
export type TCreateUserInput = z.infer<typeof createUserSchema>;

// Update schema can extend create schema
export const updateUserSchema = createUserSchema.partial().extend({
  id: z.string().uuid(),
});

export type TUpdateUserInput = z.infer<typeof updateUserSchema>;
```

## Naming

| Schema | Convention | Example |
|--------|-----------|---------|
| Create/input | `create[Entity]Schema` | `createUserSchema` |
| Update | `update[Entity]Schema` | `updateUserSchema` |
| Query params | `[entity]QuerySchema` | `userQuerySchema` |
| API response | `[entity]ResponseSchema` | `userResponseSchema` |
| Env variables | `envSchema` | `envSchema` (one file) |

## Server Action Validation

Always validate input in Server Actions before processing (see [nextjs.md](./nextjs.md) for full action pattern):

```tsx
"use server";

export async function createUser(formData: FormData): Promise<IActionResult> {
  const raw = Object.fromEntries(formData);
  const parsed = createUserSchema.safeParse(raw);

  if (!parsed.success) {
    return { success: false, error: parsed.error.issues[0].message };
  }

  // parsed.data is fully typed as TCreateUserInput
  const user = await db.user.create({ data: parsed.data });
  return { success: true, data: { id: user.id } };
}
```

## API Route Validation

When using API Route Handlers (for external APIs/webhooks only — see [nextjs.md](./nextjs.md)), always validate request bodies:

```tsx
// app/api/users/route.ts
import { NextResponse } from "next/server";
import { createUserSchema } from "@/lib/validations/user";

export async function POST(req: Request) {
  const body = await req.json();
  const parsed = createUserSchema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json(
      { error: "Validation failed", details: parsed.error.flatten() },
      { status: 400 },
    );
  }

  // parsed.data is fully typed as TCreateUserInput
  const user = await createUser(parsed.data);
  return NextResponse.json(user, { status: 201 });
}
```

## Form Validation with Mantine + Zod

Use `zodResolver` from `mantine-form-zod-resolver` to connect Zod schemas with `@mantine/form`:

```tsx
import { useForm, zodResolver } from "@mantine/form";
import { createUserSchema } from "@/lib/validations/user";

export function CreateUserForm() {
  const form = useForm({
    initialValues: { email: "", displayName: "", role: "viewer" as const },
    validate: zodResolver(createUserSchema),
  });

  // form.getInputProps() automatically shows Zod error messages
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

## Environment Variables

Validate all env vars at startup with Zod:

```tsx
// lib/env.ts
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: z.string().startsWith("pk_"),
  CLERK_SECRET_KEY: z.string().startsWith("sk_"),
  CLERK_WEBHOOK_SECRET: z.string().startsWith("whsec_"),
  NEXT_PUBLIC_CLERK_SIGN_IN_URL: z.string().default("/sign-in"),
  NEXT_PUBLIC_CLERK_SIGN_UP_URL: z.string().default("/sign-up"),
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
});

// Skip validation in test environment — env vars are not available
const isTest = process.env.NODE_ENV === "test" || process.env.VITEST === "true";

export const env = isTest
  ? (process.env as unknown as z.infer<typeof envSchema>)
  : envSchema.parse(process.env);
```

The test bypass is necessary because Vitest runs in isolation without `.env` values. The env module is imported transitively through `db/index.ts` and Server Actions, which would crash all tests that touch those modules.

## Rules

- **Never trust client input** — always validate on the server, even if the client also validates.
- **Never define TypeScript types separately** from schemas — use `z.infer<>` to derive them.
- **Use `.safeParse()`** in Server Actions and API routes (return error result), use `.parse()` only where you want to throw (env validation, internal logic).
- **Keep schemas lean** — validate shape and basic constraints. Complex business rules belong in the service layer.
