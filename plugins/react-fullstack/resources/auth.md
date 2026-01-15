# Authentication System

Uses **Supabase Auth** with server-side session management.

## Key Components

- **Auth Provider:** Supabase (`@supabase/ssr`, `@supabase/supabase-js`)
- **Session Storage:** Server-side cookies (encrypted, httpOnly)
- **Token Refresh:** Automatic refresh 5 minutes before expiry
- **Auth Utilities:** `.server/utils/auth.ts`

## Session Structure

```typescript
interface AuthSession {
  userId: string;              // Supabase user ID
  email: string;               // User email
  role: 'superadmin' | 'owner' | 'entity';
  supabaseToken: string;       // Access token
  supabaseRefreshToken: string; // Refresh token
  expiresAt: number;           // Unix timestamp
  userName?: string;           // Display name
  ownerId?: string;            // For owner role
  zoneIds?: string[];          // For zone access
}
```

## Auth Helper Functions

```typescript
// Get current session (returns null if not authenticated)
const session = await getAuthSession(request);

// Require authentication (throws redirect to login if not authenticated)
const { user, headers } = await requireAuth(request);

// Require specific role (throws redirect if unauthorized)
const { user, headers } = await requireRole(request, 'owner');

// Require superadmin access
const { user, headers } = await requireSuperAdmin(request);

// Optional auth (for pages that work both ways)
const { user, headers } = await optionalAuth(request);

// Setup session after login
const { user, headers } = await setupUserSession(
  request, userId, email, supabaseToken, supabaseRefreshToken, expiresAt
);

// Logout and destroy session
await logout(request, redirectTo);
```

## Auth Flow

1. User signs in via Supabase Auth
2. `setupUserSession()` creates server-side session cookie
3. Session includes user role determined by memberships
4. Loaders/actions use `requireAuth()` or `requireSuperAdmin()`
5. Auto-refresh tokens before expiry
6. Invalid sessions redirect to login

## Usage in Loaders/Actions

```typescript
export async function loader({ request }: Route.LoaderArgs) {
  const { user, headers } = await requireAuth(request);
  
  // user is now available
  const trpc = trpcServer(request);
  const data = await trpc.events.getAll();
  
  return json({ data }, { headers });
}
```

Always pass `headers` in response to handle token refresh.
