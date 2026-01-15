---
description: Scaffold a new React project with interactive configuration
argument-hint: [project-name]
allowed-tools: Bash(npm:*), Bash(pnpm:*), Bash(bun:*), Bash(yarn:*), Bash(npx:*), Bash(mkdir:*), Bash(cd:*), Bash(git:*), AskUserQuestion, Read, Write, Edit
---

# Create React Project

You are scaffolding a new React project. This command guides users through creating a properly configured React application.

## Step 1: Get Project Name

If `$ARGUMENTS` is empty, ask the user for the project name.

## Step 2: Gather Requirements

Ask the user ALL of the following questions using AskUserQuestion. You can ask up to 4 questions at a time, so batch them efficiently.

### Batch 1: Core Setup

**Project Type**
- Frontend Only (Recommended for SPAs) - Client-side React application
- Fullstack with tRPC - Includes API, database, and authentication

**Package Manager**
- pnpm (Recommended) - Fast, disk-efficient
- npm - Standard, widely compatible
- bun - Ultra-fast, all-in-one
- yarn - Classic alternative

**Framework/Router**
- React Router 7 (Recommended) - File-based routing, SSR, loaders/actions
- Next.js App Router - Full-stack React framework
- Vite + TanStack Router - Type-safe SPA routing
- Vite + React Router - Lightweight SPA

**UI Library**
- shadcn/ui (Recommended) - Copy-paste components, Tailwind CSS
- None - Start minimal
- Radix UI only - Unstyled primitives

### Batch 2: Additional Options (based on Project Type)

#### If "Frontend Only" Selected:

**State Management**
- TanStack Query + Zustand (Recommended) - Server + client state
- TanStack Query only - Server state focused
- Jotai - Atomic state management
- None - Minimal setup

**Form Library**
- Conform (Recommended for React Router 7) - Progressive enhancement
- React Hook Form + Zod - Popular choice
- None - Handle forms manually

#### If "Fullstack with tRPC" Selected:

**Database**
- Supabase (Recommended) - Hosted PostgreSQL + Auth included
- Local PostgreSQL - Self-managed database
- PlanetScale - Serverless MySQL

**Auth Provider**
- Supabase Auth (Recommended) - Built-in with database
- Clerk - Full-featured auth platform
- Auth.js (NextAuth) - Best for Next.js

### Batch 3: Common Features (multi-select)

**Additional Features** (allow multiple selections)
- TypeScript strict mode
- Tailwind CSS
- Prettier + ESLint
- Vitest + Testing Library

---

## Step 3: Scaffold the Project

Based on the user's selections, run the appropriate scaffold command.

Replace `$PKG_MGR` with the selected package manager (pnpm, npm, bun, yarn).
Replace `$PROJECT_NAME` with the project name from arguments or user input.

### React Router 7

```bash
$PKG_MGR create remix@latest $PROJECT_NAME --template remix-run/react-router-templates/default
cd $PROJECT_NAME
```

### Next.js

```bash
$PKG_MGR create next-app@latest $PROJECT_NAME --typescript --tailwind --eslint --app --src-dir
cd $PROJECT_NAME
```

### Vite + React

```bash
$PKG_MGR create vite@latest $PROJECT_NAME --template react-ts
cd $PROJECT_NAME
$PKG_MGR install
```

---

## Step 4: Install Dependencies

After scaffolding completes, install additional dependencies based on selections.

### shadcn/ui (if selected)

```bash
$PKG_MGR dlx shadcn@latest init -y
$PKG_MGR dlx shadcn@latest add button card input form dialog
```

### TanStack Query

```bash
$PKG_MGR add @tanstack/react-query
```

### Zustand (if selected)

```bash
$PKG_MGR add zustand
```

### Form Libraries

For Conform:
```bash
$PKG_MGR add @conform-to/react @conform-to/zod zod
```

For React Hook Form:
```bash
$PKG_MGR add react-hook-form @hookform/resolvers zod
```

### Fullstack Dependencies (if Fullstack selected)

```bash
# tRPC
$PKG_MGR add @trpc/server @trpc/client @trpc/react-query

# Drizzle ORM
$PKG_MGR add drizzle-orm postgres
$PKG_MGR add -D drizzle-kit

# Supabase (if selected)
$PKG_MGR add @supabase/supabase-js @supabase/ssr

# Zod for validation
$PKG_MGR add zod
```

### Testing (if selected)

```bash
$PKG_MGR add -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

### Linting (if selected)

```bash
$PKG_MGR add -D prettier eslint eslint-config-prettier
```

---

## Step 5: Create Project Structure

Create the appropriate folder structure based on project type.

### Frontend Only Structure

```bash
mkdir -p app/components/ui
mkdir -p app/components/forms
mkdir -p app/utils/hooks
mkdir -p app/utils/validation
mkdir -p app/utils/constants
mkdir -p app/lib
```

### Fullstack Structure

```bash
mkdir -p app/.server/db
mkdir -p app/.server/trpc/routers
mkdir -p app/.server/utils
mkdir -p app/components/ui
mkdir -p app/components/forms
mkdir -p app/utils/hooks
mkdir -p app/utils/validation
mkdir -p app/lib
```

---

## Step 6: Generate Starter Files (Fullstack Only)

If fullstack was selected, create these starter files:

### app/.server/trpc/trpc.ts

```typescript
import { initTRPC, TRPCError } from '@trpc/server';
import type { Context } from './context';

const t = initTRPC.context<Context>().create();

export const router = t.router;
export const publicProcedure = t.procedure;

export const protectedProcedure = t.procedure.use(({ ctx, next }) => {
  if (!ctx.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }
  return next({ ctx: { ...ctx, user: ctx.user } });
});
```

### app/.server/trpc/context.ts

```typescript
import type { User } from '@supabase/supabase-js';

export type Context = {
  user: User | null;
  request: Request;
};

export function createContext(request: Request, user: User | null): Context {
  return { request, user };
}
```

### app/.server/db/index.ts

```typescript
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

const connectionString = process.env.DATABASE_URL!;
const client = postgres(connectionString);

export const db = drizzle(client, { schema });
```

### app/.server/db/schema.ts

```typescript
import { pgTable, uuid, text, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: text('email').notNull().unique(),
  name: text('name'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});
```

### drizzle.config.ts (at project root)

```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './app/.server/db/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

### .env.example

```
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
```

---

## Step 7: Initialize Git (if not already)

```bash
git init
git add .
git commit -m "Initial project setup with create-react-project"
```

---

## Step 8: Summary

After completing all steps, provide the user with a clear summary:

### What was created

1. **Project location:** `./$PROJECT_NAME/`
2. **Framework:** [selected framework]
3. **Project type:** [Frontend Only / Fullstack]

### Installed packages

List all packages that were installed.

### Next steps

```bash
cd $PROJECT_NAME
$PKG_MGR run dev
```

For fullstack projects, also mention:
1. Copy `.env.example` to `.env` and fill in credentials
2. Run `$PKG_MGR drizzle-kit push` to sync database schema
3. Set up Supabase project at https://supabase.com (if using Supabase)

### Recommended skills

Suggest installing complementary skills for ongoing development:
- **react-frontend** - Component patterns, hooks, performance optimization
- **fullstack-trpc** - tRPC procedures, auth, permissions (if fullstack)

### Available commands

```bash
$PKG_MGR run dev      # Start development server
$PKG_MGR run build    # Build for production
$PKG_MGR run test     # Run tests (if configured)
```
