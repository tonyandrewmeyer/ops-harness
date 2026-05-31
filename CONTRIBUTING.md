We welcome contributions to `ops-harness`!

`ops-harness` packages the legacy `Harness` unit-testing API as a standalone
distribution. The code itself is maintained alongside the rest of Ops in the
[canonical/operator](https://github.com/canonical/operator) repository. Before
working on changes, please consider [opening an
issue](https://github.com/canonical/operator/issues) explaining your use case.
If you would like to chat with us about your use cases or proposed
implementation, you can reach us at
[Matrix](https://matrix.to/#/#charmhub-charmdev:ubuntu.com) or
[Discourse](https://discourse.charmhub.io/).

Note that `Harness` is deprecated. We accept fixes that keep existing charms
working, but we generally don't add new features. For new tests, prefer the
state-transition testing API in
[`ops.testing`](https://documentation.ubuntu.com/ops/latest/reference/ops-testing/).

For detailed technical information about developing `ops-harness`, see
[HACKING.md](./HACKING.md).

# AI

You're welcome to submit pull requests that are partly or entirely generated using generative AI tools. However, you must review the code yourself before moving the PR out of draft -- by submitting the PR, you are claiming personal responsibility for its quality and suitability. If you are not capable of reviewing the PR (for example, if you are not fluent in Python, or are not familiar with Ops), please do not submit the PR (maybe you'd like to open an issue instead). PRs that are clearly (co-)authored by tools will be closed without review unless there is a human author that claims responsibility for the PR.

Please do not use tools (such as GitHub Copilot) to provide PR reviews. The Charm Tech team also has access to these tools, and will use them when appropriate.

# Pull requests

Changes are proposed as [pull requests on GitHub](https://github.com/canonical/operator/pulls).

- Work on a branch in your own fork.
- Sequence your commits logically if possible. But don't worry too much -- we'll squash to `main` after review.
- Don't force-push after review has started.
- Follow [conventional commit style](https://www.conventionalcommits.org/en/) for the PR title (not required for individual commits).

Examples of PR titles:

- fix: correct the type hinting for relation data
- test: cover secret rotation in the harness
- ci: adjust the workflow that runs the unit tests

We consider Ops too small a project to use scopes, so we don't use them.

## Branch updates

Before you ask for review, please rebase your branch onto `main` so that your changes will merge cleanly.

If you need to bring in the latest changes from `main` after the review has started, please use a merge commit.

# Tests

Changes should include tests. Tests live in the top-level `test` directory, in
`test/test_testing.py`. Try to find the most logical place to add tests, based
on the code that is tested.

Run the tests with `tox -e unit` (see [HACKING.md](./HACKING.md) for details).

# Coding style

We have a team [Python style guide](./STYLE.md), most of which is enforced by CI checks. Please be complete with docstrings and keep them informative for _users_, as the [Ops library reference](https://documentation.ubuntu.com/ops/latest/reference/) is automatically generated from Python docstrings.

# Copyright

The format for copyright notices is documented in the [LICENSE](LICENSE). New files should begin with a copyright line with the current year (e.g. Copyright 2024 Canonical Ltd.) and include the full boilerplate (see APPENDIX of [LICENSE](LICENSE)). The copyright information in existing files does not need to be updated when those files are modified -- only the initial creation year is required.

# Reviews

All changes require review before being merged. Code review typically examines:

- Code quality
- Test coverage
- User experience

When evaluating design decisions, we give priority to the following personas:

- Charm authors and maintainers (highest priority)
- Contributors to the Ops codebase
- Juju developers
