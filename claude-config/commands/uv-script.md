---
description: Create a Python script using uv with inline dependencies
---

Create a self-contained Python script using the `uv` inline script dependency format as documented at https://docs.astral.sh/uv/guides/scripts/#declaring-script-dependencies

ALWAYS use this format:

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "package-name",
#   "another-package>=1.0.0",
# ]
# ///

# Your code here
```

Requirements:
1. Start with `#!/usr/bin/env -S uv run` shebang
2. Include inline metadata block with `requires-python` and `dependencies`
3. Make the script executable and self-contained
4. No separate requirements.txt or virtual environment needed

After creating the script, make it executable with `chmod +x script.py`
