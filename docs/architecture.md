# Architecture

## Overview

Mortolabs is a **Claude Code plugin marketplace** that distributes skills and commands for React development. Users can install plugins from this marketplace using:

```bash
/plugin install <plugin-name>@mortolabs
```

## Marketplace Structure

```
mortolabs/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace manifest (lists all plugins)
├── .claude/
│   └── settings.local.json   # Local Claude Code settings
├── docs/                     # Documentation
├── plugins/                  # Plugin directory
│   ├── react-frontend/       # Frontend skill plugin
│   ├── react-fullstack/      # Fullstack skill plugin
│   └── create-react-project/ # Project scaffolding command
```

## Plugin Types

### 1. Skill Plugins

Skills are model-invoked: Claude automatically uses them based on context.

| Plugin | Purpose | Auto-triggers on |
|--------|---------|------------------|
| `react-frontend` | Frontend development patterns | React components, hooks, state management |
| `react-fullstack` | Fullstack tRPC development | API procedures, auth, permissions, database |

**Skill structure:**
```
plugin-name/
├── SKILL.md          # Main skill definition (required)
├── LICENSE.txt       # License file
└── resources/        # Reference documentation
    └── *.md          # Detailed guides loaded on demand
```

### 2. Command Plugins

Commands are user-invoked via `/plugin-name:command`.

| Plugin | Command | Purpose |
|--------|---------|---------|
| `create-react-project` | `/create-react-project:init` | Interactive project scaffolding |

**Command structure:**
```
plugin-name/
├── commands/
│   └── command-name.md   # Command definition
└── LICENSE.txt
```

## Marketplace Configuration

The `.claude-plugin/marketplace.json` defines the marketplace:

```json
{
  "name": "mortolabs",
  "owner": { "name": "Jose Mortola" },
  "metadata": {
    "description": "Custom plugins and skills for Claude Code",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./plugins/plugin-name",
      "description": "...",
      "version": "1.0.0"
    }
  ]
}
```

## Skill Activation Flow

```
User Request
     │
     ▼
┌─────────────────┐
│ Claude analyzes │
│ request context │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     No match     ┌──────────────┐
│ Match against   │ ───────────────► │ Use default  │
│ skill names &   │                  │ behavior     │
│ descriptions    │                  └──────────────┘
└────────┬────────┘
         │ Match found
         ▼
┌─────────────────┐
│ Load SKILL.md   │
│ into context    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Load resources/ │ (on demand)
│ if referenced   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Apply skill     │
│ instructions    │
└─────────────────┘
```

## Tech Stack Coverage

### react-frontend Skill

- React 18+ with TypeScript strict mode
- Tailwind CSS + shadcn/ui
- TanStack Query (server state) + Zustand (client state)
- React Router 7 or Next.js patterns
- Conform or React Hook Form for forms

### react-fullstack Skill

- tRPC with React Query client
- Drizzle ORM with PostgreSQL
- Supabase Auth with server-side sessions
- Hierarchical permission system (Owner → Event → Entity)
- React Router 7 loaders/actions pattern

### create-react-project Command

- Interactive project scaffolding
- Supports pnpm, npm, bun, yarn
- React Router 7, Next.js, or Vite templates
- Optional fullstack setup (tRPC, Drizzle, Supabase)
