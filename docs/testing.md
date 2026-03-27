# bonmark — Testing strategy

## Overview

All tests use Python's stdlib `unittest` — no pytest or other third-party test runners. This is consistent with the project's no-external-dependencies constraint.

Run the full suite locally:

```bash
uv run python -m unittest discover -s tests -v
```

---

## Layout

```
tests/
├── __init__.py
├── fixtures/
│   └── minimal.html       # Minimal Netscape bookmark file for parser tests
└── test_smoke.py          # Smoke test — parser round-trip
```

---

## Fixture: `tests/fixtures/minimal.html`

A minimal valid Netscape bookmark file designed to exercise the parser's edge cases:

```html
<!DOCTYPE NETSCAPE-Bookmark-file-1>
<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=UTF-8">
<TITLE>Bookmarks</TITLE>
<H1>Bookmarks Menu</H1>
<DL><p>
    <DT><H3 ADD_DATE="1700000000">Dev &amp; Tools</H3>
    <DL><p>
        <DT><A HREF="https://example.com" ADD_DATE="1700000001">Example&#x2019;s Site</A>
        <DT><A HREF="https://python.org" ADD_DATE="1700000002" ICON="data:image/png;base64,abc">Python</A>
        <HR>
    </DL><p>
</DL>
```

What each element covers:

| Element | What it tests |
|---|---|
| `&amp;` in folder title | HTML entity unescaping |
| `&#x2019;` (curly apostrophe) in bookmark title | Unicode smart-quote normalisation |
| `ICON=` attribute | Icon field parsing and preservation |
| `<HR>` inside a folder | Separator token handling |
| Nested `<DL>` | Stack-based tree builder — folder nesting |

---

## Smoke test: `tests/test_smoke.py`

Seven assertions that verify a basic parse round-trip:

| # | Assertion | What it guards |
|---|---|---|
| 1 | Root node type is `"folder"` | Parser returns a well-formed tree root |
| 2 | Exactly one child folder under root | `<H3>` → folder token correctly consumed |
| 3 | Folder title is `"Dev & Tools"` | `&amp;` unescaped once to `&` |
| 4 | Exactly two bookmarks in the folder | `<A>` → bookmark tokens correctly consumed |
| 5 | One bookmark title is `"Example's Site"` | Curly quote `\u2019` normalised to straight `'` |
| 6 | Exactly one separator in the folder | `<HR>` → separator token correctly consumed |
| 7 | Python bookmark `icon` field starts with `"data:image/png"` | ICON attribute parsed and preserved |

The tests call `bonmark.parse(path)` directly (not via subprocess), so `bonmark.py` must expose a public `parse(path: str) -> dict` function returning the root folder node.

---

## CI plan

A `test` job will be added to `.github/workflows/ci.yml` once `bonmark.py` is implemented. It will run the full test suite against Python **3.8** and **3.12** using a matrix strategy, covering the oldest and newest supported versions.

```yaml
test:
  name: Tests (Python ${{ matrix.python-version }})
  runs-on: ubuntu-latest
  strategy:
    matrix:
      python-version: ["3.8", "3.12"]
  steps:
    - uses: actions/checkout@v4
    - uses: astral-sh/setup-uv@v5
      with:
        enable-cache: true
        cache-dependency-glob: "uv.lock"
        python-version: ${{ matrix.python-version }}
    - run: uv sync
    - run: uv run python -m unittest discover -s tests -v
```

This job is intentionally **not added yet** — it would fail on every push until `bonmark.py` exists.
