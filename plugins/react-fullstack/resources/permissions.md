# Permission System

Hierarchical access control with three scope levels: **Owner → Event → Entity**.

## Permission Hierarchy

### Owner-level access (`scopeType: 'owner'`)
- Can access ALL events under that owner
- Can access ALL entities in those events
- Highest privilege level

### Event-level access (`scopeType: 'event'`)
- Can access specific event
- Can access ALL entities in that event
- Mid-level privilege

### Entity-level access (`scopeType: 'entity'`)
- Can access ONLY specific entity
- Lowest privilege level

## Role-Based Permissions

- **`read`** - View-only access
- **`write`** - Full CRUD access + can invite others

## Superadmin Override

Users with `isSuperadmin: true` bypass ALL permission checks. They have implicit `write` access to everything. Used for system administrators.

## Permission Utilities

Location: `.server/utils/permissions.ts`

### Access Checks

```typescript
// Check if user has access to resource with required role
await hasAccess(userId, resource, requiredRole);
// resource: { ownerId, eventId?, entityId? }
// requiredRole: 'read' | 'write' (default: 'read')

// Get highest role user has for resource
await getUserRole(userId, resource); // Returns 'read' | 'write' | null

// Check if user can invite others (requires 'write' role)
await canInviteUsers(userId, resource);
```

### Membership Queries

```typescript
// Get all memberships for a user
await getUserMemberships(userId);

// Get owners user can access
await getAccessibleOwners(userId);

// Get events user can access (optionally filtered by owner)
await getAccessibleEvents(userId, ownerId?);

// Get entities user can access within an event
await getAccessibleEntities(userId, eventId);
```

## Permission Check Pattern

```typescript
// In tRPC procedures or loaders
const user = await getAuthSession(request);
if (!user) throw new TRPCError({ code: "UNAUTHORIZED" });

// Check access before any operation
const canAccess = await hasAccess(user.userId, {
  ownerId: input.ownerId,
  eventId: input.eventId,
  entityId: input.entityId
}, 'write'); // or 'read'

if (!canAccess) {
  throw new TRPCError({ code: "FORBIDDEN" });
}
```

## Key Rules

- Memberships are unique per `(user, scope)`
- Always consider scope + role when adding or modifying queries
- If a change affects memberships, roles, or cross-scope access, call it out explicitly before implementing
