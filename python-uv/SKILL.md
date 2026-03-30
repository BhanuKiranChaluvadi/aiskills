---
name: python-uv
description: Always run Python code in a uv-managed environment and scaffold new Python projects with a consistent workflow
---

# Python + uv skill

This skill enforces one rule above all others:

**All Python-related work must be executed through `uv`.**

The system Python interpreter must not be used directly for project work unless the user explicitly asks for it.

## Core policy

- Never use bare `python`, `pip`, `pytest`, `ruff`, `mypy`, or similar tools directly from the host system.
- Always use `uv run` to execute Python code and Python-based tools.
- Always use `uv sync` to install or synchronize dependencies from `pyproject.toml` and `uv.lock`.
- Prefer `pyproject.toml` as the single source of truth for project metadata and dependencies.
- Prefer `uv add` to add dependencies.
- Prefer `uv remove` to remove dependencies.
- Do not recommend `pip install` globally.
- Do not manually create or activate a virtual environment with `python -m venv` unless the user explicitly requests that workflow.
- Any solution that uses `python`, `pip`, `venv`, or `virtualenv` directly is invalid unless the user explicitly requested it.

## Hard constraint

If a Python command would run outside the uv-managed environment, stop and switch to the appropriate `uv` command.

Valid examples:

```bash
uv run python main.py
uv run python -m package.module
uv run pytest
uv run ruff check .
uv run ruff format .
```

Invalid examples:

```bash
python main.py
pip install requests
pytest
ruff check .
python -m venv .venv
source .venv/bin/activate
```

## Preconditions

Before doing Python work, verify that `uv` is available:

```bash
command -v uv >/dev/null 2>&1 || { echo "uv is required but not installed"; exit 1; }
```

If `uv` is not installed, stop and tell the user that `uv` must be installed first.

## Existing project workflow

When working in an existing Python project:

1. Check whether `pyproject.toml` exists.
2. If it exists, run `uv sync`.
3. Execute all Python-related commands through `uv run`.
4. If tooling is missing, add it via `uv add --dev`.
5. Do not fall back to system Python.

### Standard bootstrap for an existing project

```bash
command -v uv >/dev/null 2>&1 || { echo "uv is required but not installed"; exit 1; }
[ -f pyproject.toml ] || { echo "pyproject.toml is missing"; exit 1; }
uv sync
```

## New project scaffolding

When the user asks to create a new Python project, scaffold it with `uv`.

### Default project layout

Use this layout unless the user asked for a different structure:

```text
project-name/
├── pyproject.toml
├── README.md
├── .gitignore
├── src/
│   └── project_name/
│       ├── __init__.py
│       └── main.py
├── tests/
│   └── test_basic.py
```

### Create a new application project

```bash
uv init --package project-name
cd project-name
uv add --dev pytest ruff
mkdir -p tests
cat > tests/test_basic.py <<'EOF'
def test_placeholder():
    assert True
EOF
```

### Create a minimal script-based project

```bash
mkdir project-name
cd project-name
uv init --bare
uv add --dev pytest ruff
mkdir -p src
cat > src/main.py <<'EOF'
def main() -> None:
    print("Hello from uv")

if __name__ == "__main__":
    main()
EOF
mkdir -p tests
cat > tests/test_basic.py <<'EOF'
def test_placeholder():
    assert True
EOF
```

### Recommended package entry point

For packaged projects, prefer a simple `main()` entry point:

```python
def main() -> None:
    print("Hello from the project")

if __name__ == "__main__":
    main()
```

## Dependency management

### Add a runtime dependency

```bash
uv add requests
```

### Add development dependencies

```bash
uv add --dev pytest ruff mypy
```

### Remove a dependency

```bash
uv remove requests
```

### Synchronize dependencies

```bash
uv sync
```

## Command patterns

### Run a script

```bash
uv run python script.py
```

### Run a package module

```bash
uv run python -m package.module
```

### Run tests

```bash
uv run pytest
```

### Run a single test file

```bash
uv run pytest tests/test_basic.py
```

### Run linting

```bash
uv run ruff check .
```

### Run formatting

```bash
uv run ruff format .
```

### Run type checks

```bash
uv run mypy src
```

## Quality defaults

Unless the user asks otherwise:

- Use `src/` layout for importable packages.
- Use `pytest` for tests.
- Use `ruff` for linting and formatting.
- Keep code simple and explicit.
- Prefer standard library solutions before adding dependencies.
- Add type hints for public functions.
- Keep entry points small and move logic into reusable functions.

## Behavior rules for the assistant

When handling a Python task:

1. Assume `uv` is the required environment manager.
2. Prefer editing `pyproject.toml` over ad hoc dependency installation.
3. Prefer `uv add` and `uv sync` over any `pip`-based command.
4. Use `uv run` for every Python execution step.
5. If the project does not exist and the user wants a new one, scaffold it with `uv init`.
6. If tests or lint tools are needed but not present, add them as dev dependencies.
7. Never silently bypass uv.
8. Never suggest a global installation-based workflow unless the user explicitly asks for one.

## Failure handling

- If `uv` is missing, stop and report that `uv` must be installed.
- If `pyproject.toml` is missing in an existing project, report that the project is not initialized for uv and suggest scaffolding or initialization with `uv init`.
- If a command fails because a tool is missing, add it with `uv add --dev` and retry through `uv run`.
- Do not fall back to `pip`, `python -m venv`, or the system interpreter.

## Example end-to-end workflow

```bash
uv init --package demo_app
cd demo_app
uv add --dev pytest ruff
uv sync
uv run ruff check .
uv run pytest
uv run python -m demo_app.main
```
