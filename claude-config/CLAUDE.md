## ⛔ ABSOLUTE PRIORITIES - READ FIRST

### WRITING COMMENTS

- Write clear, concise, and informative comments. 
- Only include comments that add value and context to the code. 
- Avoid redundant or obvious comments, especially when the variable or function name is self-explanatory.
- Code should be self-documenting. If you need a comment to explain what the code does, consider refactoring the code to make it clearer.

### 🔍 MANDATORY SEARCH TOOL: ripgrep (rg)

**OBLIGATORY RULE**: ALWAYS use `ripgrep` (command: `rg`) as your PRIMARY and FIRST tool for ANY code search, pattern matching, or grepping task. This is NON-NEGOTIABLE.

**Basic syntax**:
```bash
# Basic search (recursively searches current directory by default)
rg '<pattern>'

# Search specific file types
rg '<pattern>' -t <type>

# Common types: py, ts, js, tsx, java, rust, go, md, yaml, json
```

**Common usage patterns**:
```bash
# Find function definitions in Python
rg 'def \w+\(' -t py

# Find class declarations
rg 'class \w+' -t py

# Find imports in TypeScript
rg "import .* from" -t ts

# Find React components (function declarations)
rg 'function \w+\(' -t tsx

# Find async functions in Python
rg 'async def \w+' -t py

# Case-insensitive search
rg -i '<pattern>'

# Show context lines (before/after)
rg -C 3 '<pattern>'

# Search with glob patterns
rg '<pattern>' -g '*.tsx'

# List files containing matches (no content)
rg -l '<pattern>'

# Count matches per file
rg -c '<pattern>'
```

**When to use each tool**:
- ✅ **ripgrep (rg)**: 95% of cases - fast, respects .gitignore, regex support, file type filtering
- ⚠️ **grep**: ONLY as fallback if rg is unavailable
- ❌ **NEVER** use `grep -r` when `rg` is available

**Enforcement**: If you use `grep -r` for code searching without attempting `rg` first, STOP and retry with ripgrep. This is a CRITICAL requirement.

### 🐍 Running Python with uv

**PREFERRED**: Use `uv run` instead of `python3` or `python3 -c` for running Python scripts and one-liners. uv automatically manages environments and dependencies.

**Quick one-liners** (instead of `python3 -c`):
```bash
# Run inline code via stdin
echo 'print("Hello world!")' | uv run -

# Using here-documents for multi-line
uv run - <<EOF
import json
print(json.dumps({"key": "value"}, indent=2))
EOF
```

**Running scripts with dependencies**:
```bash
# Run a script that needs packages (no manual pip install!)
uv run --with requests script.py

# Multiple dependencies
uv run --with requests --with rich script.py

# With version constraints
uv run --with 'requests<3' --with 'rich>12' script.py
```

**Inline script metadata** (declare deps in the script itself):
```python
# /// script
# dependencies = ["requests", "rich"]
# requires-python = ">=3.12"
# ///

import requests
from rich import print
# ...
```

Then just run with `uv run script.py` — deps are auto-installed.

**Why use uv**:
- ✅ No manual `pip install` or virtualenv management
- ✅ Reproducible — dependencies declared explicitly
- ✅ Fast — aggressive caching
- ✅ Works in projects or standalone

Reference: [uv scripts guide](https://docs.astral.sh/uv/guides/scripts/)

### Using proper Git commands

- Never use `git -C <path> status|commit|add|etc.` commands. Always use `git status|commit|add|etc. <path>` commands.
