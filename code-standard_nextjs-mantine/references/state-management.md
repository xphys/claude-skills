# State Management

## Decision Tree

Choose the right tool based on the type of state:

| State type | Solution | Example |
|-----------|----------|---------|
| Server data (async) | **TanStack Query** | User list, dashboard stats, search results |
| Global client state | **Zustand** | Auth session, sidebar open/closed, user preferences |
| Local component state | **useState / useReducer** | Form inputs, toggles, dialog visibility |
| URL state | **Next.js searchParams** | Filters, pagination, sort order |
| Form state | **@mantine/form** | Form values, validation, dirty tracking |

---

## TanStack Query (Server State)

All server data fetching on the client must go through TanStack Query. Use **Server Actions** as the `queryFn` / `mutationFn` — never raw `fetch("/api/...")` for internal data.

```tsx
// hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { getUsers, getUserById, createUser } from "@/actions/userActions";

// Query key factory — keeps keys consistent and type-safe
export const userKeys = {
  all: ["users"] as const,
  lists: () => [...userKeys.all, "list"] as const,
  list: (filters: Record<string, unknown>) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, "detail"] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};

export function useUsers() {
  return useQuery({
    queryKey: userKeys.lists(),
    queryFn: () => getUsers(),
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: userKeys.detail(id),
    queryFn: () => getUserById(id),
    enabled: !!id,
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

### TanStack Query Rules

- **Query key factory** per entity — define a `[entity]Keys` object for consistent cache keys.
- **One hook per query/mutation** — export from `hooks/` or colocated with the feature.
- **Invalidate, don't manually set cache** — use `invalidateQueries` after mutations.
- **Use `enabled`** to conditionally fetch — never put queries inside `if` blocks.
- **Never call `useQuery` inside `useEffect`** — TanStack Query handles refetching.

---

## Zustand (Global Client State)

Use Zustand for client-side state that is **not** server data and needs to be shared across components.

```tsx
// stores/useAuthStore.ts
import { create } from "zustand";

interface AuthState {
  user: { id: string; email: string } | null;
  isAuthenticated: boolean;
  login: (user: AuthState["user"]) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  isAuthenticated: false,
  login: (user) => set({ user, isAuthenticated: true }),
  logout: () => set({ user: null, isAuthenticated: false }),
}));
```

```tsx
// stores/useUIStore.ts
import { create } from "zustand";

interface UIState {
  isSidebarOpen: boolean;
  toggleSidebar: () => void;
}

export const useUIStore = create<UIState>((set) => ({
  isSidebarOpen: true,
  toggleSidebar: () => set((state) => ({ isSidebarOpen: !state.isSidebarOpen })),
}));
```

### Zustand Rules

- **One store per domain** — `useAuthStore`, `useUIStore`, `useSettingsStore`. Not one giant global store.
- **File naming**: `stores/use[Domain]Store.ts`.
- **Keep stores flat** — avoid deeply nested state objects.
- **Selectors for performance** — select only the slice you need to avoid unnecessary re-renders:

```tsx
// Good — only re-renders when isSidebarOpen changes
const isSidebarOpen = useUIStore((state) => state.isSidebarOpen);

// Bad — re-renders on ANY store change
const { isSidebarOpen } = useUIStore();
```

- **No async logic in stores** — data fetching belongs in TanStack Query, not Zustand.
- **Use `persist` middleware** only for state that must survive page refresh (e.g. user preferences, theme choice).

### When NOT to Reach for Zustand

- Data from the server → use **TanStack Query**
- State used by one component → use **useState**
- State derived from URL → use **searchParams / useSearchParams**
- Form values → use **@mantine/form**
