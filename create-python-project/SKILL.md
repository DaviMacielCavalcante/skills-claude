---
name: create-python-project
description: Use this skill whenever the user wants to create, scaffold, set up, initialize, or bootstrap a new Python project. Triggers include "create a Python project", "start a new Python project", "new Python repo", "scaffold a Python package", "set up a Python CLI", "build a new Streamlit app", "criar projeto Python", "novo projeto Python", "iniciar projeto Python", "começar um projeto Python do zero", "montar um pacote Python", or any request to generate the initial structure of a Python codebase (pyproject.toml, virtualenv, dependency setup). Trigger even when the user does not say "uv" — uv is the default tool. Trigger even if the user only asks for a CLAUDE.md for a Python project, since the standard scaffold includes one. The skill handles project-type detection, dependency elicitation, tooling configuration (ruff, mypy, pytest), and writes a CLAUDE.md tailored to the project.
---

# Create Python Project

Scaffold a new Python project using `uv`, with sensible tooling defaults (ruff, mypy, pytest), and a `CLAUDE.md` that primes future Claude sessions on the project's conventions. The skill is interactive: it briefly elicits the user's choices, then produces a working project that can be `cd`'d into and run immediately.

## Why this skill exists

Starting a Python project well is mostly mechanical (init, deps, lint config, gitignore) but easy to do inconsistently. This skill removes that friction and — crucially — drops a `CLAUDE.md` into the root so every future Claude session in the repo knows the conventions without being re-told. The CLAUDE.md is the most valuable artifact here: a small upfront investment that pays back every interaction afterward.

## Pre-flight check

Before scaffolding anything, verify `uv` is available:

```bash
uv --version
```

If it isn't, tell the user how to install it (`curl -LsSf https://astral.sh/uv/install.sh | sh` on Linux/macOS, or `powershell -c "irm https://astral.sh/uv/install.ps1 | iex"` on Windows) and stop. Don't fall back to pip/poetry/pdm — this skill is uv-specific by design, because uv is faster, has built-in Python version management, and the user prefers it.

## Workflow

The flow is: **elicit → init → add deps → configure → write CLAUDE.md → sync → report**. Each step is described below. Match the user's language (Portuguese, English, etc.) throughout.

### 1. Elicit the project info

Detect what the user already told you in the request (project name, type, dependencies they mentioned) and skip those questions. Ask only what's missing. Use `ask_user_input_v0` if available — it's tappable and much faster than typing on mobile. Otherwise, ask in plain text in a single grouped message.

The questions, in order:

1. **Project name** — directory name. If user gave one, confirm rather than re-ask.
2. **Project type** — one of:
   - `library` — importable Python package (uv init --lib, src/ layout)
   - `cli` — command-line tool with entry point (uv init --package)
   - `app` — generic application: dashboard, data pipeline, web API, script, notebook (uv init, flat layout)
3. **App flavor** *(only if type = app)* — drives dependency suggestions:
   - `dashboard` (Streamlit, Plotly/Altair)
   - `pipeline` (Polars or Pandas, loguru, pydantic)
   - `web-api` (FastAPI, uvicorn, pydantic)
   - `script` (minimal, no suggested deps)
   - `notebook` (Jupyter or Marimo)
4. **Runtime dependencies** — show suggestions based on type/flavor (see "Dependency suggestions" below). Always let the user override or add their own. "None" is a valid answer.
5. **Dev dependencies** — propose `ruff`, `mypy`, `pytest` as defaults; ask if the user wants to keep all three, drop any, or add more (e.g., `pytest-cov`, `hypothesis`, `mkdocs`). Don't force them.
6. **Python version** — default to the latest stable (currently 3.14 unless the user has constraints). Confirm if you're unsure.

Keep elicitation tight. Don't ask for license, author, or git remote — those aren't needed for a working scaffold and the user can fill them in later.

#### Dependency suggestions

Use these as starting points for the runtime deps prompt. They are suggestions — the user picks.

| Type / flavor | Suggested runtime deps |
|---|---|
| library | (often none — leave empty unless user names some) |
| cli | `typer`, `rich`, `loguru` |
| app + dashboard | `streamlit`, `plotly`, `loguru` |
| app + pipeline | `polars`, `loguru`, `pydantic` |
| app + web-api | `fastapi`, `uvicorn[standard]`, `pydantic`, `loguru` |
| app + script | `loguru` (suggest; user may opt out) |
| app + notebook | `jupyter` |

`loguru` shows up in most rows because the user uses it as the default logging library across personal and work projects — it's nearly always wanted when there's any kind of structured output. Don't force it on libraries (where users should use `logging` and let the consumer configure it) or notebooks (where logging usually isn't the point).

### 2. Initialize the project with uv

Map the project type to the right `uv init` flag:

```bash
# library
uv init --lib --python 3.14 <name>

# cli
uv init --package --python 3.14 <name>

# app (any flavor)
uv init --python 3.14 <name>
```

`cd` into the new directory before adding dependencies.

### 3. Add dependencies

Add runtime deps and dev deps separately so they land in the right `pyproject.toml` groups:

```bash
uv add <runtime-deps>
uv add --dev <dev-deps>
```

If the user said "no runtime deps", skip the first line. Always make sure the dev deps step runs (assuming the user kept any) — having lint/type/test tools available is more important than people often realize when starting out.

### 4. Configure tooling

Append the ruff and mypy sections to `pyproject.toml`. The exact snippets are in `references/pyproject_config.md` — read that file and copy the sections, adjusting `target-version` / `python_version` to match the user's choice. The defaults reflect the user's preferences: NumPy-style docstrings, line length 100, type hints required, strict mypy.

Then create `.gitignore` with the standard Python entries (also in `references/pyproject_config.md`). If `uv init` already created one, append the missing entries rather than overwriting.

### 5. Write CLAUDE.md

This is the most important artifact. Read `references/claude_md_template.md` and fill it in. The template has placeholders like `{{PROJECT_NAME}}`, `{{PROJECT_DESCRIPTION}}`, `{{LAYOUT_NOTES}}`, `{{KEY_DEPS}}` — replace each with project-specific content. Do not paste the template verbatim; the placeholders need real values.

If the user gave you a 1–2 sentence project description in step 1, use it. If not, ask one short question now: "Em uma frase, do que se trata o projeto?" (or English equivalent).

Place `CLAUDE.md` at the project root (next to `pyproject.toml`).

### 6. Sync and verify

Run `uv sync` to install everything and verify the lockfile resolves:

```bash
uv sync
```

If it fails, debug it (usually a typo in a package name or a Python version mismatch) before declaring success. Then run a quick sanity check:

```bash
uv run ruff check .
uv run mypy .
```

These should both pass on a fresh project. If they don't, fix the config — don't ship a broken scaffold.

### 7. Report to the user

Summarize concisely what was created. Keep it to ~5 lines:

- Project name and path
- Layout type (library / CLI / app)
- Runtime + dev deps installed
- Where the entry point is (e.g., `src/<name>/__init__.py`, `main.py`, or the CLI command)
- The 2–3 most likely next commands (e.g., `cd <name> && uv run <name>`, or `uv run streamlit run main.py`)

Mention CLAUDE.md was created and that it'll guide future Claude sessions in the repo.

## Notes and edge cases

- **Don't over-scaffold.** No CI configs, Docker, GitHub Actions, pre-commit hooks, or documentation site unless the user explicitly asks. Each of those is a separate decision and adding them by default creates more friction (broken CI on a fresh repo, etc.) than value.
- **If the user wants pip / poetry / pdm**, gently note that this skill is uv-specific and ask if they want to proceed with uv anyway, or skip the skill and set up the alternative manually. Don't silently switch.
- **Existing directories**: if the project name matches an existing non-empty directory, stop and ask before overwriting. `uv init` will refuse to clobber a non-empty dir but it's nicer to catch this with a clear message.
- **Multiple uv init runs**: if something goes wrong mid-flow and the user wants to retry, `rm -rf <name>` first; don't try to repair a half-initialized directory.
- **Language**: respond and elicit in the user's language. If they wrote in Portuguese, the questions, the CLAUDE.md description prompt, and the final summary should all be in Portuguese. The CLAUDE.md template itself is in English (since it's instructions to Claude) but project-specific filled-in parts can be in the user's language.

## Reference files

- `references/claude_md_template.md` — template for the CLAUDE.md, with placeholders. Read and fill in for each project.
- `references/pyproject_config.md` — ruff, mypy, and gitignore snippets. Copy and adjust as needed.
