# Cinema HUB OG — Search UX QA Report

## Release
Search UX v2: shared 10-minute cleanup + full-result pagination + caption-preserving display + anywhere/any-order matching.

## Source baseline
`CinemaHUB-OG-Auto-Filter-Bot-main.zip` supplied by the project owner.

## Checks completed

### Python syntax
- `bot.py` — PASS (`python -m py_compile`)
- `ingest.py` — PASS (`python -m py_compile`)
- `python -S -m py_compile bot.py ingest.py` — PASS

### Search logic smoke tests
- Query terms are converted into independent regex conditions under `$and`, so word order is no longer required.
- The searchable `normalized` field already contains title + original caption + filename, so caption-only matches are included.
- `count_movies()` counts the complete matching set with MongoDB `count_documents()`.
- `find_movies()` retrieves only the requested page using `skip()` + `limit()`.
- Pagination numbering continues correctly across pages.
- A simulated 527-result search renders `527 result(s)` with `1-8 shown` on page 1.
- A simulated page 2 starts at result 9.
- Eight caption-preserving result blocks stay below Telegram's 4096-character text limit in the smoke test.
- Filter value discovery uses MongoDB `distinct()` rather than the old capped result list.

### Caption preservation
- Original database-channel captions continue to be stored unchanged.
- Search result display now includes a compact preview of the important original caption lines.
- The title line is suppressed from the preview when it duplicates the indexed title, avoiding redundant text.

### Shared search-message lifecycle
- The original Request Group message ID and bot result message ID are persisted in the search document.
- Both receive the same fixed expiry timestamp when the first result message is persisted.
- Pagination/filter edits do not change that expiry timestamp.
- Cleanup targets both messages.
- Deletion failures caused by temporary Telegram permission/network errors are not marked complete, allowing a later 30-second cleanup pass to retry.
- Already-missing Telegram messages are treated as successfully cleaned up.

### Performance / UX
- The previous artificial 250ms and 100ms search/reveal waits were removed.
- First results, pagination, and filter updates use direct message edits.
- The existing search reaction remains intact.

## Backwards compatibility
- `MAX_SEARCH_RESULTS` remains accepted as an environment variable for compatibility, but it is no longer used as a search-result ceiling.
- `SEND_ALL_LIMIT` remains unchanged.
- `ingest.py` remains present and untouched.
- Existing F-Sub, Linkpays, delivery, auto-indexing, historical indexing, admin, and Render startup flows were not intentionally rewritten.

## Validation limitation
Live Telegram, MongoDB, Linkpays, and Render calls were not executed in this build environment because they require the deployment's live secrets and outbound network access. Source compilation and local logic smoke tests passed.
