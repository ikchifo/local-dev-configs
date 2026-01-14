---
name: uv-python-scripts
description: Use when creating standalone Python scripts, utility scripts, automation scripts, or data processing scripts. Creates self-contained, executable Python files with inline dependencies using uv.
---

# UV Python Scripts

**Purpose:** Create self-contained, executable Python scripts with inline dependencies using `uv`.

**When to use:**
- Creating standalone Python scripts
- Writing utility or automation scripts
- Data processing scripts
- Any Python file that isn't part of a larger package/project

---

## Script Format

Every standalone Python script must follow this format:

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "requests",
#   "pandas>=2.0.0",
# ]
# ///

import requests
import pandas as pd

# Your code here
```

---

## Requirements

1. **Shebang**: Always start with `#!/usr/bin/env -S uv run`
2. **Inline metadata**: Include the `# /// script` block with:
   - `requires-python`: Specify minimum Python version
   - `dependencies`: List all required packages with optional version constraints
3. **Self-contained**: The script must be executable without separate requirements.txt or virtual environment setup

---

## Quick Examples

### Minimal Script (no dependencies)

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.12"
# dependencies = []
# ///

print("Hello, world!")
```

### Script with Dependencies

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

response = httpx.get("https://api.github.com")
print(response.json())
```

### Data Processing Script

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "pandas>=2.0.0",
#   "pyarrow",
# ]
# ///

import pandas as pd
from pathlib import Path

def process_csv(input_path: Path, output_path: Path) -> None:
    df = pd.read_csv(input_path)
    # Process data...
    df.to_parquet(output_path)

if __name__ == "__main__":
    import sys
    process_csv(Path(sys.argv[1]), Path(sys.argv[2]))
```

---

## Usage

After creating a script (e.g., `script.py`), users can run it directly:

```bash
# Make executable and run
chmod +x script.py
./script.py

# Or explicitly with uv
uv run script.py

# With arguments
./script.py arg1 arg2
```

---

## Benefits

- **No manual setup**: No need to create virtual environments
- **Self-documenting**: Dependencies declared inline
- **Portable**: Works on any machine with `uv` installed
- **Reproducible**: Pinned versions ensure consistent execution
- **Fast**: uv caches dependencies aggressively

---

## When NOT to Apply

Do **NOT** use this pattern for:
- Files within a package managed by `pyproject.toml`
- Modules that are imported by other code
- Library code (non-executable)
- Test files

For these cases, use standard project structure with `pyproject.toml`.

---

## Version Constraints

Use standard PEP 440 version specifiers:

```python
# dependencies = [
#   "requests",           # Any version
#   "pandas>=2.0.0",      # Minimum version
#   "numpy>=1.24,<2.0",   # Version range
#   "rich==13.7.0",       # Exact version
# ]
```

---

## Adding Extra Indexes

For packages from private indexes:

```python
# /// script
# requires-python = ">=3.12"
# dependencies = ["private-package"]
#
# [[tool.uv.index]]
# url = "https://private.pypi.example.com/simple"
# ///
```

---

## Reference

- [uv scripts guide](https://docs.astral.sh/uv/guides/scripts/)
- [PEP 723 – Inline script metadata](https://peps.python.org/pep-0723/)
