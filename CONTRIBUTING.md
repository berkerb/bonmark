# Contributing to bonmark

## Setup

This project uses [uv](https://docs.astral.sh/uv/) for environment and dependency management.

```bash
git clone https://github.com/berkerb/bonmark.git
cd bonmark
uv sync                   # creates .venv and installs the package + dev deps
uv run pre-commit install # install git hooks
bonmark bookmarks.html
```

No install needed either — you can also run directly:

```bash
python bonmark.py bookmarks.html
```

If you don't have uv: `pip install uv` or see the [uv install guide](https://docs.astral.sh/uv/getting-started/installation/).

## Code quality

This project uses [Ruff](https://docs.astral.sh/ruff/) for linting and formatting. Pre-commit hooks run it automatically on every commit.

To run manually:

```bash
uv run ruff check .        # lint
uv run ruff format .       # format
uv run ruff check --fix .  # lint + auto-fix
```

CI runs the same checks on every PR and will block merging if they fail.

## Branching strategy

This project uses [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow). `master` is always releasable and protected — all changes go through a branch and PR.

### Branch naming

Branch names mirror the conventional commit prefix:

| Branch pattern | Use for |
|---|---|
| `feat/<short-description>` | New feature or subcommand |
| `fix/<short-description>` | Bug fix |
| `docs/<short-description>` | Documentation only |
| `chore/<short-description>` | Maintenance (deps, CI, config) |
| `refactor/<short-description>` | Refactoring |

Example: a `feat: add regroup subcommand` commit lives on a `feat/regroup-subcommand` branch.

Delete branches after merging.

### Releases

Releases are tagged directly on `master` using SemVer: `v0.1.0`, `v0.2.0`, etc. The tag message matches the CHANGELOG entry for that version.

## Workflow

1. Branch from `master` using the naming convention above
2. Make your changes
3. Commit using [Conventional Commits](#commit-style)
4. Open a pull request into `master`
5. Delete the branch after merge

## Commit style

This project uses [Conventional Commits](https://www.conventionalcommits.org/):

| Prefix | Use for |
|---|---|
| `feat:` | New feature or subcommand |
| `fix:` | Bug fix |
| `refactor:` | Code change that isn't a fix or feature |
| `docs:` | Documentation only |
| `test:` | Adding or updating tests |
| `chore:` | Maintenance (deps, config, CI) |

## Code style

- **Single file:** all implementation lives in `bonmark.py`
- **No third-party dependencies** — stdlib only (`urllib`, `concurrent.futures`, `re`, `html`, `os`, `sys`, `configparser`, `datetime`)
- **Python 3.8+** compatible
- Follow the data model and parser approach described in [`docs/spec.md`](docs/spec.md)

## Testing

Tests are not yet implemented. When added, run with:

```bash
uv run pytest
```

## Reporting bugs

Open an issue using the [bug report template](https://github.com/berkerb/bonmark/issues/new?template=bug_report.md).
Please include your OS, Python version, bonmark version (`bonmark --version`), and the source browser of the bookmark file.
