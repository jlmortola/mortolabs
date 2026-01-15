# Plugins

## Installation

```bash
/plugin install <plugin-name>@mortolabs
```

---

## react-frontend

**Type:** Skill (auto-triggered)

**Triggers on:** React components, hooks, state management, forms, UI development

### What It Teaches Claude

- React 18+ functional component patterns
- TypeScript strict mode with proper prop typing
- Tailwind CSS + shadcn/ui component styling
- TanStack Query for server state
- Zustand for client state
- React Router 7 loaders/actions (if applicable)
- Performance optimization (memo, useCallback, useMemo)
- Accessibility patterns

### Key Rules

1. **Never fetch in useEffect** - Use loaders or TanStack Query
2. **No `any` types** - Strict TypeScript
3. **Props use `readonly`** - Immutable by default
4. **Small, focused components** - Single responsibility

### Resources Included

| File | Contents |
|------|----------|
| `react-patterns.md` | Component patterns, hooks, state, performance |
| `react-router.md` | File-based routing, loaders, actions, layouts |
| `typescript.md` | Type conventions, generics, Zod integration |

---

## react-fullstack

**Type:** Skill (auto-triggered)

**Triggers on:** API development, tRPC, database, authentication, permissions

### What It Teaches Claude

- tRPC procedure creation with Zod validation
- Drizzle ORM queries and mutations
- Supabase Auth with server-side sessions
- Permission system (Owner → Event → Entity hierarchy)
- Security best practices

### Key Rules

1. **Always check permissions** - Use `hasAccess()` before operations
2. **Always include headers** - For token refresh in responses
3. **Use `trpcServer(request)`** - For server-side tRPC calls
4. **Validate all inputs** - Zod schemas on every procedure

### Permission Model

```
Owner Level
    │
    ├── Event Level
    │       │
    │       └── Entity Level
```

Roles: `read` (view-only) or `write` (full CRUD)

### Resources Included

| File | Contents |
|------|----------|
| `trpc.md` | Router structure, procedure patterns |
| `auth.md` | Session management, auth helpers |
| `permissions.md` | Access control functions |
| `schema.md` | Database schema reference |

---

## create-react-project

**Type:** Command (user-invoked)

**Usage:**
```bash
/create-react-project:init [project-name]
```

### What It Does

Interactive project scaffolding wizard that asks about:

1. **Project Type**
   - Frontend Only (SPA)
   - Fullstack with tRPC

2. **Package Manager**
   - pnpm, npm, bun, yarn

3. **Framework**
   - React Router 7
   - Next.js App Router
   - Vite + TanStack Router
   - Vite + React Router

4. **UI Library**
   - shadcn/ui
   - Radix UI
   - None

5. **Additional Options**
   - State management
   - Form library
   - Database (for fullstack)
   - Auth provider (for fullstack)
   - Testing setup

### Output

Creates a fully configured project with:
- Selected framework scaffolded
- Dependencies installed
- Folder structure created
- Starter files for fullstack (if selected)
- Git initialized

### Example

```bash
/create-react-project:init my-saas-app

# Prompts for configuration...
# Creates ./my-saas-app/ with full setup
```

---

## Using Multiple Plugins Together

The skills complement each other:

1. **Start with `create-react-project:init`** - Scaffold a new project
2. **`react-frontend` activates** - When building UI components
3. **`react-fullstack` activates** - When building API/backend (if fullstack)

Example workflow:
```
/create-react-project:init my-app
  → Select "Fullstack with tRPC"
  → Project created

"Create a user profile page"
  → react-frontend skill activates
  → Builds component with proper patterns

"Add a tRPC procedure to update user settings"
  → react-fullstack skill activates
  → Creates procedure with permissions, validation
```
