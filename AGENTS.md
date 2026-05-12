# AGENTS.md

## What this repo is

This is **not** a code project — it is an automated **mirror/archive of Bunny.net's OpenAPI specifications**. There is no application to build or test. The repo's job is:

1. Periodically scrape `https://docs.bunny.net/openapi` to discover spec download links.
2. Download each spec, normalize it (pretty-print via `jq`), version it from `.info.version`, and compare SHA-256 against the previous copy.
3. Commit changed specs into `data/`, regenerate `data/manifest.json`, and publish `data/` to GitHub Pages at `https://toshy.github.io/bunnynet-openapi-specification/`.
4. Tag a dated release if anything changed.

The whole pipeline is one Taskfile target invoked by one GitHub Actions workflow on a cron.

## Architecture / key files

- `Taskfile.yml` — the **only** logic in the repo. Task `openapi:specs` is a single bash block doing fetch → parse → download → hash → manifest. Requires **`jq` 1.7.1+** and `wget`.
- `.github/workflows/openapi.yml` — cron `0 0,6,12,18 * * *` + `workflow_dispatch`. Installs Task + jq, runs `task openapi:specs`, then uses `stefanzweifel/git-auto-commit-action` (file_pattern `data/*.json`) and `actions/deploy-pages` (path `./data`). Release tag format: `YYYY.MM.DD-<run_id>` with emoji 🤖 (cron) or ✍️ (manual).
- `data/manifest.json` — source of truth consumers fetch. Each entry: `{source, sourceDescription, updatedAt, fileName, fileUrl, fileSha256}`. `updatedAt` is preserved from the previous manifest when the hash is unchanged — **do not** bump it on every run.
- `data/*-v<semver>.json` — the mirrored specs. Filename = `<base>-v<MAJOR.MINOR.PATCH>.json`.

## Conventions specific to this repo

- **Filename mapping is hardcoded** in `Taskfile.yml` via a `case "$lower_text"` block. To add/rename an API, edit both:
  - the `grep -E '(Core Platform API|...|Edge Scripting API)'` allowlist filter, and
  - the corresponding `*"<name>"*) base_filename="<slug>" ;;` case arm.
  Current slugs: `core-api`, `origin-errors-api`, `edge-storage-api`, `stream-api`, `shield-api`, `edge-scripting-api`.
- **Version normalization**: leading `v` is stripped, then `1` → `1.0.0`, `1.2` → `1.2.0`. Default when `.info.version` is missing is `1.0.0`.
- **JSON is pretty-printed with `jq --indent 2`** before hashing — hashes are over the reformatted file, not the upstream bytes. Don't change formatting flags or every file will appear "changed".
- **GitHub Pages URL is built from env var `GIHUB_PAGES_URL`** (note the typo — keep it; it's referenced as `${GIHUB_PAGES_URL}` in the jq call and as `$GIHUB_PAGES_URL` in `env:`).
- Only `data/*.json` is auto-committed. Never write generated artifacts elsewhere.
- The bash block uses associative arrays (`declare -A`) and `while read < <(...)` — it is bash-specific; `shell: bash` is set explicitly. Don't port to POSIX sh.

## Developer workflows

- Run the full pipeline locally: `task openapi:specs` (needs `task`, `jq>=1.7.1`, `wget`). Output lands in `./data/`.
- List tasks: `task` (default target runs `task --list`).
- There are **no tests, no linters, no package manager, no language runtime**. Don't add a `package.json`, `requirements.txt`, etc. unless the user explicitly asks.
- To trigger the real pipeline: run the `OpenAPI specification` workflow via `workflow_dispatch` in GitHub Actions.

## When making changes

- Edits to the scraping/parsing logic almost always belong in `Taskfile.yml`'s `openapi:specs` block. Verify by running it locally and diffing `data/manifest.json` — only `fileSha256`/`updatedAt`/new entries should change.
- If upstream HTML structure changes, the `grep -oP '<a[^>]*href=...'` line is the parser to fix.
- Preserve the unchanged-file short-circuit (hash compare → skip overwrite → reuse previous `updatedAt`); downstream consumers rely on `updatedAt` being meaningful.
- The workflow has `timeout-minutes: 5` and `wget` uses `--tries=5 --retry-on-http-error=429,500,502,503,504`; keep network calls resilient and fast.

