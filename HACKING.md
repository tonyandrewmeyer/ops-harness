We welcome contributions to `ops-harness`! Before you start work on a contribution, please also read [CONTRIBUTING.md](./CONTRIBUTING.md).

# Setting up a Dev Environment

To work on `ops-harness` you will need Python >= 3.10. Linting, type checking,
and testing are performed using [`tox`](https://tox.readthedocs.io/en/latest/).

First, make sure to install [uv](https://docs.astral.sh/uv/), for example:

```sh
sudo snap install astral-uv --classic
```

Then install `tox` with extensions, as well as a range of Python versions:

```sh
uv tool install tox --with tox-uv
uv tool update-shell
```

Optionally, to run checks automatically before each commit, install
[pre-commit](https://pre-commit.com/#install) and run `pre-commit install`.

You can validate that you have a working installation by running:

```sh
tox --version
```

For improved performance on the tests, install the library that allows
PyYAML to use C speedups:

```sh
sudo apt-get install libyaml-dev
```

# Repository layout

The harness uses a `src` layout:

- `src/ops/testing/harness/` -- the `Harness` source, contributed to the `ops`
  namespace as `ops.testing.harness`. `ops` and `ops.testing` are
  [namespace packages](https://peps.python.org/pep-0420/) (no `__init__.py`),
  so installing `ops-harness` alongside `ops` merges the two.
- `test/` -- the unit tests (`test/test_testing.py`).

Because of the `src` layout, the source is importable as `testing.harness` when
`src/ops` is on the `PYTHONPATH`. The `tox` environments set this up for you.

# Testing

The following are likely to be useful during development:

```sh
# Run linting, type checking, and unit tests
tox

# Run only the unit tests
tox -e unit

# Run only the tests matching a certain pattern
tox -e unit -- -k <pattern>

# Run the linter
tox -e lint

# Run the static type checker
tox -e static

# Format the code using Ruff
tox -e fmt
```

For more in depth debugging, you can run `pytest` directly. Make sure that
`ops` and the test dependencies are installed, and that `src/ops` is on the
`PYTHONPATH`:

```sh
uv run --with ops --with pytest --with pyyaml --with websocket-client \
    env PYTHONPATH=src/ops pytest
```

# Dependencies

The dependencies of `ops-harness` are kept as minimal as possible, to avoid
bloat and to minimise conflict with the charm's dependencies. The dependencies
are listed in [pyproject.toml](pyproject.toml) in the `project.dependencies`
section. `ops-harness` depends on `ops` itself, which provides the framework
internals that the harness drives.

# Dev Tools

## Formatting and Checking

Test environments are managed with [tox](https://tox.wiki/) and executed with
[pytest](https://pytest.org), with coverage measured by
[coverage](https://coverage.readthedocs.io/).
Static type checking is done using [pyright](https://github.com/microsoft/pyright),
and extends the Python 3.10 type hinting support through the
[typing_extensions](https://pypi.org/project/typing-extensions/) package.

Formatting uses [Ruff](https://docs.astral.sh/ruff/).

All tool configuration is kept in [pyproject.toml](pyproject.toml). The list of
dependencies can be found in the relevant `tox.ini` environment `deps` field.

## Building

The build backend is [setuptools](https://pypi.org/project/setuptools/), and
the build frontend is [uv](https://docs.astral.sh/uv/):

```sh
uv build
```

This produces a source distribution and a wheel in `dist/`. Both include the
PEP 561 `py.typed` marker, so type checkers pick up the harness's inline type
annotations.
