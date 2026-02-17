# Modules (Complex Feature Components)

Use `src/modules/` for complex, self-contained features. A module is a **mini-app** — it can contain anything the global project structure has (`components/`, `hooks/`, `actions/`, `stores/`, `validations/`, `types/`, `services/`, `constants/`), but scoped to that feature only. Simple components still go in `components/`.

## When to Use modules/ vs components/

| Criteria | `components/` | `modules/` |
|----------|--------------|------------|
| Sub-components | 0–2, simple | 3+, or complex |
| Own server actions | No | Yes |
| Own types/validations | No (uses shared) | Yes (feature-scoped) |
| Own stores | No | Yes (if needed) |
| Own hooks | 0–1 | 2+ |
| Total files | 1–3 | 4+ |

**Rule of thumb**: If the feature needs its own folder for any domain concern (actions, types, validations, stores, etc.) — it belongs in `modules/`.

## Module Structure

```
src/modules/order-management/
├── order-management.md                  # Human-readable module summary
├── index.ts                              # Public exports (barrel file)
├── OrderManagement.tsx                   # Main entry component
├── OrderManagement.test.tsx              # Tests for main component
├── OrderManagement.test.md              # Test case checklist
├── components/                           # Sub-components (private)
│   ├── OrderTable.tsx
│   ├── OrderTable.test.tsx               # Colocated test
│   ├── OrderFilters.tsx
│   ├── OrderFilters.test.tsx
│   ├── OrderDetailModal.tsx
│   ├── OrderDetailModal.test.tsx
│   └── OrderForm.tsx
├── hooks/                                # Feature-scoped hooks
│   ├── useOrders.ts                      # TanStack Query wrapper
│   ├── useOrders.test.ts
│   └── useOrderFilters.ts
├── actions/                              # Feature-scoped server actions
│   └── orderActions.ts
├── stores/                               # Feature-scoped Zustand stores
│   └── useOrderUIStore.ts
├── validations/                          # Feature-scoped Zod schemas
│   └── orderSchema.ts
├── types/                                # Feature-scoped types
│   └── order.ts
├── services/                             # Feature-scoped business logic (optional)
│   └── orderCalculations.ts
└── constants.ts                          # Feature-scoped constants (optional)
```

A module can have **any or all** of these folders — only create what the feature actually needs. The structure mirrors the global project:

| Global | Module equivalent | Purpose |
|--------|------------------|---------|
| `components/` | `modules/[feature]/components/` | Sub-components private to this feature |
| `hooks/` | `modules/[feature]/hooks/` | Feature-scoped React hooks |
| `actions/` | `modules/[feature]/actions/` | Feature-scoped Server Actions |
| `stores/` | `modules/[feature]/stores/` | Feature-scoped Zustand stores |
| `lib/validations/` | `modules/[feature]/validations/` | Feature-scoped Zod schemas |
| `types/` | `modules/[feature]/types/` | Feature-scoped TypeScript types |
| `services/` | `modules/[feature]/services/` | Feature-scoped business logic |
| `lib/constants.ts` | `modules/[feature]/constants.ts` | Feature-scoped constants |

### Module Naming

- Module folder: **kebab-case** — `order-management/`, `user-profile/`, `inventory-dashboard/`
- Files inside: follow the same conventions as the rest of the project (PascalCase for components, camelCase for hooks/utils)

## The Barrel File (index.ts)

The barrel file defines the module's **public API**. Only export what the rest of the app needs.

```tsx
// modules/order-management/index.ts

// Public — used by pages and other modules
export { OrderManagement } from "./OrderManagement";
export type { IOrder, TOrderStatus } from "./types/order";

// Everything else is PRIVATE:
// - Sub-components (OrderTable, OrderFilters, etc.)
// - Internal hooks (useOrders, useOrderFilters)
// - Actions, validations, constants
```

### Import Rules

```tsx
// Good — import from the module barrel
import { OrderManagement } from "@/modules/order-management";
import type { IOrder } from "@/modules/order-management";

// Bad — reaching into module internals
import { OrderTable } from "@/modules/order-management/components/OrderTable";
import { useOrders } from "@/modules/order-management/hooks/useOrders";
```

Exception: If a hook or type genuinely needs to be shared, **move it to the global `hooks/` or `types/` directory** rather than exporting from the module.

## Full Example: Order Management Module

### Types

```tsx
// modules/order-management/types/order.ts
export interface IOrder {
  id: string;
  customerName: string;
  status: TOrderStatus;
  total: number;
  createdAt: Date;
}

export type TOrderStatus = "pending" | "processing" | "shipped" | "delivered" | "cancelled";
```

### Validation

```tsx
// modules/order-management/validations/orderSchema.ts
import { z } from "zod";

export const createOrderSchema = z.object({
  customerName: z.string().min(1, "Customer name is required"),
  items: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
  })).min(1, "At least one item is required"),
});

export type TCreateOrderInput = z.infer<typeof createOrderSchema>;

export const orderFilterSchema = z.object({
  status: z.enum(["pending", "processing", "shipped", "delivered", "cancelled"]).optional(),
  search: z.string().optional(),
  page: z.number().int().positive().default(1),
});

export type TOrderFilters = z.infer<typeof orderFilterSchema>;
```

### Server Actions

```tsx
// modules/order-management/actions/orderActions.ts
"use server";

import { revalidatePath } from "next/cache";
import { createOrderSchema } from "../validations/orderSchema";
import type { IOrder } from "../types/order";

interface IActionResult<T = void> {
  success: boolean;
  data?: T;
  error?: string;
}

export async function getOrders(): Promise<IOrder[]> {
  return db.order.findMany({ orderBy: { createdAt: "desc" } });
}

export async function createOrder(formData: FormData): Promise<IActionResult<{ id: string }>> {
  const raw = Object.fromEntries(formData);
  const parsed = createOrderSchema.safeParse(raw);

  if (!parsed.success) {
    return { success: false, error: parsed.error.issues[0].message };
  }

  try {
    const order = await db.order.create({ data: parsed.data });
    revalidatePath("/orders");
    return { success: true, data: { id: order.id } };
  } catch {
    return { success: false, error: "Failed to create order" };
  }
}
```

### Hooks

```tsx
// modules/order-management/hooks/useOrders.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { getOrders, createOrder } from "../actions/orderActions";

const orderKeys = {
  all: ["orders"] as const,
  lists: () => [...orderKeys.all, "list"] as const,
};

export function useOrders() {
  return useQuery({
    queryKey: orderKeys.lists(),
    queryFn: () => getOrders(),
  });
}

export function useCreateOrder() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (formData: FormData) => createOrder(formData),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: orderKeys.lists() });
    },
  });
}
```

### Store (Feature-Scoped Zustand)

```tsx
// modules/order-management/stores/useOrderUIStore.ts
import { create } from "zustand";

interface IOrderUIState {
  selectedOrderId: string | null;
  isFilterPanelOpen: boolean;
  selectOrder: (id: string | null) => void;
  toggleFilterPanel: () => void;
}

export const useOrderUIStore = create<IOrderUIState>((set) => ({
  selectedOrderId: null,
  isFilterPanelOpen: false,
  selectOrder: (id) => set({ selectedOrderId: id }),
  toggleFilterPanel: () => set((state) => ({ isFilterPanelOpen: !state.isFilterPanelOpen })),
}));
```

Same Zustand rules as global stores: use selectors, keep flat, no async logic.

### Sub-Component

```tsx
// modules/order-management/components/OrderTable.tsx
import { Table, Badge } from "@mantine/core";
import type { IOrder, TOrderStatus } from "../types/order";

const STATUS_COLORS: Record<TOrderStatus, string> = {
  pending: "yellow",
  processing: "blue",
  shipped: "cyan",
  delivered: "green",
  cancelled: "red",
};

interface IOrderTableProps {
  orders: IOrder[];
  onRowClick: (id: string) => void;
}

export function OrderTable(props: IOrderTableProps) {
  return (
    <Table highlightOnHover>
      <Table.Thead>
        <Table.Tr>
          <Table.Th>Customer</Table.Th>
          <Table.Th>Status</Table.Th>
          <Table.Th>Total</Table.Th>
        </Table.Tr>
      </Table.Thead>
      <Table.Tbody>
        {props.orders.map((order) => (
          <Table.Tr key={order.id} onClick={() => props.onRowClick(order.id)} style={{ cursor: "pointer" }}>
            <Table.Td>{order.customerName}</Table.Td>
            <Table.Td>
              <Badge color={STATUS_COLORS[order.status]}>{order.status}</Badge>
            </Table.Td>
            <Table.Td>${order.total.toFixed(2)}</Table.Td>
          </Table.Tr>
        ))}
      </Table.Tbody>
    </Table>
  );
}
```

### Main Entry Component

```tsx
// modules/order-management/OrderManagement.tsx
"use client";

import { Stack, Title, Button, Group } from "@mantine/core";
import { useDisclosure } from "@mantine/hooks";
import { useOrders } from "./hooks/useOrders";
import { useOrderUIStore } from "./stores/useOrderUIStore";
import { OrderTable } from "./components/OrderTable";
import { OrderFilters } from "./components/OrderFilters";
import { OrderDetailModal } from "./components/OrderDetailModal";

export function OrderManagement() {
  const { data: orders, isLoading } = useOrders();
  const [isModalOpen, { open, close }] = useDisclosure(false);
  const selectOrder = useOrderUIStore((state) => state.selectOrder);

  if (isLoading) return <OrderManagementSkeleton />;

  return (
    <Stack gap="md">
      <Group justify="space-between">
        <Title order={2}>Orders</Title>
        <Button onClick={open}>New Order</Button>
      </Group>

      <OrderFilters />
      <OrderTable orders={orders ?? []} onRowClick={selectOrder} />
      <OrderDetailModal opened={isModalOpen} onClose={close} />
    </Stack>
  );
}

function OrderManagementSkeleton() {
  // loading skeleton...
  return null;
}
```

### Page Consumption

```tsx
// app/(auth)/orders/page.tsx
import { OrderManagement } from "@/modules/order-management";

export default function OrdersPage() {
  return <OrderManagement />;
}
```

## Internal Imports Within a Module

Inside a module, use **relative imports** — not `@/modules/...`:

```tsx
// Good — relative within the module
import { OrderTable } from "./components/OrderTable";
import { useOrders } from "./hooks/useOrders";
import type { IOrder } from "./types/order";

// Bad — absolute path within own module
import { OrderTable } from "@/modules/order-management/components/OrderTable";
```

## Module Readme (`[module-name].md`)

Every module must have a `[module-name].md` file in its root. This is a short, human-readable summary so developers can quickly understand what the module does without reading the code.

Keep it brief — a few lines, not a full doc. Include: purpose, key components, and main actions.

```md
<!-- modules/order-management/order-management.md -->
# Order Management

Manages the full order lifecycle: listing, filtering, creation, and status updates.

## Key Components
- **OrderManagement** — Main entry. Renders table, filters, and create modal.
- **OrderTable** — Sortable table with status badges and row click to detail.
- **OrderFilters** — Search, status, and priority filter controls.
- **OrderDetailModal** — View/edit order details.
- **OrderForm** — Create/edit form with Zod validation.

## Actions
- `getOrders` — Paginated, filterable order list.
- `createOrder` — Validates input, inserts order, revalidates.
- `updateOrderStatus` — Status transition (e.g., pending → shipped).
- `deleteOrder` — Admin-only soft delete.
```

## Rules Summary

- **Self-contained** — a module can have anything global has (components, hooks, actions, stores, validations, types, services, constants). Only create what the feature needs.
- **Barrel file is the contract** — only export what the app needs, everything else is private.
- **Module readme** — every module has `[module-name].md` with a short summary of purpose, components, and actions.
- **Tests colocated** — `OrderTable.tsx` + `OrderTable.test.tsx` side by side.
- **Relative imports inside** the module, `@/modules/[name]` from outside.
- **Don't share internals** — if something needs to be shared across modules, move it to the global directory (`hooks/`, `types/`, `actions/`, `stores/`, etc.).
- **One module = one concern** — don't create "god modules" that own too many domains.
- **Module folder is kebab-case** — `order-management/`, not `OrderManagement/` or `orderManagement/`.
- **Same rules apply** — all global conventions (Zustand selectors, TanStack Query patterns, Zod validation, a11y, etc.) apply identically inside modules.
