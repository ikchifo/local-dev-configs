# Claude Code Skills - Quick Reference

## Auto-Activated Skills

### Python Development
**Triggers:** `python`, `fastapi`, `flask`, `pytest`, `async`, `pandas`
**Files:** `*.py`, `pyproject.toml`
**Skill:** python-dev-guidelines

### Golang Development
**Triggers:** `go`, `golang`, `k8s controller`, `goroutine`, `grpc`
**Files:** `*.go`, `go.mod`
**Skill:** golang-dev-guidelines

---

## Superpowers Workflows

### 🔴 CRITICAL Priority (Always Use)

#### Debugging
**Triggers:** `bug`, `error`, `failing`, `broken`, `crash`
**Skill:** superpowers:systematic-debugging
**Process:** Investigate → Analyze → Test hypothesis → Fix

#### Verification
**Triggers:** `done`, `complete`, `commit`, `pr`
**Skill:** superpowers:verification-before-completion
**Process:** Run tests → Check output → Verify claims

---

### 🟡 HIGH Priority (Recommended)

#### Test-Driven Development
**Triggers:** `implement feature`, `add feature`, `fix bug`
**Skill:** superpowers:test-driven-development
**Process:** RED (fail) → GREEN (pass) → REFACTOR

#### Brainstorming
**Triggers:** `design`, `architecture`, `how should`, `best way`
**Skill:** superpowers:brainstorming
**Process:** Refine ideas before coding

#### Code Review
**Triggers:** `review`, `code review`, `verify implementation`
**Skill:** superpowers:requesting-code-review
**Process:** Systematic review before merge

#### Testing Anti-Patterns
**Triggers:** `write test`, `mock`, `stub`
**Skill:** superpowers:testing-anti-patterns
**Process:** Avoid testing mocks, test-only production code

#### Root Cause Tracing
**Triggers:** `trace error`, `root cause`, `stack trace`
**Skill:** superpowers:root-cause-tracing
**Process:** Trace back through call stack

---

### 🟢 MEDIUM Priority (Helpful)

#### Planning
**Triggers:** `create plan`, `implementation plan`, `roadmap`
**Skill:** superpowers:writing-plans
**Process:** Detailed task breakdowns

#### Executing Plans
**Triggers:** `execute plan`, `run plan`
**Skill:** superpowers:executing-plans
**Process:** Batch execution with checkpoints

#### Git Worktrees
**Triggers:** `feature work`, `new branch`, `isolate work`
**Skill:** superpowers:using-git-worktrees
**Process:** Isolated development branches

#### Condition-Based Waiting
**Triggers:** `flaky test`, `race condition`, `timeout`
**Skill:** superpowers:condition-based-waiting
**Process:** Replace timeouts with condition polling

---

## Common Commands

### Python
```bash
# Run tests
pytest

# Type checking
mypy src/

# Lint & format
ruff check .
ruff format .

# Run with uv
uv run python -m myapp.main

# Add dependencies
uv add package-name
uv add --dev pytest mypy
```

### Golang
```bash
# Run tests
go test ./...
go test -race ./...

# Run specific test
go test -run TestName ./pkg/service

# Build
go build -o bin/app ./cmd/app

# Format
gofmt -w .
go vet ./...

# Update deps
go mod tidy
go get -u ./...
```

---

## Customization Locations

**Skill Rules:** `~/.claude/skills/skill-rules.json`
**Settings:** `~/.claude/settings.json`
**Hooks:** `~/.claude/hooks/`

### Add Custom Keywords

Edit `~/.claude/skills/skill-rules.json`:

```json
{
  "python-dev-guidelines": {
    "promptTriggers": {
      "keywords": [
        "python",
        // ADD YOUR TERMS:
        "vertex-ai",
        "kubeflow"
      ]
    }
  }
}
```

### Add Project Paths

```json
{
  "fileTriggers": {
    "pathPatterns": [
      "**/*.py",
      // ADD YOUR PATHS:
      "services-pilot/**/*.go",
      "_hyperkube/**/*.py"
    ]
  }
}
```

---

## Troubleshooting

**Skills not activating?**
```bash
# Check hooks are executable
ls -la ~/.claude/hooks/*.sh

# Reinstall dependencies
cd ~/.claude/hooks && npm install

# Validate JSON
cat ~/.claude/skills/skill-rules.json | jq .
```

**Test hook manually:**
```bash
echo '{"prompt":"create python api","session_id":"test"}' | \
  ~/.claude/hooks/skill-activation-prompt.sh
```

---

## When to Use What

| Situation | Skill to Use |
|-----------|--------------|
| Creating new Python API | python-dev-guidelines + brainstorming |
| Creating new Go service | golang-dev-guidelines + brainstorming |
| Bug in tests | systematic-debugging |
| Implementing new feature | test-driven-development |
| Before committing | verification-before-completion |
| Writing tests | testing-anti-patterns |
| Planning large feature | writing-plans |
| Race condition in tests | condition-based-waiting |
| Complex error to debug | root-cause-tracing |
| Starting isolated feature | using-git-worktrees |
| Ready to merge | requesting-code-review |

---

## Quick Tips

1. **Let skills suggest themselves** - The system watches your prompts
2. **Skills stack** - Multiple skills can activate together
3. **Critical skills** - Always use for debugging and verification
4. **File triggers** - Skills activate when editing certain files
5. **Customize freely** - Add your own keywords and paths

---

For full details, see: `~/.claude/SETUP_SUMMARY.md`
