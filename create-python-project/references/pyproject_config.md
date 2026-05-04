# pyproject.toml configuration snippets

Append these sections to the project's `pyproject.toml` after `uv init` runs. Adjust `target-version` and `python_version` to match the user's chosen Python version (e.g., `py312` / `"3.12"` if they picked 3.12 instead of 3.14).

## Ruff configuration

```toml
[tool.ruff]
line-length = 100
target-version = "py314"

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # pyflakes
    "I",    # isort
    "N",    # pep8-naming
    "UP",   # pyupgrade
    "B",    # flake8-bugbear
    "SIM",  # flake8-simplify
    "RUF",  # ruff-specific rules
    "D",    # pydocstyle (docstring conventions)
]
ignore = [
    "D203",  # one-blank-line-before-class (conflicts with D211)
    "D213",  # multi-line-summary-second-line (conflicts with D212)
]

[tool.ruff.lint.pydocstyle]
convention = "numpy"

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = ["D"]      # don't require docstrings in tests
"__init__.py" = ["D104"]      # don't require module docstring in __init__.py
```

## Mypy configuration

```toml
[tool.mypy]
python_version = "3.14"
strict = true
warn_unreachable = true
warn_unused_ignores = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
no_implicit_optional = true
show_error_codes = true
pretty = true
```

If the project uses libraries without type stubs (common in data engineering — e.g., older versions of some Spark or Databricks utilities), the user may need to add an override later:

```toml
[[tool.mypy.overrides]]
module = ["some_untyped_lib.*"]
ignore_missing_imports = true
```

Don't pre-add this; let it appear naturally when needed.

## Pytest configuration

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = "-ra --strict-markers"
```

## .gitignore

The standard Python `.gitignore`. `uv init` may already create one — if so, check whether these entries are present and append the missing ones rather than overwriting.

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python

# Virtual environments
.venv/
venv/
env/
ENV/

# Distribution / packaging
build/
dist/
*.egg-info/
*.egg
.eggs/

# Tool caches
.mypy_cache/
.ruff_cache/
.pytest_cache/
.tox/
.nox/
.coverage
.coverage.*
htmlcov/

# Jupyter
.ipynb_checkpoints/

# Environment files
.env
.env.local
.env.*.local

# Editor / IDE
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store

# uv
# (uv.lock SHOULD be committed for apps; for libraries it's a judgment call.
# Default: keep it committed — easier to debug reproducibility later.)
```

## Notes

- For **libraries**, some teams prefer not to commit `uv.lock` because consumers will resolve their own. This is debatable; the safer default is to commit it and ignore for distribution. Don't add `uv.lock` to `.gitignore` unless the user asks.
- For **apps** (dashboards, pipelines, CLIs), always commit `uv.lock` — reproducibility matters.
- The ruff `D` rule family is enabled to enforce NumPy-style docstrings. If the user finds it too noisy on early-stage code, the `per-file-ignores` section can be expanded — but don't disable it project-wide; it's high-signal for the kind of code the user writes.
