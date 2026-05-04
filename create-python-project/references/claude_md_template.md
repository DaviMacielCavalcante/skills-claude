# CLAUDE.md template

Use this as the starting point for the project's `CLAUDE.md`. Replace every `{{PLACEHOLDER}}` with real, project-specific content. Sections that don't apply to the project type (e.g., the CLI entry point section for a library) should be removed rather than left empty.

The goal of this file is to give future Claude sessions the context they need to be useful in this repo from message one — without the user having to re-explain conventions every time.

---

```markdown
# {{PROJECT_NAME}}

{{PROJECT_DESCRIPTION}}

## Project layout

{{LAYOUT_NOTES}}

<!-- For library: src/<name>/__init__.py is the package root; tests live in tests/. -->
<!-- For CLI: src/<name>/__init__.py exports the entry point; the CLI command is registered in pyproject.toml under [project.scripts]. -->
<!-- For app: main.py (or app.py / pipeline.py) is the entry point at the project root. -->

## Tooling

This project uses **uv** as the package and project manager. Always use uv for dependency operations — never edit `pyproject.toml` dependencies by hand, and never invoke `pip` directly.

Common commands:

- `uv sync` — install/update all dependencies from the lockfile
- `uv add <package>` — add a runtime dependency
- `uv add --dev <package>` — add a development dependency
- `uv remove <package>` — remove a dependency
- `uv run <command>` — run a command inside the project's virtualenv
- `uv run python <script.py>` — run a Python script
- `uv run pytest` — run the test suite
- `uv run ruff check .` — lint
- `uv run ruff format .` — format
- `uv run mypy .` — type-check

## Key dependencies

{{KEY_DEPS}}

<!-- Brief 1-line note about what each significant runtime dep is for, e.g.: -->
<!-- - polars: dataframe operations (preferred over pandas for new code) -->
<!-- - loguru: structured logging -->
<!-- - pydantic: data validation and config schemas -->

## Coding conventions

- **Type hints are required** on all function signatures (parameters and return types). `mypy` is configured strictly; new code should pass without `# type: ignore` unless absolutely necessary.
- **Docstrings use NumPy style** (not Google or Sphinx). Module-level docstrings are encouraged for non-trivial modules. Function docstrings should include `Parameters`, `Returns`, and (when relevant) `Raises` and `Examples` sections.
- **Line length is 100** characters (configured in ruff).
- **Imports are sorted by ruff** (isort-compatible). Don't manually re-order them; run `uv run ruff check --fix .` instead.
- **Prefer composition over inheritance** for non-trivial code. Prefer pure functions for transformations; reserve classes for things with real state or protocol-driven design.
- **Naming**: snake_case for functions and variables, PascalCase for classes, UPPER_SNAKE for constants. Avoid single-letter names except in tight loops or short comprehensions.

### Docstring example (NumPy style)

```python
def transform_records(records: list[dict[str, Any]], strict: bool = False) -> pl.DataFrame:
    """Convert a list of raw records into a typed Polars DataFrame.

    Parameters
    ----------
    records : list of dict
        Input records. Each dict should have keys matching the schema.
    strict : bool, default False
        If True, raise on schema mismatch. If False, coerce where possible.

    Returns
    -------
    pl.DataFrame
        DataFrame with the canonical schema applied.

    Raises
    ------
    ValueError
        If `strict=True` and a record fails schema validation.
    """
```

## Testing

- Tests live in `tests/` at the project root.
- Test files are named `test_*.py`; test functions are named `test_*`.
- Run the full suite with `uv run pytest`. Add `-x` to stop on first failure, `-k <pattern>` to filter, `-vv` for verbose output.
- Prefer parametrized tests (`@pytest.mark.parametrize`) over loops. Prefer fixtures over setup/teardown.
- Aim for tests that exercise behavior, not implementation details. Don't mock things that are cheap to run for real.

## Adding a new dependency

1. `uv add <package>` (runtime) or `uv add --dev <package>` (dev tooling, test deps).
2. Mention the addition in the PR / commit message — what it's for, why it was chosen.
3. Update the **Key dependencies** section above if it's a significant runtime dep.

## What to avoid

- Don't introduce alternative formatters/linters (black, flake8, pylint, isort) — ruff covers all of these and adding more creates conflicts.
- Don't pin exact dep versions in `pyproject.toml` unless there's a known incompatibility. uv's lockfile handles reproducibility.
- Don't commit `.venv/`, `__pycache__/`, or any of the cache directories — they're in `.gitignore`.
- Don't write to module-level state at import time. Prefer functions or classes that own their state.

## Project-specific notes

{{PROJECT_SPECIFIC_NOTES}}

<!-- Anything unique to this project: data sources, external services, deployment targets, -->
<!-- naming conventions for tables/columns, medallion architecture layers, etc. -->
<!-- If there's nothing project-specific yet, leave this as: "(none yet — add as the project develops)" -->
```

---

## Notes for filling in the template

- `{{PROJECT_NAME}}` — same as the directory name.
- `{{PROJECT_DESCRIPTION}}` — 1–2 sentence description from the user. If the user didn't give one, ask before writing the file.
- `{{LAYOUT_NOTES}}` — describe the layout for the chosen project type. Use the inline HTML comments in the template as reference and replace them with prose.
- `{{KEY_DEPS}}` — bullet list of significant runtime deps with one-line purpose. Skip if no runtime deps.
- `{{PROJECT_SPECIFIC_NOTES}}` — likely empty initially; leave a stub like `(none yet — add as the project develops)`.

Remove any section that doesn't apply (e.g., a library project might not need an entry point note). The goal is a tight, useful file — not a comprehensive checklist.
