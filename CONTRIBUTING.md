# Contributing to bonmark

## Setup

No install needed for development — just clone and run directly:

```bash
git clone https://github.com/berkerb/bonmark.git
cd bonmark
python bonmark.py bookmarks.html
```

For the pip-installable entry point, install in editable mode:

```bash
pip install -e .
bonmark bookmarks.html
```

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
python -m pytest
```

## Reporting bugs

Open an issue using the [bug report template](https://github.com/berkerb/bonmark/issues/new?template=bug_report.md).
Please include your OS, Python version, bonmark version (`bonmark --version`), and the source browser of the bookmark file.
