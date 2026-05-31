# ops-harness

`ops-harness` provides `Harness`, the legacy unit-testing API for charms built
with the [`ops`](https://pypi.org/project/ops/) framework. It packages the
`Harness` class as a standalone distribution that `ops` re-exports as
`ops.testing.Harness` -- the same way `ops` re-exports the `ops-scenario`
package's state-transition API under `ops.testing`.

> [!IMPORTANT]
> `Harness` is deprecated. For new charms, and where reasonable for existing
> charms, prefer the state-transition testing API in
> [`ops.testing`](https://documentation.ubuntu.com/ops/latest/reference/ops-testing/)
> (the `ops-scenario` package). `ops-harness` exists so that charms that still
> rely on `Harness` can continue to do so as the class is removed from the core
> `ops` distribution.

- `ops-harness` requires Python 3.8 or above.
- It depends on `ops` 2.23 or above.

## Installation

```sh
pip install ops-harness
```

Or, with [uv](https://docs.astral.sh/uv/):

```sh
uv add ops-harness
```

## Usage

Add `ops-harness` to your charm's test dependencies, then import `Harness` from
`ops.testing`:

```python
from ops.testing import Harness

from charm import MyCharm


def test_config_changed():
    harness = Harness(MyCharm, meta="name: my-charm")
    harness.begin()
    harness.update_config({"log-level": "debug"})
    assert harness.charm.model.config["log-level"] == "debug"
    harness.cleanup()
```

For the full API, see the
[`ops.testing.Harness` reference documentation](https://documentation.ubuntu.com/ops/latest/reference/ops-testing-harness/).

## Contributing and developing

Anyone can contribute to `ops-harness`. See [CONTRIBUTING.md](./CONTRIBUTING.md)
for the contribution process, and [HACKING.md](./HACKING.md) for how to set up a
development environment and run the tests.

## Code of conduct

When contributing, you must abide by the
[Ubuntu Code of Conduct](./CODE_OF_CONDUCT.md).

## License

`ops-harness` is free software, distributed under the Apache Software License,
version 2.0. See [LICENSE](./LICENSE) for details.
