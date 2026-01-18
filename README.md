# Claude Helpers

A Claude Code plugin marketplace for React development.

## Installation

Add the marketplace and install plugins in Claude Code:

```bash
# Add marketplace
/plugin marketplace add jlmortola/claude-helpers

# Install plugins
/plugin install react-skills@claude-helpers
/plugin install dev-tools@claude-helpers
```

## Available Plugins

### react-skills

**Type:** Skills (auto-activate based on context)

Bundled React development skills:

#### react-frontend

Frontend patterns for React 18+, TypeScript, Tailwind CSS, and shadcn/ui:

- Component patterns and props typing
- Custom hooks and state management
- TanStack Query and Zustand patterns
- React Router 7 loaders/actions
- Performance optimization
- Accessibility

#### react-fullstack

Fullstack development with tRPC, Drizzle ORM, and Supabase Auth:

- tRPC procedure creation
- Database operations with Drizzle
- Authentication flows
- Permission system
- Security best practices

---

### dev-tools

**Type:** Commands

Development tools for project setup and documentation:

#### docs:create

Generate project documentation automatically.

```bash
/dev-tools:create
```

Features:
- Creates `docs/` folder with architecture, development, API docs
- Generates `CLAUDE.md` quick reference
- Optionally creates `PROJECT_INDEX.json` for codebase analysis (requires Python)
- Uses git to detect changes and only update affected docs

#### init

Interactive project scaffolding wizard.

```bash
/dev-tools:init my-app
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
