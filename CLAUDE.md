# Claude Helpers

Claude Code plugin marketplace for React development skills and commands.

## Quick Reference

```bash
# Add marketplace
/plugin marketplace add jlmortola/claude-helpers

# Install plugins
/plugin install react-skills@claude-helpers
/plugin install dev-tools@claude-helpers

# Test locally
claude --plugin-dir ./
```

## Available Plugins

| Plugin | Contains | Type |
|--------|----------|------|
| `react-skills` | react-frontend, react-fullstack | Skills (auto-trigger) |
| `dev-tools` | docs, create-react-project | Commands |

### react-skills

Auto-triggers on React/UI and API/backend work. Includes:
- **react-frontend**: React 18+, TypeScript, Tailwind, shadcn/ui patterns
- **react-fullstack**: tRPC, Drizzle ORM, Supabase Auth patterns

### dev-tools

Commands for project setup and documentation:
- `/dev-tools:create` - Generate docs/, CLAUDE.md, PROJECT_INDEX.json
- `/dev-tools:init` - Scaffold new React projects

## Directory Structure

```
plugins/
├── react-frontend/       # Frontend skill
│   ├── SKILL.md
│   └── resources/
├── react-fullstack/      # Fullstack skill
│   ├── SKILL.md
│   └── resources/
├── create-react-project/ # Scaffolding command
│   └── commands/
└── docs/                 # Documentation tools
    └── commands/
        ├── create.md
        └── scripts/
```

## Adding Skills/Commands

1. Create directory in `plugins/`
2. Add `SKILL.md` (for skills) or `commands/*.md` (for commands)
3. Register in `.claude-plugin/marketplace.json` under appropriate bundle
4. Add `LICENSE.txt`

## Marketplace Config

Location: `.claude-plugin/marketplace.json`

```json
{
  "plugins": [
    {
      "name": "bundle-name",
      "source": "./",
      "strict": false,
      "skills": ["./plugins/skill-name"],
      "commands": ["./plugins/command-name/commands"]
    }
  ]
}
```

## Plugin Types

**Skills** - Model-invoked based on context
- Require `SKILL.md` with `name` and `description` frontmatter
- Optional `resources/` for detailed docs

**Commands** - User-invoked via `/plugin:command`
- Require `commands/*.md` with `description` frontmatter
- Filename becomes command name

## Detailed Documentation

- [Architecture](docs/architecture.md) - Marketplace structure and data flow
- [Development](docs/development.md) - Creating and testing plugins
- [Plugins](docs/plugins.md) - Detailed plugin documentation
