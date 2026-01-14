---
name: python-dev-guidelines
description: Use when writing Python code, setting up Python projects, building APIs with FastAPI/Flask, data processing with Pandas/NumPy, async Python, or testing with pytest. Covers modern tooling (uv, ruff) and best practices.
---

# Python Development Guidelines

**Purpose:** Best practices and patterns for Python development including modern tooling, testing, and code quality.

**When to use:**
- Writing Python code (services, scripts, utilities)
- Setting up Python projects
- Implementing APIs with FastAPI/Flask
- Data processing with Pandas/NumPy
- Working with async Python

## Quick Reference

### Modern Python Tooling

**Use `uv` for dependency management:**
```bash
# Create new project
uv init my-project

# Add dependencies
uv add fastapi uvicorn

# Run scripts with inline dependencies
uv run --with requests script.py
```

**Project structure:**
```
project/
├── pyproject.toml          # Project config (PEP 621)
├── uv.lock                 # Lock file
├── src/
│   └── myapp/
│       ├── __init__.py
│       ├── main.py
│       └── models/
├── tests/
│   ├── __init__.py
│   └── test_main.py
└── README.md
```

### Code Quality Standards

**Type hints (required):**
```python
from typing import Optional, List, Dict
from pathlib import Path

def process_data(
    file_path: Path,
    limit: Optional[int] = None
) -> List[Dict[str, str]]:
    """Process data from file with optional limit."""
    pass
```

**Use Pydantic for validation:**
```python
from pydantic import BaseModel, Field, validator

class User(BaseModel):
    id: int
    name: str = Field(..., min_length=1)
    email: str
    age: Optional[int] = None

    @validator('email')
    def validate_email(cls, v):
        if '@' not in v:
            raise ValueError('Invalid email')
        return v
```

**Error handling:**
```python
from typing import Union
from dataclasses import dataclass

@dataclass
class Result:
    """Result type for better error handling."""
    success: bool
    data: any = None
    error: Optional[str] = None

def safe_operation() -> Result:
    try:
        data = risky_operation()
        return Result(success=True, data=data)
    except Exception as e:
        return Result(success=False, error=str(e))
```

### Testing Standards

**Use pytest:**
```python
import pytest
from myapp.service import process_data

def test_process_data_success():
    result = process_data(Path("test.csv"))
    assert result.success
    assert len(result.data) > 0

def test_process_data_invalid_file():
    with pytest.raises(FileNotFoundError):
        process_data(Path("nonexistent.csv"))

@pytest.fixture
def sample_data():
    return {"id": 1, "name": "test"}
```

**Async testing:**
```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_api_endpoint():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get("/users/1")
        assert response.status_code == 200
```

### FastAPI Patterns

**Layered architecture:**
```python
# models.py
from pydantic import BaseModel

class UserCreate(BaseModel):
    name: str
    email: str

class User(UserCreate):
    id: int

# service.py
from typing import Optional

class UserService:
    async def create_user(self, user: UserCreate) -> User:
        # Business logic here
        pass

    async def get_user(self, user_id: int) -> Optional[User]:
        pass

# routes.py
from fastapi import APIRouter, Depends, HTTPException

router = APIRouter(prefix="/users", tags=["users"])
user_service = UserService()

@router.post("/", response_model=User)
async def create_user(user: UserCreate):
    return await user_service.create_user(user)

@router.get("/{user_id}", response_model=User)
async def get_user(user_id: int):
    user = await user_service.get_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### Data Processing Patterns

**Pandas best practices:**
```python
import pandas as pd
from pathlib import Path

def load_and_process(file_path: Path) -> pd.DataFrame:
    """Load CSV and process with type safety."""
    df = pd.read_csv(
        file_path,
        dtype={
            'id': 'int64',
            'name': 'string',
            'value': 'float64'
        },
        parse_dates=['created_at']
    )

    # Use method chaining
    return (df
        .dropna(subset=['id'])
        .assign(normalized_value=lambda x: x['value'] / x['value'].max())
        .sort_values('created_at')
    )
```

### Async Patterns

**Concurrent operations:**
```python
import asyncio
from typing import List

async def fetch_user(user_id: int) -> User:
    # async operation
    pass

async def fetch_multiple_users(user_ids: List[int]) -> List[User]:
    """Fetch users concurrently."""
    tasks = [fetch_user(uid) for uid in user_ids]
    return await asyncio.gather(*tasks)
```

## Key Principles

1. **Type everything** - Use type hints for all function signatures
2. **Validate inputs** - Use Pydantic models for data validation
3. **Test thoroughly** - Write tests before or alongside code
4. **Use modern tools** - uv, ruff, mypy for productivity
5. **Async when I/O** - Use async/await for I/O-bound operations
6. **Explicit errors** - Return Result types or raise specific exceptions
7. **Document with types** - Type hints + docstrings = clear intent

## Common Commands

```bash
# Run tests
pytest

# Type checking
mypy src/

# Linting & formatting
ruff check .
ruff format .

# Run with uv
uv run python -m myapp.main

# Add dev dependencies
uv add --dev pytest mypy ruff
```

## Resources

For more details on specific topics, use the Skill tool to access:
- `python-testing-patterns` - Advanced testing strategies
- `python-async-patterns` - Deep dive into async/await
- `python-api-development` - FastAPI advanced patterns
