# bonmark — Claude instructions

Full implementation spec is in [`docs/spec.md`](docs/spec.md). Key rules for implementation:

## Commits
Use Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`.

## Implementation constraints
- Single file: `bonmark.py`
- No third-party dependencies — stdlib only (`urllib`, `concurrent.futures`, `re`, `html`, `os`, `sys`, `configparser`, `datetime`)
- Python 3.8+

## CLI conventions
- ANSI color output; disable automatically when stdout is not a TTY
- Every prompt shows a default in brackets; Enter accepts it
- Errors → stderr; all other output → stdout
- Exit codes: `0` success, `1` error, `2` completed but no changes made
- The input file is **never** modified

## Data model
```python
{'type': 'bookmark', 'title': str, 'url': str, 'add_date': str, 'icon': str}
{'type': 'folder',   'title': str, 'add_date': str, 'personal_toolbar': bool, 'children': [...]}
{'type': 'separator'}
```

## Parser
- Do **not** use BeautifulSoup or lxml — they break `<DL>/<DT>` nesting
- Use a regex tokenizer emitting `(DL_OPEN, DL_CLOSE, HR, FOLDER, BOOKMARK)` tokens, then a stack-based tree builder
- For files > 5 MB, use a line-by-line tokenizer (not full-file `DOTALL` regex)

## HTML encoding
Always double-unescape titles before processing:
```python
title = html.unescape(html.unescape(raw))
title = title.replace('\u2018', "'").replace('\u2019', "'")
title = title.replace('\u201c', '"').replace('\u201d', '"')
title = title.replace('\xa0', ' ')
```

## Output filename
```python
ts = datetime.now().strftime('%Y%m%d_%H%M%S')
out_path = f"{os.path.splitext(input_path)[0]}_{ts}.html"
```
Do not re-append a timestamp if the input filename already ends in `_yyyyMMdd` or `_yyyyMMdd_HHmmss`.
