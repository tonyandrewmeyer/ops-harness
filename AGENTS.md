# Agent Instructions for ops-harness

## Project Overview

`ops-harness` packages `Harness`, the legacy unit-testing API for charms built
with the [`ops`](https://pypi.org/project/ops/) framework, as a standalone
distribution. `ops` re-exports it as `ops.testing.Harness`, the same way it
re-exports the `ops-scenario` package under `ops.testing`.

This repository uses a `src` layout and contains a single package:

- **`src/harness/`** - The `Harness` class and its supporting types, packaged as
  the standalone `harness` import package. `ops` is a regular package (not a
  namespace package), so the source lives in its own top-level `harness` package
  and `ops` imports from it rather than the source being merged into `ops.*`.
- **`test/`** - The unit tests (`test/test_testing.py`).

The source is maintained alongside the rest of Ops in
[canonical/operator](https://github.com/canonical/operator) and synced here.

`Harness` is **deprecated**. We accept fixes that keep existing charms working,
but we generally don't add new features. For new tests, the recommended API is
the state-transition testing framework in `ops.testing` (the `ops-scenario`
package).

This is a mature, production library used by many charms. Changes require
careful consideration of backward compatibility and testing.

## Key Files Reference

| File | Purpose |
|------|---------|
| `src/harness/harness.py` | The `Harness` class and its helpers |
| `src/harness/__init__.py` | Public API exports |
| `test/test_testing.py` | The unit tests |
| `pyproject.toml` | Packaging, lint, and type-check configuration |
| `tox.ini` | Test, lint, and static-check environments |
| `STYLE.md` | Team Python style guide |
| `CONTRIBUTING.md` | PR and contribution process |
| `HACKING.md` | Development setup and workflow |

## Important Notes

### Backward Compatibility
- **Always** preserve backward compatibility in the public API.
- **Document** all breaking changes in commit messages.
- **Always** preserve existing behavior unless fixing a bug.

### Test Organisation
- Tests live in `test/test_testing.py`.
- Because of the `src` layout, the package is importable as `harness` when
  `src` is on the `PYTHONPATH`; the `tox` environments set this up.

## When Making Changes

### Code Modifications
1. **Read existing code** before proposing changes - use the Read tool to understand patterns.
2. **Check tests** - examine `test/test_testing.py` for similar test patterns.
3. **Verify changes** - run `tox` after changes to ensure linting, type checks, and unit tests pass.

## Development Standards

### Language & Type Checking
- Follow conventions in STYLE.md.
- Use Ruff for formatting (`tox -e fmt`).
- Python 3.10+ with **full type hints** required (check with `tox -e lint` and `tox -e static`).
- Use modern `x: int | None` annotations, not old-style `x: Optional[int]`.
- Always provide a return type, other than for `__init__` and in test code.

### Import Style
```python
# DO: Import modules, not objects (except typing)
import ops
import subprocess
from typing import Generator  # typing is an exception

class MyCharm(ops.CharmBase):
    def handler(self, event: ops.PebbleReadyEvent):
        subprocess.run(['echo', 'hello'])

# DON'T: Import objects directly
from ops import CharmBase, PebbleReadyEvent  # Avoid this
```

## Documentation Guidelines

Comments are always full sentences that end with punctuation. Avoid using comments to explain *what* the code is *doing*, use them (sparingly, as required) to explain *why* the code is doing what it is doing.

### Docstring Style

Use Google-style docstrings for all public APIs, with proper formatting. The text is used to generate reference documentation with Sphinx so must be appropriate ReST.

```python
def my_function(param: str, count: int = 1) -> list[str]:
    """Brief one-line summary.

    Longer description providing more context. Focus on what the function
    does for users, not implementation details.

    Args:
        param: Description of the parameter.
        count: Number of times to repeat. Defaults to 1.

    Returns:
        A list of processed strings.

    Raises:
        ValueError: If count is negative.
    """
```

**Documentation writing tips:**
- Use **active voice**: "Create a check" not "A check is created"
- Be **objective**: Avoid "simply", "easily", "just"
- Be **concise**: No long introductions, get to the point
- Use short sentences and simple phrasing
- Be consistent with choice of words
- Avoid words or phrases specific to US or UK English where possible, and use British English otherwise
- State conditions **positively**: What should happen, not what shouldn't
- Spell out abbreviations and avoid Latin: "for example" not "e.g."
- Use **sentence case** for headings

### Version Dependencies

Only document Juju version dependencies in docstrings:

```python
def new_feature():
    """Do something new.

    .. jujuadded:: 3.5
        Further functionality was added in Juju 3.6.
    """
```

Don't document Ops version changes in docstrings - that's in the changelog.

## Pull Request Guidelines

See CONTRIBUTING.md for more details.

Follow conventional commit style in PR titles:
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `refactor:` - Code refactoring
- `test:` - Test additions or updates
- `chore:` - Maintenance tasks
- `ci:` - CI/CD changes

Examples:
- `fix: correct type hints for relation data`
- `test: cover secret rotation in the harness`

The project does not use conventional commit "scopes".

### Before Submitting
1. Add tests for any new functionality.
2. Update docstrings for any API changes.
3. Ensure backward compatibility.
4. Run `tox -e fmt` to format code.
5. Run `tox -e lint` to check linting.
6. Run `tox -e static` to check types.
7. Run `tox -e unit` to verify unit tests pass.
