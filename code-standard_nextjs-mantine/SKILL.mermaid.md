# Project Folder Architecture

> Layered folder dependency diagram. See [SKILL.md](SKILL.md) for the full coding standard.

```mermaid
graph BT
    %% ── Data Layer (bottom) ──
    DB[("PostgreSQL<br/><small>source of truth</small>")]
    ORM["db/<br/><small>Drizzle ORM · schema/ · index.ts</small>"]
    DB --> ORM

    %% ── Business Logic Layer ──
    Services["services/<br/><small>business logic · reusable queries</small>"]
    ORM --> Services

    %% ── Server Layer ──
    Actions["actions/<br/><small>Server Actions · thin · Zod validate</small>"]
    API["app/api/<br/><small>webhooks only</small>"]
    ORM --> Actions
    Services --> Actions
    Services --> API

    %% ── Page Layer ──
    App["app/<br/><small>pages · layouts · Server Components</small>"]
    Actions --> App
    API --> App

    %% ── UI Layer (top) ──
    Components["components/<br/><small>ui/ · forms/ · layouts/</small>"]
    Modules["modules/<br/><small>self-contained features<br/>own actions/ · hooks/ · stores/ · types/</small>"]
    App --> Components
    App --> Modules

    %% ── Support (side — used across layers) ──
    Hooks["hooks/<br/><small>TanStack Query wrappers<br/>shared custom hooks</small>"]
    Stores["stores/<br/><small>Zustand · global client state</small>"]
    Types["types/<br/><small>shared interfaces · db $infer · z.infer</small>"]
    Validations["lib/validations/<br/><small>Zod schemas · shared client+server</small>"]
    Lib["lib/<br/><small>utils · constants · env · auth helpers</small>"]
    Theme["theme/<br/><small>Mantine createTheme() · component overrides</small>"]

    Hooks -.-> Components
    Hooks -.-> Modules
    Stores -.-> Components
    Stores -.-> Modules
    Types -.-> Actions
    Types -.-> Services
    Types -.-> Components
    Types -.-> Modules
    Validations -.-> Actions
    Validations -.-> Components
    Validations -.-> Modules
    Lib -.-> Actions
    Lib -.-> Services
    Lib -.-> Components
    Theme -.-> Components
    Theme -.-> Modules

    %% ── Auth (spans all server layers) ──
    Auth["middleware.ts + lib/auth.ts<br/><small>Clerk · route protection · RBAC</small>"]
    Auth -.-> App
    Auth -.-> Actions
    Auth -.-> API

    %% ── Styling ──
    classDef data fill:#4a6741,stroke:#333,color:#fff
    classDef server fill:#5b6abf,stroke:#333,color:#fff
    classDef page fill:#7c5cbf,stroke:#333,color:#fff
    classDef ui fill:#bf5b5b,stroke:#333,color:#fff
    classDef support fill:#666,stroke:#333,color:#fff
    classDef auth fill:#bf8c2e,stroke:#333,color:#fff

    class DB,ORM data
    class Services,Actions,API server
    class App page
    class Components,Modules ui
    class Hooks,Stores,Types,Validations,Lib,Theme support
    class Auth auth
```

## Legend

| Color | Layer | Can import from |
|-------|-------|-----------------|
| Green | **Data** — `db/` | Nothing (lowest) |
| Blue | **Server** — `services/`, `actions/`, `app/api/` | Data, Support |
| Purple | **Pages** — `app/` | Server, Support |
| Red | **UI** — `components/`, `modules/` | Pages (via props), Support |
| Gray | **Support** — `hooks/`, `stores/`, `types/`, `lib/`, `theme/` | Used across all layers |
| Gold | **Auth** — `middleware.ts`, `lib/auth.ts` | Spans all server layers |

## Key Rules

- **Arrows point UP** — lower layers never import from upper layers.
- `db/` is **server-only** — never imported in client components.
- `modules/` are self-contained — they have their own actions/, hooks/, stores/, types/ inside.
- `stores/` and `hooks/` support **component-level and above** — not used in services/ or db/.
- `types/` and `lib/validations/` are **shared everywhere** — client and server.
