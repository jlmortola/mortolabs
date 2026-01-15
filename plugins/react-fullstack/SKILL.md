---
name: fullstack-trpc
description: Fullstack development with tRPC, React Router 7, Drizzle ORM, and Supabase Auth. Use when building APIs, database operations, authentication flows, or full-stack features. Covers type-safe APIs, permission systems, data modeling, and security best practices.
license: LICENSE.txt
---

# Fullstack tRPC Development

## Tech Stack

**API:** tRPC with React Query client
**Database:** Drizzle ORM with PostgreSQL
**Auth:** Supabase Auth with server-side sessions
**Routing:** React Router 7 (loaders/actions pattern)
**Validation:** Zod schemas (shared client/server)

## Core Principles

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

**This project uses React Router 7 loaders/actions:**

```typescript
// In loaders - use trpcServer for server-side calls
import { trpcServer } from '~/.server/utils/trpc-server';

export async function loader({ request }: Route.LoaderArgs) {
  const { user, headers } = await requireAuth(request);
  const trpc = trpcServer(request);

  const events = await trpc.events.getAll();
  return json({ events }, { headers });
}

// In actions - handle mutations
export async function action({ request }: Route.ActionArgs) {
  const { user, headers } = await requireAuth(request);
  const formData = await request.formData();
  // ... mutation logic
  return json({ success: true }, { headers });
}
```

**Last resort:** If `useEffect` is unavoidable, handle loading states, errors, and stale responses properly (abort controller, cleanup, race conditions).

**Follow existing patterns in the codebase.** Check how other pages/components fetch data and use the same approach.

### tRPC Usage

**Server-side (loaders/actions):** Use `trpcServer(request)` - no HTTP overhead, type-safe.

**Client-side (components):** Use tRPC React Query hooks, but prefer loaders for initial data.

```typescript
// ❌ AVOID: Client-side fetching for initial page data
const { data } = trpc.events.getAll.useQuery();

// ✅ PREFER: Server-side loader
export async function loader({ request }: Route.LoaderArgs) {
  const trpc = trpcServer(request);
  return json({ events: await trpc.events.getAll() });
}

// ✅ OK: Client-side for mutations and subsequent fetches
const mutation = trpc.events.create.useMutation();
```

### Security

**Authentication:**
- Always use `requireAuth(request)` or `requireRole(request, role)` in loaders/actions
- Always pass `headers` in response (handles token refresh)
- Use `httpOnly` cookies for session storage (prevents XSS token theft)

```typescript
export async function loader({ request }: Route.LoaderArgs) {
  const { user, headers } = await requireAuth(request);
  // ... your logic
  return json({ data }, { headers }); // Always include headers!
}
```

**Authorization:**
- Always check permissions before operations using `hasAccess()`
- Never trust client-side permission checks alone
- Check at the procedure level, not just route level

```typescript
// In tRPC procedures
const canAccess = await hasAccess(ctx.user.userId, {
  ownerId: input.ownerId,
  eventId: input.eventId,
}, 'write');

if (!canAccess) {
  throw new TRPCError({ code: 'FORBIDDEN' });
}
```

**Input Validation:**
- Client-side validation is for **UX only** (immediate feedback)
- **Server-side validation is security** - never trust client input
- Use Zod schemas on both client and server
- All tRPC inputs must have Zod validation

```typescript
// tRPC procedure with input validation
create: protectedProcedure
  .input(z.object({
    name: z.string().min(1).max(255),
    email: z.string().email(),
    ownerId: z.string().uuid(),
  }))
  .mutation(async ({ ctx, input }) => {
    // input is already validated and typed
  }),
```

**XSS Prevention:**
- React escapes content by default - trust it
- **Never** use `dangerouslySetInnerHTML` unless absolutely necessary
- If you must render HTML, sanitize with DOMPurify first

**CSRF Protection:**
- tRPC mutations use POST requests with JSON bodies
- Supabase session cookies use `SameSite=Lax`
- For form submissions, verify origin headers server-side

**Sensitive Data:**
- **Never** expose secrets in client bundles
- Database credentials, API keys stay in `.server/` only
- Environment variables without `VITE_` prefix are server-only
- Never log sensitive data (passwords, tokens, PII)

### Permission System

This project uses hierarchical permissions: **Owner → Event → Entity**

| Scope | Access Level |
|-------|--------------|
| Owner | All events and entities under that owner |
| Event | Specific event and all its entities |
| Entity | Only that specific entity |

**Roles:** `read` (view-only) or `write` (full CRUD + invite)

**Superadmin:** Users with `isSuperadmin: true` bypass all checks.

```typescript
// Check access
await hasAccess(userId, { ownerId, eventId, entityId }, 'read');

// Get user's highest role
await getUserRole(userId, { ownerId, eventId });

// Get accessible resources
await getAccessibleOwners(userId);
await getAccessibleEvents(userId, ownerId);
await getAccessibleEntities(userId, eventId);
```

## API Patterns

### Creating tRPC Procedures

```typescript
import { z } from 'zod';
import { router, protectedProcedure } from '../trpc';
import { hasAccess } from '../../utils/permissions';
import { TRPCError } from '@trpc/server';

export const exampleRouter = router({
  getById: protectedProcedure
    .input(z.object({ id: z.string().uuid() }))
    .query(async ({ ctx, input }) => {
      // 1. Permission check
      const canAccess = await hasAccess(ctx.user.userId, {
        ownerId: input.ownerId,
      }, 'read');

      if (!canAccess) {
        throw new TRPCError({ code: 'FORBIDDEN' });
      }

      // 2. Database query
      const result = await db.query.example.findFirst({
        where: eq(example.id, input.id),
      });

      if (!result) {
        throw new TRPCError({ code: 'NOT_FOUND' });
      }

      return result;
    }),

  create: protectedProcedure
    .input(z.object({
      name: z.string().min(1),
      ownerId: z.string().uuid(),
    }))
    .mutation(async ({ ctx, input }) => {
      // 1. Permission check (write access)
      const canWrite = await hasAccess(ctx.user.userId, {
        ownerId: input.ownerId,
      }, 'write');

      if (!canWrite) {
        throw new TRPCError({ code: 'FORBIDDEN' });
      }

      // 2. Create record
      const [created] = await db.insert(example).values({
        ...input,
        createdBy: ctx.user.userId,
      }).returning();

      return created;
    }),
});
```

### Database Operations

Use Drizzle ORM for type-safe database access:

```typescript
import { db } from '~/.server/db';
import { events, tickets } from '~/.server/db/schema';
import { eq, and } from 'drizzle-orm';

// Select with relations
const event = await db.query.events.findFirst({
  where: eq(events.id, eventId),
  with: {
    tickets: true,
    zones: true,
  },
});

// Insert
const [newEvent] = await db.insert(events).values({
  name: 'New Event',
  ownerId,
}).returning();

// Update
await db.update(events)
  .set({ status: 'published' })
  .where(eq(events.id, eventId));

// Delete (cascades per schema)
await db.delete(events).where(eq(events.id, eventId));
```

## References

- **tRPC routers and procedures**: See [resources/trpc.md](resources/trpc.md)
- **Authentication system**: See [resources/auth.md](resources/auth.md)
- **Database schema**: See [resources/schema.md](resources/schema.md)
- **Permission system**: See [resources/permissions.md](resources/permissions.md)

## Quality Checklist

- [ ] All tRPC inputs validated with Zod
- [ ] Permission checks before every operation
- [ ] `requireAuth()` in all protected loaders/actions
- [ ] Headers passed in all responses (token refresh)
- [ ] No secrets in client-accessible code
- [ ] Database operations use transactions where needed
- [ ] Error handling with proper tRPC error codes
- [ ] No `dangerouslySetInnerHTML`, no client-side secrets
