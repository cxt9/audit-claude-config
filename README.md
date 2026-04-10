# audit-claude-config

A [Claude Code skill](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/overview) that audits your entire Claude configuration across all levels — global settings, Desktop app, project configs, and MCP servers — and produces a structured inventory with security findings and cleanup recommendations.

## What it does

When triggered, Claude will scan:

- **User-global config** (`~/.claude/`) — plugins, skills, permissions, hooks, blocklists
- **Claude Desktop app** — MCP servers, extensions, cowork settings, trusted folders
- **Project-level configs** — per-project `.claude/` directories, `.mcp.json` files, CLAUDE.md files
- **User-project overrides** — per-project settings stored outside the repo

And produce a report with:

1. **Summary table** — one row per config level
2. **Plugin inventory** — all installed plugins with versions and staleness indicators
3. **MCP server map** — every MCP server, where it appears, credential exposure, duplication
4. **Per-project breakdown** — what each project has configured
5. **Flags and recommendations** — security issues, duplication, stale configs, cost considerations

## Install

### Option 1: Copy to personal skills (recommended)

```bash
git clone https://github.com/cxt9/audit-claude-config.git
cp -r audit-claude-config ~/.claude/skills/audit-claude-config
```

### Option 2: Symlink (auto-updates with git pull)

```bash
git clone https://github.com/cxt9/audit-claude-config.git ~/audit-claude-config
ln -s ~/audit-claude-config ~/.claude/skills/audit-claude-config
```

### Option 3: Project-level only

```bash
# Inside your project directory:
mkdir -p .claude/skills
cp -r audit-claude-config .claude/skills/audit-claude-config
```

## Usage

In any Claude Code session, just ask:

- "audit my claude config"
- "what plugins do I have installed?"
- "what MCP servers do I have?"
- "show my config"
- "what's configured where?"

Claude will automatically match the skill based on your request and run the audit.

## Example output

```
| Level              | Plugins | MCP Servers | Skills | Key Flags                    |
|--------------------|---------|-------------|--------|------------------------------|
| User-Global        | 40      | 0           | 5      | Voice ON, custom status line |
| Claude Desktop     | 0       | 2           | 1 ext  | Cowork ON, 9 trusted folders |
| Personal Projects  | --      | 10 projects | varies | 3 with custom skills         |
| Work Projects      | --      | 12 projects | varies | Heavy AWS + Atlassian usage  |
```

The full report includes MCP server deduplication analysis, hardcoded credential warnings, and prioritized cleanup recommendations.

## What it checks for

### Security
- Hardcoded API tokens/keys in `.mcp.json` files
- Service role keys and database passwords in config files
- Overly broad permissions
- Suppressed security warnings

### Duplication
- MCP servers repeated across many projects that could be global
- Identical configs across projects

### Stale configs
- Old git worktrees
- Empty/minimal project configs
- Pending MCP authentication
- Unused plugins

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Works on macOS and Windows (checks platform-appropriate config paths)

## License

MIT
