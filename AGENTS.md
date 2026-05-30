# Repository Guidelines

## Project Structure & Module Organization

This repository defines a local Codex skill for reading and explaining books.
`SKILL.md` contains the skill instructions and expected workflow. `scripts/`
contains helper tools, currently `extract_book.py`, which extracts text from
PDF, EPUB, MOBI, and TXT files into structured JSON. Source book files live at
the repository root as `.epub` files. Generated reading notes belong in
`workspace/`, while extracted JSON files such as `workspace/<book>_extracted.json`
are ignored by Git.

## Build, Test, and Development Commands

Install optional extraction dependencies as needed:

```bash
python3 -m pip install pymupdf pypdf ebooklib mobi beautifulsoup4
```

Extract a book into JSON:

```bash
python3 scripts/extract_book.py <book_path> -o workspace/<book_slug>_extracted.json
```

Limit extraction during debugging:

```bash
python3 scripts/extract_book.py <book_path> --max-pages 20
```

Run a basic syntax check before committing Python changes:

```bash
python3 -m py_compile scripts/extract_book.py
```

## Coding Style & Naming Conventions

Use Python 3 with four-space indentation. Keep helper functions small and named
with `snake_case`. Prefer clear standard-library code and lightweight fallbacks
before adding new dependencies. Generated filenames should use short,
descriptive slugs, for example `workspace/nietzsche_extracted.json`, to avoid
overwriting older outputs.

## Testing Guidelines

There is no formal test suite yet. For extractor changes, verify at least one
small sample per affected format when possible and inspect metadata fields such
as `processed_pages`, `content_pages`, and `skipped_pages`. For EPUB changes,
also verify that extracted chapters follow `content.opf` spine order or the
book table of contents; numeric chapter files such as `chapter10.xhtml` must not
sort before `chapter2.xhtml`. Always run `python3 -m py_compile
scripts/extract_book.py` after editing the script.

## Parallel Execution Guidelines

Prefer parallel execution for independent read-only and validation work: file
searches, file reads, metadata inspection, chapter-order checks, `git diff`,
`wc`, `jq`, and syntax checks. For larger reading tasks, use the pattern
parallel context gathering, serial judgment and edits, parallel verification,
then a concise summary.

Do not parallelize writes to the same file, `apply_patch` with other writes,
steps with strict dependencies, or commands that can race over generated files.
If parallelism would make the work harder to audit or could broaden the edit
scope, run the steps serially.

## Commit & Pull Request Guidelines

The existing history uses short, imperative commit subjects, for example
`Add book reader skill and reading notes`. Follow that style and keep each
commit focused. Pull requests should describe the changed workflow, list manual
verification commands, and note any new dependencies or generated files.

## Security & Generated Content

Do not commit extracted JSON, caches, or temporary files. Avoid adding copyrighted
book text to tracked files unless it is brief, necessary, and clearly justified.
