# Harness Sync State

This file tracks the last-synced commit from
[canonical/operator](https://github.com/canonical/operator) and documents
the adaptations made in this repository to maintain Python 3.8 / ops 2.23+
compatibility.

## Upstream source

| Item | Value |
|------|-------|
| Repository | `canonical/operator` |
| In-scope file | `ops/_private/harness.py` |
| ops-harness file | `src/harness/harness.py` |
| Out of scope | `testing/` (ops-scenario), `ops/tracing/`, `tracing/` |

---

## 2026-06-01 — baseline established

**Upstream SHA:** `6e02f95ae3467f67ea14819485368d0a2d3f74a0`
**Upstream commit subject:** `fix: treat remote unit zero as explicit (#2454)`
**Synced on:** 2026-06-01

### In-scope upstream commits pulled this cycle

_None — this entry establishes the baseline. canonical/operator HEAD at time
of sync equals the baseline SHA; no new in-scope commits exist._

### How the baseline was determined

The initial ops-harness sync commit (`8da3737`, "Sync harness from
canonical/operator 3.7.1") explicitly recorded `canonical/operator@6e02f95a`
as its source. The current canonical/operator `main` HEAD is also `6e02f95`,
confirming this is the correct baseline with no gap.

The last commit that actually touched `ops/_private/harness.py` within the
50-commit shallow history visible at baseline time was `208af9b` ("fix: pass
the endpoint name through to relation-get"), which predates `6e02f95` and is
therefore already included in the synced content.

### Standing adaptations

The following changes are permanently applied to `src/harness/harness.py`
relative to the upstream `ops/_private/harness.py`. Every future sync must
re-apply (or carry forward) these diffs.

#### 1. Absolute imports (always required)

Upstream uses relative imports from inside the `ops` package:

```python
# upstream
from .. import charm, framework, model, pebble, storage
from ..charm import CharmBase, CharmMeta, RelationRole
from ..jujucontext import JujuContext
from ..model import Container, RelationNotFoundError, _StatusName
from ..pebble import ExecProcess
from . import yaml
```

ops-harness rewrites these as absolute imports because it lives outside the
`ops` package:

```python
# ops-harness
from ops import charm, framework, model, pebble, storage
from ops._private import yaml
from ops.charm import CharmBase, CharmMeta, RelationRole
from ops.model import Container, RelationNotFoundError
from ops.pebble import ExecProcess
```

`TYPE_CHECKING`-guarded imports are likewise rewritten to the `ops.*` form.

#### 2. ops 2.x / 3.x symbol shim

ops 3.x renamed several internal symbols relative to ops 2.x.  The harness
targets ops 2.23+, so the following runtime shim is injected after the
absolute-import block:

```python
if typing.TYPE_CHECKING:
    from ops.jujucontext import JujuContext
    from ops.model import _StatusName
else:
    try:
        from ops.jujucontext import JujuContext
    except ImportError:
        from ops.jujucontext import _JujuContext as JujuContext
    try:
        from ops.model import _StatusName
    except ImportError:
        from ops.model import StatusName as _StatusName
```

The JujuContext constructor also changed name (`from_dict` → `_from_dict`);
the call site is guarded with a matching try/except:

```python
try:
    self._juju_context = JujuContext._from_dict(context_environ)
except AttributeError:
    self._juju_context = cast(
        'JujuContext',
        JujuContext.from_dict(context_environ),  # type: ignore[attr-defined]
    )
```

#### 3. Python 3.8/3.9 runtime type-alias compatibility

`from __future__ import annotations` makes *annotations* lazy, but type
aliases, functional `TypedDict`s, base classes, and `default_factory`
arguments are **evaluated at import time**.  Upstream uses PEP 604 (`X | Y`)
and PEP 585 (`dict[...]`, `list[...]`) syntax that only works at runtime on
Python 3.10+/3.9+.  These are replaced with `typing.*` equivalents:

| Upstream (Python 3.10+) | ops-harness (Python 3.8+) |
|-------------------------|---------------------------|
| `bytes \| str \| …` | `typing.Union[bytes, str, …]` |
| `int \| None` | `typing.Optional[int]` |
| `dict[K, V]` (at runtime) | `typing.Dict[K, V]` |
| `list[T]` (at runtime) | `typing.List[T]` |
| `ExecResult \| None` | `typing.Optional[ExecResult]` |
| `collections.abc.Callable[[…], …]` (subscripted at runtime) | `typing.Callable[[…], …]` |
| `dict[str, str \| int \| float \| bool]` as base class | `typing.Dict[str, typing.Union[…]]` |

#### 4. Dataclass `default_factory` with typed constructors

Upstream uses `default_factory=dict[int, set[str]]` and
`default_factory=set[str]`, which call the generic alias as a zero-argument
factory — valid on Python 3.9+ but not 3.8.  Replaced with named helper
functions:

```python
def _new_grants() -> dict[int, set[str]]:
    return {}

def _new_user_secrets_grants() -> set[str]:
    return set()

# in the dataclass:
grants: dict[int, set[str]] = dataclasses.field(default_factory=_new_grants)
user_secrets_grants: set[str] = dataclasses.field(default_factory=_new_user_secrets_grants)
```

#### 5. `State` dummy-class docstring

Upstream has a bare `pass` body for the dummy `State` placeholder class.
ops-harness adds a one-line docstring to satisfy linters:

```python
"""Dummy placeholder for ops.testing.State.

Only used as a type annotation when scenario is not installed,
so the body is intentionally empty.
"""
```

---

## Sync procedure (for future maintainers)

1. Note the `Last-synced SHA` from the most recent entry in this file.
2. In the `canonical/operator` checkout, run:
   ```
   git log --oneline <last-sha>..HEAD -- ops/_private/harness.py
   ```
   to list in-scope upstream commits since the last sync.
3. Compute the diff:
   ```
   git diff <last-sha>..HEAD -- ops/_private/harness.py
   ```
4. Apply the diff to `src/harness/harness.py`, re-applying the standing
   adaptations from §1–5 above to any new code.
5. Update this file: new SHA, date, commit list, and any new adaptation notes.
6. Branch `sync/upstream-YYYY-MM-DD`, commit
   `chore: sync harness from canonical/operator@<short-sha>`, push, open PR.
