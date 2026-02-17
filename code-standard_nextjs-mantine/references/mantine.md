# Mantine Usage

## Styling Priority (highest to lowest)

1. **Mantine component props** — `<Button size="lg" color="blue">`
2. **Mantine `style` prop with CSS variables** — `style={{ marginTop: "var(--mantine-spacing-md)" }}`
3. **CSS Modules** (`*.module.css`) — for complex, component-specific styles
4. **Never** use inline style objects with pixel values, Tailwind, or styled-components.

## Theme

- Define theme overrides in `theme/index.ts` using `createTheme()`.
- Use Mantine CSS variables and spacing scale (`xs`, `sm`, `md`, `lg`, `xl`) — never hardcoded pixel values for spacing.
- Use `theme.colors.brand` for primary brand color, configured in theme.

```tsx
// theme/index.ts
import { createTheme } from "@mantine/core";

export const theme = createTheme({
  primaryColor: "brand",
  colors: {
    brand: [
      "#f0f4ff", "#d9e2ff", "#b3c6ff", "#8da9ff",
      "#668dff", "#4070ff", "#1a54ff", "#0040e6",
      "#0033b3", "#002680",
    ],
  },
  fontFamily: "'Inter', sans-serif",
  headings: { fontFamily: "'Inter', sans-serif" },
  defaultRadius: "md",
});
```

## Component Usage Patterns

```tsx
// Use Mantine layout components, not raw HTML
// Good
<Stack gap="md">
  <Group justify="space-between">
    <Text>Label</Text>
    <Badge>Status</Badge>
  </Group>
</Stack>

// Bad
<div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
  <div style={{ display: "flex", justifyContent: "space-between" }}>
    <span>Label</span>
    <span className="badge">Status</span>
  </div>
</div>
```

## Forms

- Use `@mantine/form` with `useForm()` for form state management.
- Use **Zod schemas** with `zodResolver` for form validation — see [validation.md](./validation.md) for full pattern.
- Use Mantine form components (`TextInput`, `Select`, `Checkbox`, etc.) — never raw HTML inputs.
- Always provide a `label` prop on every input for accessibility (see [accessibility.md](./accessibility.md)).

```tsx
import { useForm, zodResolver } from "@mantine/form";
import { TextInput, Button } from "@mantine/core";
import { loginSchema } from "@/lib/validations/auth";

export function LoginForm() {
  const form = useForm({
    initialValues: { email: "", password: "" },
    validate: zodResolver(loginSchema),
  });

  function handleSubmit(values: typeof form.values) {
    // submit logic
  }

  return (
    <form onSubmit={form.onSubmit(handleSubmit)}>
      <TextInput label="Email" {...form.getInputProps("email")} />
      <TextInput label="Password" type="password" {...form.getInputProps("password")} />
      <Button type="submit" mt="md">Log in</Button>
    </form>
  );
}
```

## Notifications

- Use `@mantine/notifications` with `notifications.show()` — never `window.alert()`.

## Modals

- Use `@mantine/modals` with the modals manager — never a custom modal implementation unless Mantine modals are insufficient.
