# Contributing to Astroid

This guide explains how to prepare a development environment from a clean checkout, build the project, and run the local test suite on macOS using the tooling we currently exercise on `main`.

## Prerequisites

- Homebrew-installed Python 3.13; the executable lives at `/opt/homebrew/bin/python3.13` after `brew install python@3.13`.
- [`uv`](https://github.com/astral-sh/uv) for fast dependency installs (`brew install uv`).
- macOS command-line tools that can create symlinks (required by a few tests).

## Create a Python 3.13 virtual environment

All commands below assume the repository root (`/Users/danielzayas/Development/pylint-dev/astroid` in the examples).

```bash
/opt/homebrew/bin/python3.13 -m venv .venv
source .venv/bin/activate
python --version  # should report Python 3.13.9
```

If you need to recreate the environment, remove `.venv/` first (`rm -rf .venv`).

## Install dependencies with uv

1. Install all runtime, testing, and tooling requirements:

   ```bash
   source .venv/bin/activate
   uv pip install -r requirements_full.txt
   ```

2. Install Astroid itself in editable mode so changes are immediately available to tests:

   ```bash
   uv pip install -e .
   ```

3. (Optional) If you plan to produce wheels and sdists locally, install the `build` frontend once:

   ```bash
   uv pip install build
   ```

`uv` writes to `~/.cache/uv`; if sandboxed shells block that directory, rerun commands with the necessary permissions.

## Build source and wheel distributions

With the environment activated:

```bash
python -m build
```

This creates `dist/astroid-<version>.tar.gz` and `dist/astroid-<version>-py3-none-any.whl`. The build process automatically spins up an isolated build backend environment, so no additional steps are required beyond having `build` installed in the venv.

To clean up build outputs, remove the `build/`, `dist/`, and `*.egg-info/` directories.

## Run the test suite

We run the full pytest suite with uv so it uses the virtualenv interpreter and dependency graph:

```bash
uv run pytest
```

Pytest is already configured via `[tool.pytest.ini_options]` in `pyproject.toml` to look inside `tests/`, to fail on warnings, and to skip tests marked `acceptance`. You can filter subsets of tests with standard pytest selectors (for example `uv run pytest tests/test_manager.py -k namespace`).

### Current status on `main`

Running the suite on 2025-12-04 with Python 3.13.9 produced:

- 1,898 total tests: 1,830 passed, 49 skipped, 17 xfailed, **2 failed**.
- Failing tests:
  - `tests/test_get_relative_base_path.py::TestModUtilsRelativePath::test_symlink_resolution` – expected `_get_relative_base_path` to resolve a symlink inside a temp directory but received `None`.
  - `tests/test_manager.py::AstroidManagerTest::test_module_is_not_namespace` – `util.is_namespace` returned `True` for one of the inferred `EXT_LIB_DIRS`, causing the final assertion to fail.

Please consult the latest pytest logs before relying on these results, as the status can change when new commits land on `main`.

## Additional tooling

- `pre-commit run --all-files` keeps formatting (Black), lint (Ruff/Pylint), and metadata checks aligned with CI.
- `tox` is available for matrix-style testing (`uv run tox -e py313`), though the standard workflow described above is sufficient for most contributions.

Feel free to file issues or start a discussion on the [Astroid Discord server](https://discord.gg/Egy6P8AMB5) if you run into setup problems.

