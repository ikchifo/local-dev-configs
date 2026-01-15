---
name: uv-python-scripts
description: Use when creating standalone Python scripts, utility scripts, automation scripts, or data processing scripts that are not part of a larger package.
---

# UV Python Scripts

Create self-contained, executable Python scripts with inline dependencies.

## Script Format

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "httpx>=0.25.0",
#   "rich>=13.0.0",
# ]
# ///

import httpx
from rich import print

# Your code here
```

**First run downloads dependencies (cached afterward).**

## Requirements

1. **Shebang**: `#!/usr/bin/env -S uv run`
2. **Inline metadata**: `# /// script` block with `requires-python` and `dependencies`
3. **Self-contained**: No separate requirements.txt or venv needed

## Running Scripts

```bash
chmod +x script.py && ./script.py      # Direct execution
uv run script.py                        # Explicit uv
./script.py arg1 arg2                   # With arguments
```

## CLI Arguments Pattern

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.12"
# dependencies = ["typer>=0.9.0"]
# ///

import typer

def main(name: str, count: int = 1, verbose: bool = False):
    """Greet someone COUNT times."""
    for _ in range(count):
        msg = f"Hello, {name}!"
        if verbose:
            typer.echo(f"[verbose] {msg}")
        else:
            typer.echo(msg)

if __name__ == "__main__":
    typer.run(main)
```

## Error Handling Pattern

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.12"
# dependencies = ["httpx>=0.25.0"]
# ///

import sys
import httpx

def main() -> int:
    try:
        response = httpx.get("https://api.example.com", timeout=10.0)
        response.raise_for_status()
        print(response.json())
        return 0
    except httpx.HTTPError as e:
        print(f"HTTP error: {e}", file=sys.stderr)
        return 1
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        return 1

if __name__ == "__main__":
    sys.exit(main())
```

## Version Constraints

```python
# dependencies = [
#   "requests",           # Any version
#   "pandas>=2.0.0",      # Minimum version
#   "numpy>=1.24,<2.0",   # Version range
#   "rich==13.7.0",       # Exact version
# ]
```

## Private Indexes

```python
# /// script
# requires-python = ">=3.12"
# dependencies = ["private-package"]
#
# [[tool.uv.index]]
# url = "https://private.pypi.example.com/simple"
# ///
```

## When NOT to Use

- Files within a package managed by `pyproject.toml`
- Modules imported by other code
- Library/non-executable code
- Test files

## Reference

- [uv scripts guide](https://docs.astral.sh/uv/guides/scripts/)
- [PEP 723 - Inline script metadata](https://peps.python.org/pep-0723/)
