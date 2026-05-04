---
name: gen-docs
description: Use when generating or updating documentation. Creates READMEs, adds docstrings to Python functions, generates API docs, and writes CONTRIBUTING guides. Matches the author's style with Star Wars references and lighthearted tone.
---

# Generate Documentation

Create or update documentation for the specified code or project.

## Detect What's Needed

1. Read the target (file, module, or full project)
2. Determine what's missing:
   - No README.md → generate full README
   - Functions without docstrings → add docstrings
   - No CONTRIBUTING.md → generate contributing guide
   - Outdated README → update based on current code

## README Template (for projects)

Follow this structure. Tone: professional but with personality. Include a Star Wars reference near the end.

```markdown
# {Project Name} {emoji}

{One-line description of what it does and why it exists.}

## Overview

{2-3 sentences explaining the problem this solves and the approach taken.}

## Tech Stack

{List as a simple inline list, not a badge wall.}

## Getting Started

### Prerequisites

{What needs to be installed first.}

### Installation

\`\`\`bash
{Exact commands to get running from zero.}
\`\`\`

### Usage

\`\`\`python
{Minimal working example.}
\`\`\`

## Architecture

{Brief explanation + ASCII diagram if helpful.}

## Running Tests

\`\`\`bash
pytest -v
\`\`\`

## Contributing

{Brief instructions or link to CONTRIBUTING.md.}

## License

MIT License — see [LICENSE](LICENSE) for details.

---

**Author:** Davi Cavalcante — [davicc@outlook.com.br](mailto:davicc@outlook.com.br)

*{Star Wars quote that fits the project theme.}*
```

## README Template (for PyO3/maturin libraries)

Same as above but include:
- Installation section with `pip install {name}` AND `maturin develop`
- Benchmarks section comparing Python vs Rust performance
- "Why Rust?" section explaining the performance rationale
- Badge for PyPI version if published

## Docstring Style

Use NumPy-style docstrings (NEVER Google-style):

```python
def process_batch(items: list[str], batch_size: int = 150) -> list[str]:
    """Process a list of items in batches.

    Splits the input into chunks of ``batch_size`` and processes each
    chunk in parallel using rayon on the Rust side.

    Parameters
    ----------
    items : list[str]
        Raw items to process.
    batch_size : int, optional
        Number of items per batch, by default 150.

    Returns
    -------
    list[str]
        Processed items as a flat list.

    Raises
    ------
    ValueError
        If ``batch_size`` is less than 1.

    Examples
    --------
    >>> process_batch(["hello", "world"])
    ["HELLO", "WORLD"]
    """
```

## Rules

1. **Language**: English for open-source projects, Portuguese for internal/academic projects
2. **No badge walls**: Use badges sparingly (status, Python version, license — max 3)
3. **Working examples**: Every code block in the README must actually work
4. **Keep it scannable**: Someone should get the gist in 30 seconds
5. **Update, don't rewrite**: When updating existing docs, preserve the author's voice and structure

## After Generating

1. Read the output and verify code examples are correct
2. Check all links resolve
3. Verify the project structure described matches the actual structure
