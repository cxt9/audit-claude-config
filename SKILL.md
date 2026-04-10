---
name: audit-claude-config
description: Use when needing a full inventory of Claude Code plugins, MCP servers, skills, permissions, and integrations across all configuration levels. Triggers on "what do I have installed", "audit my setup", "what MCP servers do I have", "what plugins", "show my config", "what's configured where", "claude setup overview"
---

# Audit Claude Configuration

Scan all Claude configuration levels and produce a structured inventory of plugins, MCP servers, skills, permissions, and integrations.

## When to Use

- User wants to understand what's installed/configured across their Claude setup
- Auditing for security (hardcoded credentials, overly broad permissions)
- Cleaning up stale or duplicate configurations
- Onboarding to an existing machine's Claude setup

## Configuration Levels

Scan these locations in order. Each level can override or extend the ones above it.

### Level 1: User-Global

| Location | What to Check |
|----------|---------------|
| `~/.claude/settings.json` | Enabled plugins, permissions, hooks, voice, status line |
| `~/.claude/CLAUDE.md` | Global instructions |
| `~/.claude/plugins/installed_plugins.json` | Full plugin inventory with versions |
| `~/.claude/plugins/blocklist.json` | Blocked plugins |
| `~/.claude/skills/` | Personal skills (list dirs) |
| `~/.claude/keybindings.json` | Custom key bindings |
| `~/.claude/teams/` | Team configurations (if present) |
| `~/.claude/ide/` | IDE integration configs (VS Code, JetBrains) |
| `~/.claude/mcp-needs-auth-cache.json` | MCP servers pending authentication |
| `~/.claude/security_warnings_state_*.json` | Suppressed security warnings |

### Level 2: Claude Desktop App

| Location | What to Check |
|----------|---------------|
| `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) | MCP servers, trusted folders, cowork settings, preferences |
| `%APPDATA%/Claude/claude_desktop_config.json` (Windows) | Same as above |
| `~/Library/Application Support/Claude/Claude Extensions/` (macOS) | Installed desktop extensions |
| `~/Library/Application Support/Claude/extensions-installations.json` | Extension install tracking |
| `~/Library/Application Support/Claude/extensions-blocklist.json` | Blocked extensions |
| `~/Documents/Claude/` | Cowork projects and scheduled tasks |

### Level 3: Project-Level

Search the user's project directories (ask if not obvious):

| Pattern | What to Check |
|---------|---------------|
| `<project>/.claude/settings.local.json` | Project permissions, enabled MCP servers, experimental flags |
| `<project>/.claude/settings.json` | Shared project settings (committed to git) |
| `<project>/.mcp.json` | Project MCP server definitions (servers, credentials, args) |
| `<project>/CLAUDE.md` | Project-specific instructions |
| `<project>/.claude/skills/` | Project-specific skills |
| `<project>/.claude/commands/` | Custom slash commands |

### Level 4: User-Project Overrides

| Location | What to Check |
|----------|---------------|
| `~/.claude/projects/<hashed-path>/` | Per-project settings stored at user level (not in repo) |

## Execution Strategy

**Use 2-3 parallel tool calls or subagents** for speed:
1. Batch 1: User-global + Desktop app (Levels 1-2)
2. Batch 2: All project-level configs (Level 3) — search per known directory rather than a single deep glob (deep `**` globs can time out on large trees)
3. Batch 3 (if needed): Deep-read large configs found in Batch 2

**Inferring project locations:** Check the working directory, `~/Developer`, `~/Projects`, `~/repos`, `~/code`. If none contain projects, ask the user. In a subagent context where you cannot ask, search common locations and note which were checked.

## Output Format

Structure the report as:

### 1. Summary Table (one row per config level)

```
| Level | Plugins | MCP Servers | Skills | Key Flags |
```

### 2. Plugin Inventory

List all installed plugins with source, version, and whether actively used or potentially stale.

### 3. MCP Server Map

For each unique MCP server found:
- Name and type (Docker, npx, remote, etc.)
- Which levels/projects it appears in (flag duplicates)
- Credentials present (YES/NO - never print actual secrets)
- Auth status (authenticated, pending, unknown)

Include a separate row for **Desktop Extensions** (these are distinct from project-level MCP servers).

### 4. Per-Project Breakdown

Table with columns: Project | MCP Servers | Permissions Count | Notable Config

Group by project directory (e.g., work vs. personal) when the user has distinct top-level directories.

### 5. Flags and Recommendations

Group into:

**Security**: Hardcoded credentials, overly broad permissions, suppressed warnings
**Duplication**: MCP servers or configs repeated across projects that could be global
**Stale**: Empty configs, old worktrees, unused plugins, pending auth
**Cost**: Docker-based MCP servers, plugins that may not be needed

## Common MCP Server Types

These appear frequently and are candidates for global config:
- **Context7** (docs lookup) - almost always duplicated per-project
- **Browser-Tools / Puppeteer** - browser automation
- **Atlassian** (Docker) - Jira/Confluence
- **Supabase** - database
- **Firebase** - backend
- **Filesystem** - local file access
- **AWS MCP servers** - cloud operations

## Security Checks

Flag but **never print** actual values for:
- API tokens/keys in `.mcp.json` files
- Service role keys in settings
- Database connection strings with credentials
- Database passwords in MCP configs
- OAuth tokens

Report as: "Atlassian API token found in plaintext in 7 project .mcp.json files"

**When reading configs that may contain secrets:** Extract only key names and structure. If a config file has been read and contains plaintext secrets, do not reproduce them in the output. Summarize as "credentials present: YES" with the credential type.
