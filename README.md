# bonmark

> A cross-browser CLI tool for cleaning up exported browser bookmarks.

**bonmark** parses the Netscape Bookmark File format (the universal export format used by Firefox, Chrome, Edge, Brave, Opera, and Safari), applies user-selected cleanup operations interactively, and writes a new importable HTML file.

---

## Features

- **Regroup** — automatically clusters bookmarks into folders using domain heuristics and niche keyword analysis
- **Rename** — strips verbose site names, author bylines, and publication dates from bookmark titles
- **Remove** — checks links in parallel and flags dead, inaccessible, or redirected URLs
- **Dedup** — finds and removes duplicate bookmarks across URL variants (`http` vs `https`, trailing slashes, tracking parameters)
- **Interactive or scriptable** — full guided flow with `--interactive`, or bypass every prompt with flags

---

## Installation

```bash
pip install bonmark
```

Or run directly:

```bash
python bonmark.py bookmarks.html
```

---

## Usage

```
bonmark <file>                             # show file info and usage, then exit
bonmark clean <file> --interactive         # full guided flow
bonmark clean <file> -a                    # run all operations non-interactively
bonmark clean <file> -rn                   # regroup + rename only
bonmark check <file>                       # dead links + duplicates report (no changes)
```

See `bonmark --help` for the full flag reference.

---

## Status

> ⚠️ Work in progress — implementation not yet started.

---

## Contributing

*Coming soon.*

---

## License

MIT
