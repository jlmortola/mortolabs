---
description: Regenerate or create project documentation (docs/, CLAUDE.md). Uses git to detect changes and only updates affected docs.
allowed-tools: Bash(python:*), Bash(python3:*), Bash(which:*), Bash(ls:*), Bash(mkdir:*), Bash(git:*), Read, Write, Edit, AskUserQuestion
---

# Update Project Documentation

Regenerate or create project documentation based on codebase analysis. Use git to detect changes and only update relevant documentation.

## Step 1: Pre-flight Checks

Run these checks first and store results:

```
CHECK_1: Does PROJECT_INDEX.json exist in project root?
CHECK_2: Does docs/ folder exist in project root?
CHECK_3: Does CLAUDE.md exist in project root?
CHECK_4: Is this a git repository?
```

## Step 2: Decision Matrix

Based on checks, follow this logic:

| PROJECT_INDEX.json | docs/ folder | Action |
|:------------------:|:------------:|--------|
| EXISTS | EXISTS | Refresh index first, then update docs/ using index as source |
| EXISTS | MISSING | **Create docs/ using PROJECT_INDEX.json as blueprint** |
| MISSING | EXISTS | Update docs/ by scanning codebase directly |
| MISSING | MISSING | Create docs/ by scanning codebase directly |

**If PROJECT_INDEX.json does NOT exist:**

Ask the user using AskUserQuestion:
- "PROJECT_INDEX.json not found. Would you like to create it first? This provides better documentation by analyzing your codebase structure."
- Options: "Yes, create index first" / "No, scan manually"

If user accepts, run the indexer (requires Python):

1. Check Python availability:
   ```bash
   which python3 || which python
   ```

2. If Python found, run the bundled indexer:
   ```bash
   # Try plugin cache location first
   python3 ~/.claude/plugins/cache/docs@mortolabs/commands/scripts/project_index.py
   ```

   If that fails, inform user the scripts need to be accessible and suggest:
   - Installing the plugin: `/plugin install docs@mortolabs`
   - Or manually running from the plugin source

3. After indexer completes, continue with documentation generation using the new PROJECT_INDEX.json.

If Python is not found, inform user:
- "Python 3 is required for the indexer. Continuing with manual codebase scan."
- Provide link: https://www.python.org/downloads/

## Step 3: Detect Changes Using Git

**If docs/ folder EXISTS (incremental update):**

Use git to detect what has changed instead of scanning everything.

### 3.1 Find last documentation update

```bash
# Get the last commit that touched docs/ or CLAUDE.md
git log -1 --format="%H" -- docs/ CLAUDE.md
```

If no commit found (first time), do a full scan (skip to Step 4).

### 3.2 Get changed files since last docs update

```bash
# Committed changes since last docs update
git diff --name-only <last-docs-commit>..HEAD

# Uncommitted changes (staged and unstaged)
git diff --name-only HEAD
git diff --name-only --cached
```

### 3.3 Map changed files to documentation

Use this mapping to determine which docs need updating:

| Changed Files Pattern | Update These Docs |
|-----------------------|-------------------|
| `app/routes/*`, `pages/*` | `api.md`, `architecture.md` |
| `app/.server/api/*`, `server/api/*` | `api.md` |
| `**/schema.ts`, `drizzle.config.*`, `prisma/*` | `database.md` |
| `**/*auth*`, `**/*session*`, `**/*login*` | `authentication.md` |
| `**/*permission*`, `**/*access*`, `**/*role*` | `permissions.md` |
| `package.json`, `pnpm-lock.yaml`, `package-lock.json` | `development.md`, `CLAUDE.md` |
| `app/components/*`, `components/*` | `architecture.md` |
| `app/.server/*`, `server/*`, `lib/*` | `architecture.md` |
| `tests/*`, `__tests__/*`, `*.test.*`, `*.spec.*` | `testing.md` |
| `tsconfig.json`, `vite.config.*`, `*.config.*` | `development.md` |
| `.env.example`, `.env.sample` | `development.md` |

### 3.4 Read only changed files

Instead of scanning the entire codebase:
1. Only read files that have changed
2. Only update documentation sections affected by those changes
3. Preserve unchanged sections in existing docs

**If docs/ folder MISSING (full scan):**

Skip change detection and do a complete codebase scan (Step 4).

## Step 4: Gather Project Information

**If doing incremental update (changes detected):**
- Only read the changed files identified in Step 3
- Focus on extracting information relevant to affected docs

**If doing full scan (no docs/ or first time):**

**If PROJECT_INDEX.json EXISTS:**
1. Read PROJECT_INDEX.json first
2. Extract: architecture, modules, endpoints, schemas, patterns
3. Use this as your primary data source

**If PROJECT_INDEX.json MISSING:**
Scan codebase manually:
1. Read `package.json` for:
   - Project name and description
   - Scripts (dev, build, test, etc.)
   - Dependencies (detect framework: react-router, next, remix, express, etc.)
   - Dev dependencies (testing tools, linters)

2. Scan directory structure:
   - `app/` or `src/` - main source code
   - `app/.server/` or `server/` - server-only code
   - `app/routes/` or `pages/` - routing
   - `app/components/` or `components/` - UI components
   - `lib/` or `utils/` - utilities
   - `tests/` or `__tests__/` - test files

3. Find key patterns by searching for:
   - Database: `schema.ts`, `drizzle.config`, `prisma/schema.prisma`
   - Auth: files containing `auth`, `session`, `login`
   - API: `trpc`, `api/`, route handlers
   - Permissions: files containing `permission`, `access`, `role`

## Step 5: Create/Update docs/ Folder

If docs/ folder is missing, create it:
```bash
mkdir -p docs
```

Generate these documentation files based on what exists in the project:

### Always Create:

**docs/architecture.md**
```markdown
# Architecture

## Overview
[Brief description of the project and its purpose]

## Tech Stack
- Framework: [detected framework]
- Language: [TypeScript/JavaScript]
- Database: [if detected]
- Styling: [if detected]

## Directory Structure
[Map of key directories and their purposes]

## Data Flow
[How data moves through the application]

## Key Patterns
[Important architectural decisions and patterns used]
```

**docs/development.md**
```markdown
# Development

## Prerequisites
[Required tools and versions]

## Setup
[Step-by-step setup instructions]

## Commands
[All npm/pnpm scripts with descriptions]

## Environment Variables
[Required env vars, without actual values]

## Troubleshooting
[Common issues and solutions]
```

### Create If Detected:

**docs/database.md** (if schema files found)
```markdown
# Database

## Overview
[Database type and ORM used]

## Schema
[Tables/collections and their relationships]

## Migrations
[How to run migrations]

## Key Queries
[Important query patterns]
```

**docs/api.md** (if API routes found)
```markdown
# API

## Overview
[API architecture: REST, tRPC, GraphQL]

## Endpoints/Procedures
[List of all endpoints with descriptions]

## Authentication
[How API auth works]

## Error Handling
[Error response patterns]
```

**docs/authentication.md** (if auth code found)
```markdown
# Authentication

## Overview
[Auth provider/method used]

## Flow
[Login/logout/session flow]

## Protected Routes
[How routes are protected]

## Key Functions
[Important auth utilities]
```

**docs/permissions.md** (if permission system found)
```markdown
# Permissions

## Overview
[Permission model used]

## Roles
[Available roles and their capabilities]

## Access Control
[How permissions are checked]

## Key Functions
[Permission utility functions]
```

**docs/testing.md** (if test files found)
```markdown
# Testing

## Overview
[Testing framework and approach]

## Running Tests
[Commands to run tests]

## Test Structure
[How tests are organized]

## Writing Tests
[Guidelines for new tests]
```

## Step 6: Update CLAUDE.md

Create or update CLAUDE.md in project root with this structure:

```markdown
# Project: [Project Name]

[One-line description]

## Quick Reference

\`\`\`bash
# Development
[key commands from package.json]
\`\`\`

## Architecture Rules

**[Most important rule in bold]**

- [Key architectural rules - bullet points]
- [Data flow patterns]
- [What NOT to do]

## Data Flow Pattern

\`\`\`
[Simple ASCII diagram of data flow]
\`\`\`

## Key Directories

- `[dir]/` - [purpose]
- `[dir]/` - [purpose]

## [Key System 1 - e.g., Authentication]

[Brief description and key functions]

## [Key System 2 - e.g., Permissions]

[Brief description and key functions]

## Code Style

- [Language and type preferences]
- [Styling approach]
- [Component patterns]
- [Date/utility libraries]

## Security Checklist

- [Security requirements as checklist]

## Detailed Documentation

See `docs/` folder for comprehensive guides:
- [Architecture](docs/architecture.md) - [brief description]
- [Development](docs/development.md) - [brief description]
[...other docs that exist]
```

## Step 7: Summary

After completing all updates, provide a summary:

```
Documentation Updated:

Mode: [Full scan | Incremental update]
Changes detected: [list of changed files or "N/A for full scan"]

Created/Updated:
- [ ] PROJECT_INDEX.json (refreshed via /index)
- [x] docs/architecture.md
- [x] docs/development.md
- [x] docs/database.md
- [ ] docs/api.md (skipped - no changes detected)
- [x] CLAUDE.md

Files analyzed: [count]
Skipped (unchanged): [count]
```

---

## Style Guidelines

When writing documentation:

1. **Be direct** - No fluff or filler words
2. **Be comprehensive** - Cover all important aspects
3. **Be practical** - Focus on what developers need to know
4. **Use examples** - Show, don't just tell
5. **Use code blocks** - For commands, code snippets, file paths
6. **Use tables** - For structured comparisons
7. **Use ASCII diagrams** - For visual flows (keep simple)
8. **Link related docs** - Cross-reference between files
9. **No emojis** - Unless explicitly requested
10. **Keep CLAUDE.md concise** - It's a quick reference, details go in docs/

---

## Notes

- Preserve any custom sections users have added to existing docs
- If a doc file exists, update it rather than overwrite completely
- Detect project type from package.json dependencies
- Adapt terminology to the framework (routes vs pages, loaders vs getServerSideProps, etc.)
- For incremental updates, only modify sections affected by changed files
- Always show what was skipped in the summary so user knows what wasn't checked
