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

## Workflow

1. Fork the repo and create a branch from `master`
2. Make your changes
3. Commit using [Conventional Commits](#commit-style)
4. Open a pull request

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
