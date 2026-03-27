# bonmark

A cross-browser CLI tool for cleaning up exported browser bookmarks. Parses the Netscape Bookmark File format (the universal export format used by Firefox, Chrome, Edge, Brave, Opera, and Safari), applies user-selected cleanup operations interactively, and writes a new importable HTML file.

## Tool overview

The tool is a single Python CLI script (`bonmark.py`) with no required third-party dependencies.

**Default behaviour (no flags):** passing only a file prints a summary of its contents (browser detected, bookmark count, folder structure) and shows usage instructions, then exits. No changes are made. The user must pass at least one operation flag or `--interactive` to do anything.

```
bonmark bookmarks.html                          # show info and usage, then exit
bonmark clean bookmarks.html --interactive      # full guided flow
bonmark clean bookmarks.html -rn                # regroup + rename, non-interactive
bonmark check bookmarks.html                    # dead links + dupes report, no changes
```

## Subcommand architecture

bonmark uses a subcommand structure. If no subcommand is given, `info` is assumed.

### Read-only commands (never write an output file)

| Command | Description |
|---|---|
| `bonmark info <file>` | Show file stats: browser detected, bookmark count, folder structure, then exit. This is the default when no subcommand is given. |
| `bonmark check <file>` | Health check: report dead links and duplicate groups with counts. No prompts, no output file written. Designed to be run periodically as a maintenance command. |

### Write commands (each produces a new output file)

| Command | Description |
|---|---|
| `bonmark clean <file>` | The main command. Runs any combination of operations in one pass. Interactive by default (`-i`); individual operations selected via flags. |
| `bonmark regroup <file>` | Regroup bookmarks into folders only. |
| `bonmark rename <file>` | Clean up bookmark titles only. |
| `bonmark remove <file>` | Check and remove dead/inaccessible links only. |
| `bonmark dedup <file>` | Deduplicate bookmarks only. |

All write commands share the same common flags (output control, dry-run, quiet, etc.) described below. The individual subcommands (`regroup`, `rename`, `remove`, `dedup`) are convenience shortcuts — they are exactly equivalent to `bonmark clean <file> --<operation>`.

## Flags

### Operation flags (for `bonmark clean`)

| Short | Long | Meaning |
|---|---|---|
| `-i` | `--interactive` | Run the full guided interactive flow |
| `-r` | `--regroup` | Regroup bookmarks into auto-detected folders |
| `-n` | `--rename` | Apply standard title cleaning |
| `-x` | `--remove` | Check and remove dead/inaccessible links |
| `-u` | `--dedup` | Deduplicate bookmarks |
| `-a` | `--all` | Run all four operations (equivalent to `-rnxu`) |

Short flags can be combined: `bonmark clean bookmarks.html -rnx` runs regroup + rename + remove in one pass.

### Output flags (all write commands)

| Short | Long | Meaning |
|---|---|---|
| `-o PATH` | `--output PATH` | Write output to a specific file path instead of the auto-generated timestamped name |
| `-O DIR` | `--output-dir DIR` | Write output to this directory, keeping the auto-generated filename |
| `-f` | `--force` | Overwrite the output file if it already exists without prompting |
| `-D` | `--dry-run` | Preview all changes without writing output |
| | `--strip-icons` | Remove ICON= base64 favicon data to reduce output file size |

### Behaviour flags (all write commands)

| Short | Long | Meaning |
|---|---|---|
| `-y` | `--yes` | Auto-confirm all per-item prompts (e.g. URL removal, duplicate resolution) |
| `-q` | `--quiet` | Suppress all output except errors and the final summary |
| `-v` | `--verbose` | Print detailed per-item output for every operation |
| | `--no-progress` | Disable the `\r` progress bar (cleaner output in CI / log files) |

### Tuning flags

| Short | Long | Meaning |
|---|---|---|
| `-w N` | `--workers N` | Parallel workers for URL checking (default: 10) |
| `-g N` | `--max-group-size N` | Max bookmarks per group before subfolders are created (default: 40) |
| | `--timeout N` | HTTP request timeout in seconds (default: 5) |

### General flags

| Short | Long | Meaning |
|---|---|---|
| | `--version` | Print the installed version and exit |
| | `--config PATH` | Load a specific `.bonmarkrc` file instead of the default locations |

## Interactive flow

Triggered by `bonmark clean <file> --interactive` (or `-i`). The tool walks the user through up to five steps:

**Step 1 — File & browser detection**
Parse the input file, detect the source browser from file markers, report bookmark counts. Inform the user that the Netscape format is read by all major browsers so no conversion is needed.

**Step 2 — Select operations**
Ask which of the following to apply (any combination):
1. Regroup bookmarks into folders
2. Rename bookmark titles
3. Remove outdated, inaccessible, or irrelevant bookmarks
4. Deduplicate bookmarks

**Step 3 — Execute selected operations** (each described below)

**Step 4 — Write output**
Write `<original_name>_YYYYMMDD_HHmmss.html` to the same directory as the input file. Print the full path and show browser-specific import instructions.

## Operation: Regroup into folders

**Auto-grouping**

Analyse the bookmark set in two passes:

1. **Known-domain pass:** map well-known domains to standard categories using a hardcoded lookup table (e.g. `youtube.com` → Streaming, `github.com` → Development). Default category names: Streaming, Social, Development, Learning, Productivity, Gaming, News, Tools.

2. **Niche-clustering pass:** for bookmarks not matched in pass 1, extract signals from the URL and title — TLD patterns, subdomain keywords (e.g. `docs.`, `api.`, `blog.`), path segments, and title word frequency — and cluster them into inferred niche groups. Name each cluster after its most frequent keyword (e.g. "Recipes", "Finance", "Academia"). This allows the tool to surface meaningful groups even for collections the hardcoded map has no coverage for.

Present the suggested groups with bookmark counts and sample domains, then ask: **"Use these groups?"**
- If yes: use them.
- If no: ask the user to type a comma-separated list of group names.

Then ask which group uncategorized bookmarks should fall into (default: last group in the list).

**Misc threshold warning**

If more than 40% of bookmarks would land in the catch-all group, warn the user before proceeding:

```
⚠  Warning: 312 of 620 bookmarks (50%) could not be categorised and will go into Misc.
   Consider providing custom group names, or the output may not be more organised than the input.
   Continue anyway? [y/N]:
```

**Group size limit and subfolders**

Each group has a configurable maximum size (default: 40 bookmarks). If a group would exceed this limit, it is automatically split into subfolders based on the next most discriminating signal available (subdomain, path prefix, or title keyword cluster). Example:

```
Development/
  Development/Python        (28 items)
  Development/Data Eng      (35 items)
  Development/Tutorials     (19 items)
```

The subfolder threshold is tunable via `--max-group-size N` or in `.bonmarkrc`.

The toolbar folder (if present) is always preserved as-is and never regrouped.

## Operation: Rename bookmark titles

Verbose browser-assigned titles include the site name, author byline, and publication date. The standard cleaning rules are:

| Pattern | Action |
|---|---|
| ` – Real Python`, `\| Medium`, ` - Wikipedia` (and other known site suffixes) | Remove |
| `\| by Author \| Date \| Publication` (Medium/Substack bylines) | Remove |
| `Title - YouTube` | → `Title (Video)` |
| `GitHub - user/repo: Description` | → `Description (GitHub)` |
| Rate Your Music album pages | → `Album Title — Artist Name` |
| Trailing whitespace, double spaces | Normalize |
| Never return an empty string | Fall back to original title |

All suffix-stripping patterns must be anchored to end-of-string (`$`) and match against known site names only — never partial matches — to avoid accidentally stripping meaningful words from titles.

Present a brief summary of these rules and ask: **"Apply this naming standard?"**
- If yes: apply automatically.
- If no: skip renaming (manual editing is expected after export).

**HTML encoding note:** Titles in the exported file may be double-encoded (e.g. `&amp;#39;` for an apostrophe, `&amp;amp;` for `&`). Always apply `html.unescape()` twice before string comparison or processing:

```python
import html
title = html.unescape(html.unescape(raw))
```

Also normalize curly quotes and non-breaking spaces before matching:

```python
title = title.replace('\u2018', "'").replace('\u2019', "'")
title = title.replace('\u201c', '"').replace('\u201d', '"')
title = title.replace('\xa0', ' ')
```

## Operation: Remove outdated/inaccessible bookmarks

Send HTTP HEAD requests to each bookmark URL in parallel (default: **10 workers**, tunable via `--workers N`). Timeout default: 5 s.

**Result classification:**

| Response | Treatment |
|---|---|
| 2xx | OK — skip |
| 401 / 403 | Print `[skipped — requires authentication]`, do not flag for removal |
| 4xx (other) | Flag as potentially dead |
| 5xx | Flag as potentially dead |
| SSL error | Print `[warning — SSL certificate issue]`, do not flag for removal |
| Timeout | Flag as potentially inaccessible |
| Cross-domain redirect | Flag — likely a parked domain or ad redirect |

Show each flagged bookmark to the user one at a time:

```
[error] 404
       My Old Tutorial
       https://example.com/dead-page
       Remove? [y/n/s=skip remaining]:
```

Options per item: `y` (remove), `n` (keep), `s` (skip the rest of the list).

After the loop, prune removed URLs from the tree before writing output.

## Operation: Deduplicate bookmarks

Run before other operations. Duplicates accumulate in subtle ways:

- Exact URL match saved twice
- `http://` vs `https://` for the same URL
- Trailing slash differences (`/about` vs `/about/`)
- Same article syndicated on multiple domains (e.g. original blog and Medium mirror)

Normalize URLs before comparison: lowercase scheme and host, strip trailing slash, strip common tracking parameters (`?ref=`, `?utm_*`), normalize `http` → `https`. Present duplicates grouped by URL and ask the user which copy to keep (default: the one with the better title or the most recent `ADD_DATE`).

## Netscape Bookmark File format

All major browsers export bookmarks in this format. Standard HTML parsers (BeautifulSoup, lxml) fail because they restructure the `<DL>/<DT>` nesting. Use a custom regex tokenizer instead.

**Key tokens:**

| Token | Meaning |
|---|---|
| `<DL>` / `</DL>` | Folder open / close |
| `<DT><H3 ...>name</H3>` | Folder header (immediately followed by `<DL>`) |
| `<DT><A HREF="..." ADD_DATE="..." ICON="...">title</A>` | Bookmark |
| `<HR>` | Separator (renders as `│` divider in Firefox toolbar) |
| `PERSONAL_TOOLBAR_FOLDER="true"` on `<H3>` | Marks the toolbar folder |

**Parsing approach:** regex tokenizer that emits `(DL_OPEN, DL_CLOSE, HR, FOLDER, BOOKMARK)` tokens, then a stack-based tree builder. When `<DL>` is encountered, push the last-added folder onto the stack; when `</DL>` is encountered, pop.

**Writing:** always use `html.escape()` on titles and URLs before writing. Preserve `ADD_DATE`, `ICON` (base64 favicon data URI), and `PERSONAL_TOOLBAR_FOLDER` attributes.

**Browser detection heuristics:**

| Marker in file | Browser |
|---|---|
| `PERSONAL_TOOLBAR_FOLDER="true"` | Firefox |
| `Bookmarks bar` folder name | Chrome / Chromium-based |
| `Favorites Bar` folder name | Edge |
| `BookmarksBar` folder name | Safari |

## Parser error handling

The parser must handle all of the following gracefully — never crash, always report clearly:

| Condition | Handling |
|---|---|
| File not found | Exit with clear error message |
| Empty file | Exit: "File is empty" |
| Not a Netscape bookmark file (wrong DOCTYPE or no `<DL>`) | Exit: "Does not appear to be a browser bookmark export" |
| Encoding error (BOM, non-UTF-8) | Retry with `utf-8-sig`, then `latin-1`; warn if fallback used |
| Truncated file (unclosed `<DL>`) | Warn: "File appears truncated — some bookmarks may be missing"; continue with what was parsed |
| Tags inside a title (e.g. `<b>`) | Strip inline tags before using the title; never fail on them |
| Catastrophically large file | Use a line-by-line tokenizer rather than a full-file regex with `DOTALL` to avoid backtracking on files > 5 MB |
| Zero bookmarks parsed | Warn and ask the user to confirm before exiting |

## Data model

Each node in the parsed tree is a plain dict:

```python
# Bookmark
{'type': 'bookmark', 'title': str, 'url': str, 'add_date': str, 'icon': str}

# Folder
{'type': 'folder', 'title': str, 'add_date': str, 'personal_toolbar': bool, 'children': [...]}

# Separator
{'type': 'separator'}
```

## Output filename

Use a full timestamp (date + time) to avoid collisions when the tool is run multiple times in the same day:

```python
from datetime import datetime
ts = datetime.now().strftime('%Y%m%d_%H%M%S')
out_path = f"{os.path.splitext(input_path)[0]}_{ts}.html"
# e.g. bookmarks_20260327_143022.html
```

`yyyyMMdd_HHmmss` with an underscore separator is preferred over a flat `yyyyMMddHHmmss` string: it remains lexicographically sortable, is easier to read at a glance, and is safe on all filesystems. Do not re-append the timestamp if the input filename already ends in a `_yyyyMMdd` or `_yyyyMMdd_HHmmss` pattern.

## CLI conventions

- No required third-party packages. `urllib`, `concurrent.futures`, `re`, `html`, `os`, `sys`, `configparser` are sufficient.
- ANSI color output (disabled automatically if stdout is not a TTY).
- Progress bar during URL checking (overwrite the same line with `\r`).
- Every prompt has a sensible default shown in brackets; pressing Enter accepts it.
- The input file is never modified.
- Errors go to stderr; all other output goes to stdout.
- Exit codes: `0` = success, `1` = error, `2` = completed but no changes were made.

## Configuration file (.bonmarkrc)

bonmark looks for a `.bonmarkrc` file in two locations, in order of precedence:

1. `./.bonmarkrc` — current working directory (project-level overrides)
2. `~/.bonmarkrc` — user home directory (personal defaults)
3. Built-in defaults

CLI flags always take the highest precedence and override any config file value. A specific config file can be pointed to with `--config PATH`.

The file uses INI format (parsed with Python's built-in `configparser`):

```ini
[defaults]
workers = 10
max_group_size = 40
strip_icons = false
timeout = 5
output_dir =              # leave blank to write alongside the input file
quiet = false

[groups]
# Override or extend the built-in domain-to-group mapping.
# Format: domain = Group Name
news.ycombinator.com = Tech News
lobste.rs = Tech News
rateyourmusic.com = Music
```

The `[groups]` section lets users teach bonmark about domains that aren't in the built-in map, without modifying the source code. Entries here are merged with the hardcoded map; conflicts favour the user's config.

## Dry-run mode

When `--dry-run` / `-D` is passed, the tool runs all selected operations in memory and prints a preview of every change — groups assigned, titles rewritten, URLs flagged — without writing the output file. Essential for building confidence before the first real run.

## URL checking robustness

Some servers reject `HEAD` requests (returning `405 Method Not Allowed`) even for live pages. Fall back to a `GET` request with `Range: bytes=0-0` to fetch only the first byte, which most servers accept. This avoids false positives on pages that are actually accessible.

Also flag URLs where the final redirect lands on a **different domain** than the original — these are commonly dead links that now point to parked domains, link-shortener landing pages, or ad redirects.

## Resume on interrupted URL check

URL checking can take several minutes on large bookmark sets. If the process is interrupted (Ctrl-C, crash), partial results are written to a temp file (`bonmark_urlcheck_<hash>.tmp` in the system temp directory). On the next run against the same input file, the tool detects the temp file and asks:

```
Found a previous URL check in progress (847/1022 checked).
Resume from where it left off? [Y/n]:
```

The temp file is deleted automatically after a successful run or when the user declines to resume.

## Strip icons

Base64 favicon data embedded in `ICON=` attributes can add several MB to the output file for large bookmark sets. Pass `--strip-icons` to omit these attributes entirely. The bookmarks still work; browsers will re-fetch favicons when the file is imported.

## Before/after summary

Print a compact summary at the end of every write command:

```
─────────────────────────────────────
  Input:   bookmarks.html  (1,022 bookmarks)
  Output:  bookmarks_20260327_143022.html  (198 bookmarks)

  Regrouped: 198 bookmarks across 6 folders
  Renamed:   147 titles cleaned up
  Removed:   12 dead links pruned
  Dupes:     8 duplicates removed
─────────────────────────────────────
```

## Package distribution

The tool should be pip-installable with a single entry point:

```
pip install bonmark
bonmark bookmarks.html
```

This requires a `pyproject.toml` with a `[project.scripts]` entry pointing to `bonmark:main`. No dependencies beyond the Python standard library.
