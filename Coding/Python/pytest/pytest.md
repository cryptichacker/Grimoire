---
tags: [coding, python, pytest, testing]
type: cheatsheet
source: compiled reference (pytest)
last-verified: 2026-09-03
---

# pytest

## Up
- [[Python]]

pytest is the de-facto Python testing framework — plain `assert`, powerful **fixtures**, **parametrization**, and a huge plugin ecosystem. Install: `pip install pytest`.

---

## Writing Tests

```python
# test_math.py  — files test_*.py, functions test_*
def add(a, b): return a + b

def test_add():
    assert add(2, 3) == 5           # plain assert; pytest rewrites for rich output

def test_types():
    with pytest.raises(TypeError):  # assert an exception is raised
        add("a", 1)
    with pytest.raises(ValueError, match="bad"):
        raise ValueError("bad input")

import pytest
```

Grouping in a class (no `__init__`): methods named `test_*` inside `class TestX:`.

---

## Running

```bash
pytest                       # discover & run all tests
pytest test_math.py          # one file
pytest test_math.py::test_add   # one test
pytest -k "add and not slow" # keyword filter
pytest -m slow               # run marker
pytest -v                    # verbose  ; -q quiet
pytest -x                    # stop at first failure ; --maxfail=2
pytest -s                    # don't capture stdout (see prints)
pytest --lf                  # last-failed ; --ff failed-first
pytest -n auto               # parallel (pytest-xdist)
```

---

## Fixtures (setup/teardown & dependency injection)

```python
import pytest

@pytest.fixture
def db():
    conn = connect()          # setup
    yield conn                # value injected into tests
    conn.close()             # teardown (after yield)

def test_query(db):          # request the fixture by name
    assert db.query("...")

# scopes: function (default), class, module, session
@pytest.fixture(scope="session")
def app(): ...

# autouse — applied without being requested
@pytest.fixture(autouse=True)
def env(monkeypatch):
    monkeypatch.setenv("MODE", "test")
```

- **`conftest.py`** — fixtures placed here are auto-available to all tests in that directory tree (no import).
- Built-in fixtures: `tmp_path` (temp dir), `monkeypatch` (patch env/attrs), `capsys` (capture output), `caplog` (capture logs).

---

## Parametrization (one test, many cases)

```python
@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
])
def test_add(a, b, expected):
    assert add(a, b) == expected

# stack them for a matrix; id= for readable names
@pytest.mark.parametrize("x", [1, 2], ids=["one", "two"])
```

Parametrize fixtures too: `@pytest.fixture(params=[...])` → test runs once per param.

---

## Markers

```python
@pytest.mark.slow
def test_big(): ...

@pytest.mark.skip(reason="wip")
@pytest.mark.skipif(sys.platform == "win32", reason="posix only")
@pytest.mark.xfail(reason="known bug")     # expected to fail

# register in pyproject.toml to avoid warnings:
# [tool.pytest.ini_options]
# markers = ["slow: slow tests"]
```

---

## Mocking

```python
from unittest.mock import Mock, patch

def test_api(monkeypatch):
    monkeypatch.setattr("mymod.requests.get", lambda url: Mock(status_code=200))

@patch("mymod.requests.get")
def test_call(mock_get):
    mock_get.return_value.json.return_value = {"ok": True}
    assert fetch() == {"ok": True}
    mock_get.assert_called_once()
# pytest-mock plugin gives a convenient `mocker` fixture
```

---

## Config & Coverage

```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-ra -q"
markers = ["slow: marks slow tests"]
```

```bash
pip install pytest-cov
pytest --cov=mypkg --cov-report=term-missing --cov-report=html
```

Popular plugins: **pytest-cov** (coverage), **pytest-xdist** (parallel), **pytest-mock**, **pytest-asyncio** (test [[asyncio]] coroutines), **pytest-django**, **hypothesis** (property-based).

---

## Tips
- Prefer **fixtures over setup/teardown methods**; compose small fixtures.
- Use **parametrize** instead of copy-pasting near-identical tests.
- Keep tests fast and isolated; mark slow/integration tests and exclude by default (`-m "not slow"`).
- `conftest.py` is the home for shared fixtures — no imports needed.
- `--lf`/`--ff` speed up the red-green loop while fixing failures.
