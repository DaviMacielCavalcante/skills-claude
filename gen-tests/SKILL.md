---
name: python-tests
description: >
  Generate Python tests for a project. Use this skill whenever the user asks to
  write tests, create test cases, add unit tests, integration tests, or any kind
  of automated testing for a Python project. Also trigger when the user mentions
  "coverage", "pytest", "test suite", or says "I need tests for X". Tests are
  always generated in a `tests/` directory at the project root, organized by
  type in subdirectories.
---

# Python Test Generator

Generates well-structured, idiomatic Python tests using `pytest`. Tests are
organized by type in subdirectories under `tests/` at the project root.

---

## Directory Structure

Always follow this structure — never deviate:

```
<project_root>/
└── tests/
    ├── __init__.py
    ├── unit/
    │   ├── __init__.py
    │   └── test_<module>.py
    ├── integration/
    │   ├── __init__.py
    │   └── test_<feature>.py
    └── e2e/
        ├── __init__.py
        └── test_<flow>.py
```

Create only the subdirectories that are relevant to the tests being generated.
Always include `__init__.py` in every directory.

---

## Test Types

| Type | Directory | Purpose |
|---|---|---|
| Unit | `tests/unit/` | Test individual functions/classes in isolation |
| Integration | `tests/integration/` | Test interaction between modules or external services |
| End-to-end | `tests/e2e/` | Test complete user flows |

When the user does not specify a type, default to **unit tests**.

---

## Step-by-Step Workflow

### 1. Identify what to test

Read the source files relevant to the user's request. Look for:
- Public functions and methods
- Edge cases (empty inputs, None, boundary values)
- Error paths (exceptions that should be raised)
- Side effects (file I/O, API calls, database writes)

### 2. Choose the test type

- Pure functions with no external dependencies → **unit**
- Functions that call APIs, databases, or other modules → **integration**
- Full user workflows (e.g., a Streamlit page loading and filtering) → **e2e**

### 3. Create the directory structure

```bash
mkdir -p tests/unit tests/integration tests/e2e
touch tests/__init__.py tests/unit/__init__.py tests/integration/__init__.py tests/e2e/__init__.py
```

Only create the directories needed for the tests being generated.

### 4. Write the tests

Follow these conventions:

```python
# tests/unit/test_loader.py

import pytest
from src.data.loader import load


class TestLoad:
    """Tests for the load() function."""

    def test_returns_dict_on_success(self, requests_mock):
        """load() should return a dict with 'projects' and 'categories' keys."""
        requests_mock.get(
            "https://raw.githubusercontent.com/.../projects.yaml",
            text="projects: []\ncategories: []"
        )
        result = load()
        assert isinstance(result, dict)
        assert "projects" in result
        assert "categories" in result

    def test_returns_none_on_http_error(self, requests_mock):
        """load() should return None when the request fails."""
        requests_mock.get(
            "https://raw.githubusercontent.com/.../projects.yaml",
            status_code=404
        )
        result = load()
        assert result is None

    def test_filters_resource_entries(self, requests_mock):
        """load() should exclude projects with resource: true."""
        yaml_content = """
projects:
  - name: real-lib
    github_id: owner/real-lib
  - name: resource-ref
    github_id: owner/resource-ref
    resource: true
categories: []
"""
        requests_mock.get(..., text=yaml_content)
        result = load()
        assert len(result["projects"]) == 1
        assert result["projects"][0]["name"] == "real-lib"
```

**Naming conventions:**
- File: `test_<module_name>.py`
- Class: `Test<FunctionOrClass>`
- Method: `test_<what_it_does>`

**Each test method must:**
- Have a docstring describing what it verifies
- Test exactly one behavior
- Use `pytest.raises` for expected exceptions
- Use `monkeypatch` or `pytest-mock` for mocking

### 5. Check for a `conftest.py`

If tests share fixtures (e.g., a mock HTTP session, a sample YAML payload),
create a `tests/conftest.py` with shared fixtures:

```python
# tests/conftest.py

import pytest


@pytest.fixture
def sample_projects_yaml():
    return {
        "projects": [
            {"name": "rich", "github_id": "Textualize/rich", "category": "cli-helpers"}
        ],
        "categories": [
            {"category": "cli-helpers", "title": "CLI Development"}
        ]
    }
```

### 6. Verify the test file is runnable

After writing, check for obvious issues:
- All imports resolve
- No undefined variables
- Fixtures are named correctly

---

## Mocking Guide

| Scenario | Tool |
|---|---|
| HTTP requests (`requests`) | `requests-mock` or `pytest-mock` + `monkeypatch` |
| Environment variables | `monkeypatch.setenv("KEY", "value")` |
| File system | `tmp_path` fixture (built-in to pytest) |
| Streamlit components | Skip — test the underlying logic, not the UI |
| diskcache | `monkeypatch` the `Cache` class or use `tmp_path` |

---

## What NOT to test

- Streamlit UI components directly (use integration tests for the logic behind them)
- Third-party library internals
- Configuration values that are constants

---

## Output

After generating the tests, tell the user:
1. Which files were created
2. How to run them: `uv run pytest tests/` or `pytest tests/`
3. Any dependencies they need to install (e.g., `pytest-mock`, `requests-mock`)
