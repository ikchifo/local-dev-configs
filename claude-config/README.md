# Claude Code Configuration

Portable Claude Code configuration with custom agents, skills, hooks, and settings.

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- [Bun](https://bun.sh/) (for statusline and hooks)
- [terminal-notifier](https://github.com/julienXX/terminal-notifier) (macOS notifications)
- [fzf](https://github.com/junegunn/fzf) and [ripgrep](https://github.com/BurntSushi/ripgrep) (for file suggestions)

```bash
# Install dependencies (macOS)
brew install bun terminal-notifier fzf ripgrep
```

## Installation

### 1. Copy configuration to ~/.claude

```bash
# Create ~/.claude if it doesn't exist
mkdir -p ~/.claude

# Copy all config files
cp -r agents skills hooks commands ~/.claude/
cp file-suggestion.sh CLAUDE.md QUICK_REFERENCE.md SETUP_SUMMARY.md ~/.claude/

# Copy and customize settings
cp settings.template.json ~/.claude/settings.json
```

### 2. Edit settings.json

Update `~/.claude/settings.json` with your values:

```json
{
  "env": {
    "ANTHROPIC_VERTEX_PROJECT_ID": "<YOUR_VERTEX_PROJECT_ID>",
    ...
  }
}
```

If not using Vertex AI, remove the `env` block entirely.

### 3. Install hook dependencies

```bash
cd ~/.claude/hooks
npm install
```

### 4. Make scripts executable

```bash
chmod +x ~/.claude/hooks/*.sh
chmod +x ~/.claude/file-suggestion.sh
```

### 5. Install statusline (optional)

The statusline shows session cost, token usage, and context window percentage.

Uses [ccstatusline](https://github.com/sirmalloc/ccstatusline):

```bash
# Already configured in settings.json to auto-run via bunx
# Or install globally:
bun install -g ccstatusline
```

See the [ccstatusline docs](https://github.com/sirmalloc/ccstatusline#-quick-start) for customization options.

### 6. Install plugins

Install the enabled plugins via Claude Code:

```bash
claude
# Then run:
/plugins install superpowers@superpowers-marketplace
/plugins install playwright-skill@playwright-skill
/plugins install shell-scripting@claude-code-workflows
/plugins install code-simplifier@claude-plugins-official
/plugins install frontend-design@claude-plugins-official
/plugins install gopls-lsp@claude-plugins-official
```

## What's Included

### Agents

Custom agent definitions for specialized tasks:

| Agent | Description |
|-------|-------------|
| `golang-pro` | Go development with idiomatic patterns |
| `python-pro` | Python with modern tooling (uv, pytest, ruff) |
| `api-designer` | REST/GraphQL API design |
| `code-reviewer` | Code quality and security review |
| `performance-engineer` | Performance optimization |
| `platform-engineer` | Platform and infrastructure |
| `react-specialist` | React 18+ patterns |
| `refactoring-specialist` | Safe code refactoring |
| `rust-engineer` | Rust systems programming |

### Skills

Auto-activating skills based on context:

| Skill | Triggers |
|-------|----------|
| `golang-dev-guidelines` | Go files, `go`, `golang` keywords |
| `python-dev-guidelines` | Python files, `python`, `fastapi` keywords |
| `java-dev-guidelines` | Java files, `spring`, `maven` keywords |
| `uv-python-scripts` | Creating standalone Python scripts |
| `mermaid-diagrams` | Creating diagrams, flowcharts |
| `rfc-writer` | Writing technical RFCs |
| `design-principles` | UI/dashboard design |

### Hooks

| Hook | Trigger | Purpose |
|------|---------|---------|
| `skill-activation-prompt` | UserPromptSubmit | Auto-suggests relevant skills |
| `post-tool-use-tracker` | Edit/Write tools | Tracks file changes |

### Settings Features

- **File suggestions**: rg + fzf powered fuzzy file finder
- **Statusline**: Cost, tokens, context % display
- **Notifications**: macOS alerts for input needed / task complete
- **Auto-thinking**: Extended thinking enabled by default
- **Pre-approved commands**: Common safe commands auto-approved

## Customization

### Add project-specific skill triggers

Edit `~/.claude/skills/skill-rules.json`:

```json
{
  "golang-dev-guidelines": {
    "fileTriggers": {
      "pathPatterns": [
        "**/*.go",
        "your-project/**/*.go"
      ]
    }
  }
}
```

### Add custom permissions

Edit `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(your-command:*)",
      "WebFetch(domain:your-docs.com)"
    ]
  }
}
```

## Troubleshooting

### Skills not activating

```bash
# Check hooks are executable
ls -la ~/.claude/hooks/*.sh

# Reinstall dependencies
cd ~/.claude/hooks && npm install

# Validate JSON
cat ~/.claude/skills/skill-rules.json | jq .
```

### Test hooks manually

```bash
echo '{"prompt":"create python api","session_id":"test"}' | \
  ~/.claude/hooks/skill-activation-prompt.sh
```

## Documentation

- `CLAUDE.md` - Global instructions for all projects
- `QUICK_REFERENCE.md` - Skill quick reference
- `SETUP_SUMMARY.md` - Detailed setup documentation
