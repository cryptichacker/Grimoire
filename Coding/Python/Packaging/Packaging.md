---
tags: [coding, python, packaging, poetry, uv, pip]
type: cheatsheet
source: compiled reference (Python packaging — pip/venv/Poetry/uv)
last-verified: 2026-09-03
---

# Packaging

## Up
- [[Python]]

How to manage environments, dependencies, and distributable packages in Python — `venv` + `pip`, the modern **`pyproject.toml`** standard, and the two popular managers **Poetry** and **uv**.

---

## Virtual Environments + pip (stdlib baseline)

```bash
python -m venv .venv                 # create isolated env
source .venv/bin/activate            # activate (Linux/macOS); .venv\Scripts\activate on Windows
deactivate

pip install requests                 # install
pip install "requests>=2.31,<3"      # version constraint
pip install -e .                     # editable install of local project
pip install -r requirements.txt      # from a file
pip freeze > requirements.txt        # pin exact versions (flat, no dep graph)
pip list / pip show requests / pip uninstall requests
python -m pip install --upgrade pip
```

`requirements.txt` is simple but doesn't separate direct vs transitive deps or lock a resolved graph — hence the tools below.

---

## pyproject.toml (the modern standard, PEP 621)

The single config file for build metadata, dependencies, and tool settings:

```toml
[project]
name = "mypkg"
version = "0.1.0"
description = "..."
requires-python = ">=3.11"
dependencies = ["requests>=2.31", "pydantic>=2"]

[project.optional-dependencies]
dev = ["pytest", "ruff", "mypy"]

[project.scripts]
mycli = "mypkg.cli:main"             # console entry point → `mycli`

[build-system]
requires = ["hatchling"]             # or setuptools, flit, poetry-core
build-backend = "hatchling.build"

[tool.ruff]                          # tool config lives here too
line-length = 100
```

---

## Poetry (dependency manager + packaging)

```bash
pipx install poetry
poetry new mypkg          # or: poetry init  (in existing dir)
poetry add requests       # add dep (updates pyproject + poetry.lock)
poetry add --group dev pytest ruff
poetry remove requests
poetry install            # install from lock (creates/uses a venv)
poetry update             # re-resolve + update lock
poetry run pytest         # run inside the env
poetry shell              # spawn a subshell in the env
poetry build              # build sdist + wheel → dist/
poetry publish            # upload to PyPI (poetry publish -r testpypi)
poetry lock               # regenerate the lock file
```

Poetry resolves a full dependency graph and writes a **`poetry.lock`** for reproducible installs.

---

## uv (fast, all-in-one — Rust)

`uv` is a very fast drop-in for pip/venv/pip-tools and a project/Python-version manager.

```bash
# install: curl -LsSf https://astral.sh/uv/install.sh | sh
uv init mypkg                 # new project (pyproject + .venv)
uv add requests               # add dep → updates pyproject + uv.lock, installs
uv add --dev pytest ruff
uv remove requests
uv sync                       # install exactly per uv.lock (reproducible)
uv lock                       # resolve + write uv.lock
uv run pytest                 # run a command in the project env
uv run python script.py

# pip-compatible interface + Python management
uv pip install requests       # drop-in pip
uv venv                        # create a venv fast
uv python install 3.12         # install a Python version
uv tool install ruff           # install a CLI tool globally (like pipx)
```

uv is dramatically faster than pip/Poetry and increasingly the default choice for new projects.

---

## Building & Publishing (raw toolchain)

```bash
pip install build twine
python -m build               # → dist/*.whl and *.tar.gz (sdist)
twine check dist/*
twine upload dist/*           # PyPI ; --repository testpypi to test first
```

- **wheel** (`.whl`) = pre-built, fast install; **sdist** (`.tar.gz`) = source.
- Store PyPI tokens in `~/.pypirc` or use Trusted Publishing (OIDC) in CI.

---

## Tool Comparison

| Tool | Role | Lock file |
|---|---|---|
| **pip + venv** | Baseline install + isolation | `requirements.txt` (flat) |
| **pip-tools** | Compile pinned reqs from `.in` | `requirements.txt` |
| **Poetry** | Deps + build + publish, resolver | `poetry.lock` |
| **uv** | Fast deps + venv + Python + tools | `uv.lock` |
| **pipx** | Install CLI apps in isolated envs | — |
| **conda** | Cross-language envs (data/science) | `environment.yml` |

---

## Tips
- **Always use a virtual environment** — never `pip install` into system Python.
- Commit the **lock file** (`poetry.lock`/`uv.lock`) for reproducible builds; commit `pyproject.toml`, not `.venv/`.
- Prefer **`pyproject.toml`** over `setup.py` for new projects; pick one build backend (hatchling/setuptools).
- **uv** for speed and new projects; **Poetry** if you're already invested; plain **pip+venv** is always fine for simple cases.
- Use **`pipx`/`uv tool`** for CLI tools (ruff, black, httpie) so they don't pollute project envs.
