---
name: react-frontend
description: Frontend development with React 18+, TypeScript, Tailwind CSS, and shadcn/ui. Use when building UI components, routes, forms, client state, hooks, or any frontend code. Covers component patterns, performance optimization, accessibility, and clean architecture principles.
license: LICENSE.txt
---

# React Frontend Development

## Tech Stack

**Core:** React 18+, TypeScript (strict mode)
**Styling:** Tailwind CSS, shadcn/ui components, `cn()` helper
**State:** TanStack Query (server state), Zustand (client state), useReducer (complex local)
**Forms:** Conform + Zod or React Hook Form  

## File Organization

```
app/
├── routes/                 # File-based routes (if using React Router 7 / Remix)
├── components/
│   ├── ui/                # shadcn/ui primitives
│   └── forms/             # Form-specific components
├── utils/
│   ├── hooks/             # Custom React hooks
│   ├── validation/        # Zod schemas
│   └── constants/         # App constants
└── lib/                   # Shared utilities (cn, formatters)
```

## Core Principles

### Component Rules
- Functional components with hooks only
- Props typed with `type` (not interface for simple objects)
- Use `readonly` for immutable props
- Keep components small, focused, composable
- No `any` types without justification

### Data Flow

**Principle:** Never fetch data in `useEffect`. Use your stack's data fetching solution.

**Why avoid `useEffect` for fetching:**
- Creates request waterfalls (fetch after render)
- No built-in caching or deduplication
- Race conditions require manual cleanup
- No loading/error states out of the box
- Can't run on the server

**What to use instead:**

| Stack | Solution |
|-------|----------|
| React Router 7 / Remix | `loader` exports (fetch before render) |
| Next.js App Router | Server Components with `async/await` |
| Next.js Pages Router | `getServerSideProps` / `getStaticProps` |
| Plain React SPA | TanStack Query or SWR (never raw useEffect) |

```typescript
// ❌ AVOID: useEffect fetching
useEffect(() => {
  fetch('/api/user').then(r => r.json()).then(setUser);
}, []);

// ✅ PREFER: TanStack Query (for SPAs without framework loaders)
const { data: user } = useQuery({
  queryKey: ['user'],
  queryFn: () => fetch('/api/user').then(r => r.json())
});

// ✅ PREFER: React Router loader
export async function loader() {
  return json({ user: await getUser() });
}
```

**Last resort:** If `useEffect` is unavoidable, handle loading states, errors, and stale responses properly (abort controller, cleanup, race conditions).

**Follow existing patterns in the codebase.** Check how other pages/components fetch data and use the same approach.

### Performance Defaults
- `React.memo` for expensive components
- `useCallback` for event handlers passed to children
- `useMemo` for expensive calculations
- Avoid inline object creation in render

### Security

**XSS Prevention:**
- React escapes content by default - trust it
- **Never** use `dangerouslySetInnerHTML` unless absolutely necessary
- If you must render HTML, sanitize with DOMPurify first
- Avoid `eval()`, `new Function()`, and dynamic script injection

```typescript
// ❌ DANGEROUS
<div dangerouslySetInnerHTML={{ __html: userContent }} />

// ✅ SAFE: If HTML rendering is required
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userContent) }} />
```

**CSRF Protection:**
- Use `SameSite=Strict` or `SameSite=Lax` cookies for auth
- Include CSRF tokens in forms for state-changing requests
- Verify `Origin`/`Referer` headers on the server

```typescript
// Include CSRF token in forms
<input type="hidden" name="csrf_token" value={csrfToken} />

// Or in headers for fetch requests
fetch('/api/action', {
  method: 'POST',
  headers: { 'X-CSRF-Token': csrfToken },
});
```

**Input Validation:**
- Client-side validation is for **UX only** (immediate feedback)
- **Server-side validation is security** - never trust client input
- Use Zod schemas on both client and server

**Sensitive Data:**
- **Never** store secrets (API keys, tokens) in frontend code
- Environment variables prefixed with `NEXT_PUBLIC_` or `VITE_` are exposed to the client
- Prefer `httpOnly` cookies over `localStorage` for auth tokens (prevents XSS token theft)
- Don't log sensitive data to console in production

## Essential Patterns

### Component Structure
```typescript
type UserCardProps = {
  readonly user: { readonly id: string; readonly name: string };
  readonly onEdit?: (id: string) => void;
  readonly className?: string;
};

export function UserCard({ user, onEdit, className }: UserCardProps) {
  const handleEdit = useCallback(() => onEdit?.(user.id), [onEdit, user.id]);
  
  return (
    <Card className={cn('p-4', className)}>
      <h3 className="font-semibold">{user.name}</h3>
      {onEdit && <Button onClick={handleEdit} size="sm">Edit</Button>}
    </Card>
  );
}
```

### Route Module Structure (React Router 7 / Remix only)

> **Only apply if project uses React Router 7 with loaders/actions.** Check for existing `loader`/`action` exports in route files.

```typescript
import type { Route } from './+types/item';

export async function loader({ params }: Route.LoaderArgs) {
  // Data fetching here
  return json({ item });
}

export async function action({ request }: Route.ActionArgs) {
  const formData = await request.formData();
  // Mutations here
  return json({ success: true });
}

export default function ItemPage({ loaderData }: Route.ComponentProps) {
  return <div>{loaderData.item.name}</div>;
}

export function ErrorBoundary({ error }: Route.ErrorBoundaryProps) {
  return <ErrorDisplay error={error} />;
}
```

### Error Handling with Result Pattern
```typescript
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };
```

## References

- **Component patterns, hooks, state, performance**: See [resources/react-patterns.md](resources/react-patterns.md)
- **React Router 7 routes, loaders, actions, layouts** (if using React Router): See [resources/react-router.md](resources/react-router.md)
- **TypeScript conventions and style**: See [resources/typescript.md](resources/typescript.md)

## Code Conventions

### Naming
- `PascalCase`: Components, types, interfaces, enums
- `camelCase`: Variables, functions, methods
- `SCREAMING_SNAKE_CASE`: Constants
- `kebab-case`: File names

### Imports Order
```typescript
// 1. External packages
import { useState } from 'react';
// 2. Internal aliases
import { Button } from '~/components/ui/button';
// 3. Relative imports
import { formatDate } from './utils';
```

### Exports
- Prefer named exports over default (except page/route components)
- Export types separately: `export type { Props }`

## Quality Checklist

- [ ] TypeScript strict mode, no `any`
- [ ] Props use `readonly` where appropriate
- [ ] Error boundaries at appropriate levels
- [ ] Loading states handled
- [ ] Accessibility: ARIA labels, semantic HTML
- [ ] Performance: memo/useCallback where needed
- [ ] Security: no `dangerouslySetInnerHTML`, no secrets in client code
- [ ] Small, focused functions (SRP)
