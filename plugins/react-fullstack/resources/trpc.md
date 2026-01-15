# tRPC API Structure

## Available Routers

Location: `.server/api/routers/`

The `appRouter` in `.server/api/root.ts` exposes:

| Router | Purpose | Access Level |
|--------|---------|--------------|
| `user` | User session and profile management | Protected |
| `owners` | Owner CRUD operations | Superadmin only |
| `events` | Event management | Owner/event level |
| `entities` | Entity management | Owner/event/entity level |
| `venues` | Venue management | Protected |
| `zones` | Zone management (within events) | Event level |
| `ticketTypes` | Ticket type definitions | Event level |
| `tickets` | Ticket CRUD and bulk import | Entity level |

## Procedure Types

- **`publicProcedure`** - No authentication required (rarely used)
- **`protectedProcedure`** - Requires authenticated user (most common)

All procedures should validate permissions using the permission utilities.

## Server-Side Caller

Use `trpcServer(request)` in loaders/actions for type-safe server-side calls:

```typescript
import { trpcServer } from '~/.server/utils/trpc-server';

export async function loader({ request }: Route.LoaderArgs) {
  const trpc = trpcServer(request);
  
  // Call tRPC procedures directly
  const events = await trpc.events.getAll();
  const owner = await trpc.owners.getById({ id: ownerId });
  
  return json({ events, owner });
}
```

**Benefits:**
- No HTTP overhead
- Type-safe calls
- Automatic authentication context
- Avoids code duplication

## Creating New Procedures

```typescript
// In .server/api/routers/example.ts
import { z } from 'zod';
import { router, protectedProcedure } from '../trpc';
import { hasAccess } from '../../utils/permissions';
import { TRPCError } from '@trpc/server';

export const exampleRouter = router({
  getById: protectedProcedure
    .input(z.object({ id: z.string().uuid() }))
    .query(async ({ ctx, input }) => {
      // Permission check
      const canAccess = await hasAccess(ctx.user.userId, {
        ownerId: input.ownerId,
      }, 'read');
      
      if (!canAccess) {
        throw new TRPCError({ code: 'FORBIDDEN' });
      }
      
      // Query logic...
    }),
    
  create: protectedProcedure
    .input(z.object({
      name: z.string().min(1),
      ownerId: z.string().uuid(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Permission check for write
      const canWrite = await hasAccess(ctx.user.userId, {
        ownerId: input.ownerId,
      }, 'write');
      
      if (!canWrite) {
        throw new TRPCError({ code: 'FORBIDDEN' });
      }
      
      // Mutation logic...
    }),
});
```

## Adding Router to App

```typescript
// In .server/api/root.ts
import { exampleRouter } from './routers/example';

export const appRouter = router({
  // ... existing routers
  example: exampleRouter,
});
```

## Client Usage

The tRPC client is configured with React Query. In components:

```typescript
// Queries
const { data, isLoading } = trpc.events.getAll.useQuery();

// Mutations
const mutation = trpc.events.create.useMutation({
  onSuccess: () => {
    // Invalidate queries, redirect, etc.
  },
});
```

But remember: prefer loaders/actions over direct client calls for data fetching and mutations.
