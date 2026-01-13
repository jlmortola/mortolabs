# React Router 7 Patterns

## File-Based Routing

```
app/routes/
├── _index.tsx              # / (home)
├── about.tsx               # /about
├── _auth/                  # Layout group (no URL segment)
│   ├── login.tsx          # /login
│   └── signup.tsx         # /signup
├── dashboard/
│   ├── _layout.tsx        # Layout for /dashboard/*
│   ├── _index.tsx         # /dashboard
│   └── settings.tsx       # /dashboard/settings
├── items/
│   ├── _index.tsx         # /items
│   ├── $id.tsx            # /items/:id (dynamic)
│   └── $id_.edit.tsx      # /items/:id/edit
└── api/
    └── webhook.ts         # /api/webhook (resource route)
```

### Naming Conventions
- `_index.tsx` - Index route for directory
- `_layout.tsx` - Layout wrapper for nested routes
- `$param.tsx` - Dynamic segment
- `$param_.child.tsx` - Pathless layout escape
- `_prefix/` - Route group (no URL segment)
- `.server.ts` - Server-only code

## Route Module Exports

### Complete Route Module
```typescript
import type { Route } from './+types/item';
import { json, redirect } from 'react-router';

// Data loading (runs on server)
export async function loader({ request, params }: Route.LoaderArgs) {
  const user = await getUser(request);
  if (!user) throw redirect('/login');
  
  const item = await db.items.findById(params.id);
  if (!item) throw new Response('Not found', { status: 404 });
  
  return json({ item, user });
}

// Mutations (runs on server)
export async function action({ request, params }: Route.ActionArgs) {
  const formData = await request.formData();
  const intent = formData.get('intent');
  
  switch (intent) {
    case 'update':
      await db.items.update(params.id, Object.fromEntries(formData));
      return json({ success: true });
    case 'delete':
      await db.items.delete(params.id);
      return redirect('/items');
    default:
      return json({ error: 'Unknown intent' }, { status: 400 });
  }
}

// Component
export default function ItemPage({ loaderData, actionData }: Route.ComponentProps) {
  const { item } = loaderData;
  return (
    <div>
      <h1>{item.name}</h1>
      {actionData?.error && <Alert variant="error">{actionData.error}</Alert>}
    </div>
  );
}

// SEO
export function meta({ data }: Route.MetaArgs) {
  return [
    { title: data?.item?.name ?? 'Item' },
    { name: 'description', content: data?.item?.description },
  ];
}

// Error handling
export function ErrorBoundary({ error }: Route.ErrorBoundaryProps) {
  if (error.status === 404) {
    return <NotFound message="Item not found" />;
  }
  return <ErrorPage error={error} />;
}

// Resource preloading
export function links(): LinkDescriptor[] {
  return [{ rel: 'preload', href: '/api/items', as: 'fetch' }];
}
```

## Loader Patterns

### Authentication Check
```typescript
export async function loader({ request }: Route.LoaderArgs) {
  const user = await requireAuth(request);
  return json({ user });
}
```

### Query Parameters
```typescript
export async function loader({ request }: Route.LoaderArgs) {
  const url = new URL(request.url);
  const page = Number(url.searchParams.get('page')) || 1;
  const search = url.searchParams.get('q') || '';
  
  const items = await db.items.findMany({ page, search });
  return json({ items, page, search });
}
```

### Parallel Data Loading
```typescript
export async function loader({ params }: Route.LoaderArgs) {
  const [item, comments, related] = await Promise.all([
    db.items.findById(params.id),
    db.comments.findByItemId(params.id),
    db.items.findRelated(params.id),
  ]);
  
  return json({ item, comments, related });
}
```

### Conditional Loading
```typescript
export async function loader({ request, params }: Route.LoaderArgs) {
  const user = await getUser(request);
  
  const item = await db.items.findById(params.id);
  if (!item) throw new Response('Not found', { status: 404 });
  
  // Only load edit data if user can edit
  const canEdit = user && (user.id === item.ownerId || user.role === 'admin');
  
  return json({ 
    item, 
    canEdit,
    editHistory: canEdit ? await db.history.findByItemId(params.id) : null,
  });
}
```

## Action Patterns

### Form Validation
```typescript
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(1, 'Name required'),
  email: z.string().email('Invalid email'),
  description: z.string().optional(),
});

export async function action({ request }: Route.ActionArgs) {
  const formData = await request.formData();
  const result = schema.safeParse(Object.fromEntries(formData));
  
  if (!result.success) {
    return json({ 
      errors: result.error.flatten().fieldErrors 
    }, { status: 400 });
  }
  
  await db.users.create(result.data);
  return redirect('/users');
}
```

### Multiple Intents
```typescript
export async function action({ request, params }: Route.ActionArgs) {
  const formData = await request.formData();
  const intent = formData.get('intent');
  
  switch (intent) {
    case 'update':
      return handleUpdate(params.id, formData);
    case 'delete':
      return handleDelete(params.id);
    case 'duplicate':
      return handleDuplicate(params.id);
    default:
      return json({ error: 'Invalid intent' }, { status: 400 });
  }
}
```

### Optimistic UI Support
```typescript
export async function action({ request }: Route.ActionArgs) {
  const formData = await request.formData();
  const id = crypto.randomUUID(); // Generate ID server-side
  
  await db.items.create({ id, ...Object.fromEntries(formData) });
  
  return json({ id, success: true });
}
```

## Layout Routes

### Authenticated Layout
```typescript
// routes/dashboard/_layout.tsx
export async function loader({ request }: Route.LoaderArgs) {
  const user = await requireAuth(request);
  const notifications = await db.notifications.findUnread(user.id);
  return json({ user, notifications });
}

export default function DashboardLayout({ loaderData }: Route.ComponentProps) {
  return (
    <div className="min-h-screen">
      <Header user={loaderData.user} notifications={loaderData.notifications} />
      <div className="flex">
        <Sidebar />
        <main className="flex-1 p-6">
          <Outlet />
        </main>
      </div>
    </div>
  );
}
```

### Nested Layouts
```typescript
// routes/dashboard/settings/_layout.tsx
export default function SettingsLayout() {
  return (
    <div className="grid grid-cols-[200px_1fr] gap-6">
      <nav>
        <NavLink to="/dashboard/settings/profile">Profile</NavLink>
        <NavLink to="/dashboard/settings/security">Security</NavLink>
        <NavLink to="/dashboard/settings/billing">Billing</NavLink>
      </nav>
      <div>
        <Outlet />
      </div>
    </div>
  );
}
```

## Navigation

### NavLink with Active State
```typescript
<NavLink 
  to="/dashboard"
  className={({ isActive, isPending }) => 
    cn('nav-link', isActive && 'active', isPending && 'pending')
  }
>
  Dashboard
</NavLink>
```

### Programmatic Navigation
```typescript
import { useNavigate, useLocation } from 'react-router';

function LoginForm() {
  const navigate = useNavigate();
  const location = useLocation();
  
  const onSuccess = () => {
    const returnTo = location.state?.from?.pathname || '/dashboard';
    navigate(returnTo, { replace: true });
  };
}
```

### Form Navigation
```typescript
import { Form, useNavigation } from 'react-router';

function ItemForm({ item }: { item?: Item }) {
  const navigation = useNavigation();
  const isSubmitting = navigation.state === 'submitting';
  
  return (
    <Form method="post">
      <Input name="name" defaultValue={item?.name} />
      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Saving...' : 'Save'}
      </Button>
    </Form>
  );
}
```

## Error Handling

### Route Error Boundary
```typescript
export function ErrorBoundary({ error }: Route.ErrorBoundaryProps) {
  // Check for specific status codes
  if (error.status === 404) {
    return (
      <div className="text-center py-16">
        <h1 className="text-4xl font-bold">404</h1>
        <p className="text-muted-foreground">Page not found</p>
        <Button asChild className="mt-4">
          <Link to="/">Go Home</Link>
        </Button>
      </div>
    );
  }
  
  if (error.status === 403) {
    return <AccessDenied />;
  }
  
  // Generic error
  return (
    <div className="text-center py-16">
      <h1 className="text-4xl font-bold">Error</h1>
      <p className="text-muted-foreground">Something went wrong</p>
      <pre className="mt-4 text-sm">{error.message}</pre>
    </div>
  );
}
```

### Throwing Responses
```typescript
// In loader or action
if (!item) {
  throw new Response('Item not found', { status: 404 });
}

if (!canAccess) {
  throw new Response('Forbidden', { status: 403 });
}

// Redirect
throw redirect('/login');
```

## Resource Routes (API)

### JSON API Endpoint
```typescript
// routes/api/items.ts
export async function loader({ request }: Route.LoaderArgs) {
  const url = new URL(request.url);
  const items = await db.items.findMany({
    limit: Number(url.searchParams.get('limit')) || 20,
  });
  return json({ items });
}

export async function action({ request }: Route.ActionArgs) {
  if (request.method !== 'POST') {
    return json({ error: 'Method not allowed' }, { status: 405 });
  }
  
  const body = await request.json();
  const item = await db.items.create(body);
  return json({ item }, { status: 201 });
}
```

### File Upload Endpoint
```typescript
// routes/api/upload.ts
export async function action({ request }: Route.ActionArgs) {
  const formData = await request.formData();
  const file = formData.get('file') as File;
  
  if (!file) {
    return json({ error: 'No file provided' }, { status: 400 });
  }
  
  const url = await uploadToStorage(file);
  return json({ url });
}
```
