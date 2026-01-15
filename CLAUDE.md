# Mortolabs

Claude Code plugin marketplace for React development skills and commands.

## Quick Reference

```bash
# Test plugins locally
claude --plugin-dir ./

# Users install with
/plugin install <plugin-name>@mortolabs
```

## Available Plugins

| Plugin | Type | Invocation |
|--------|------|------------|
| `react-frontend` | Skill | Auto-triggers on React/UI work |
| `react-fullstack` | Skill | Auto-triggers on API/backend work |
| `create-react-project` | Command | `/create-react-project:init` |
| `docs` | Command | `/docs:create` |

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
        ├── create.md     # Creates docs/, CLAUDE.md (+ optional index)
        └── scripts/      # Python indexer for PROJECT_INDEX.json
```

## Adding a Plugin

1. Create directory in `plugins/`
2. Add `SKILL.md` (for skills) or `commands/*.md` (for commands)
3. Register in `.claude-plugin/marketplace.json`
4. Add `LICENSE.txt`

## Marketplace Config

Location: `.claude-plugin/marketplace.json`

```json
{
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
