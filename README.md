# Mortolabs

A Claude Code plugin marketplace for React development.

## Installation

Install plugins directly in Claude Code:

```bash
/plugin install react-frontend@mortolabs
/plugin install react-fullstack@mortolabs
/plugin install create-react-project@mortolabs
```

## Available Plugins

### react-frontend

**Type:** Skill (auto-activates)

Frontend development patterns for React 18+, TypeScript, Tailwind CSS, and shadcn/ui. Covers:

- Component patterns and props typing
- Custom hooks and state management
- TanStack Query and Zustand patterns
- React Router 7 loaders/actions
- Performance optimization
- Accessibility

### react-fullstack

**Type:** Skill (auto-activates)

Fullstack development with tRPC, Drizzle ORM, and Supabase Auth. Covers:

- tRPC procedure creation
- Database operations with Drizzle
- Authentication flows
- Permission system
- Security best practices

### create-react-project

**Type:** Command

Interactive project scaffolding wizard.

```bash
/create-react-project:init my-app
```

Supports:
- Multiple package managers (pnpm, npm, bun, yarn)
- Multiple frameworks (React Router 7, Next.js, Vite)
- Frontend-only or fullstack with tRPC
- shadcn/ui setup
- Testing configuration

## Local Development

Test plugins without installing:

```bash
claude --plugin-dir /path/to/mortolabs
```

## Documentation

- [Architecture](docs/architecture.md) - How the marketplace works
- [Development](docs/development.md) - Creating new plugins
- [Plugins](docs/plugins.md) - Detailed plugin documentation

## License

MIT
