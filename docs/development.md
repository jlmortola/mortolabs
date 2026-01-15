# Development

## Prerequisites

- Claude Code CLI installed and authenticated
- Git

## Testing Plugins Locally

Load the marketplace directly without installing:

```bash
# Load entire marketplace
claude --plugin-dir ./

# Load specific plugin for testing
claude --plugin-dir ./plugins/react-frontend
```

## Creating a New Skill Plugin

1. Create the plugin directory:
   ```bash
   mkdir -p plugins/my-skill/resources
   ```

2. Create `plugins/my-skill/SKILL.md`:
   ```markdown
   ---
   name: my-skill
   description: Brief description. Use when [trigger conditions].
   license: LICENSE.txt
   ---

   # My Skill

   ## Instructions
   [What Claude should do when this skill is active]

   ## References
   - See [resources/details.md](resources/details.md) for more
   ```

3. Add to `marketplace.json`:
   ```json
   {
     "name": "my-skill",
     "source": "./plugins/my-skill",
     "description": "...",
     "version": "1.0.0"
   }
   ```

4. Copy LICENSE.txt from an existing plugin.

## Creating a New Command Plugin

1. Create the plugin directory:
   ```bash
   mkdir -p plugins/my-command/commands
   ```

2. Create `plugins/my-command/commands/run.md`:
   ```markdown
   ---
   description: What this command does
   argument-hint: [args]
   allowed-tools: Bash(*), Read, Write
   ---

   # My Command

   Instructions for Claude when /my-command:run is invoked.
   ```

3. Add to `marketplace.json`.

## Plugin Structure Reference

### Skill Plugin
```
my-skill/
├── SKILL.md          # Required - main skill definition
├── LICENSE.txt       # Required - license file
└── resources/        # Optional - detailed documentation
    ├── patterns.md
    └── examples.md
```

### Command Plugin
```
my-command/
├── commands/         # Required - command definitions
│   ├── run.md
│   └── other.md
└── LICENSE.txt
```

## SKILL.md Frontmatter

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Skill identifier (lowercase, hyphens) |
| `description` | Yes | When to use this skill (max 1024 chars) |
| `license` | No | Path to license file |
| `allowed-tools` | No | Restrict tools when skill is active |
| `model` | No | Specific model to use |

## Command Frontmatter

| Field | Required | Description |
|-------|----------|-------------|
| `description` | Yes | What the command does |
| `argument-hint` | No | Expected arguments |
| `allowed-tools` | No | Tools the command can use |
| `model` | No | Specific model to use |

## Publishing Updates

1. Update version in `marketplace.json`
2. Commit changes
3. Push to repository

Users will get updates when they reinstall or Claude Code refreshes.

## Testing Commands

After loading with `--plugin-dir`:

```bash
# List available commands
/help

# Run a command
/create-react-project:init my-app
```

## Testing Skills

Skills activate automatically. Test by:

1. Starting a conversation relevant to the skill
2. Asking Claude "What skills are available?"
3. Verifying the skill is listed and triggers appropriately
