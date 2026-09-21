# Cinema HUB OG — Release Manifest

## Modified existing files
- `bot.py` — database-backed unlimited search pagination, any-order/anywhere matching across indexed title + original caption + filename, caption-preserving result previews, shared 10-minute user/result message expiry, retry-safe cleanup, faster search UI, and global filter-value discovery.
- `.env.example` — documents the legacy `MAX_SEARCH_RESULTS` setting as compatibility-only and keeps `SEARCH_RESULT_DELETE_AFTER_SECONDS=600`.
- `README.md` — updated search architecture, caption matching, and shared message-expiry behaviour.
- `UPDATE_NOTES_SEARCH_UX.md` — expanded with the v2 search backend changes.

## New documentation
- `UPGRADE_NOTES_SEARCH_V2.md`

## Intentionally preserved
- F-Sub
- Linkpays
- individual file deep links
- Send All Files
- Quality / Language / Season filters
- Prev / Next pagination UI
- MongoDB schema and indexing flow
- database-channel auto-indexing
- historical Telethon indexing
- delivered-file five-minute deletion behaviour via `DELETE_AFTER_SECONDS`
- optional Internet Archive ingestion support via the existing `ingest.py`
- Render web-service entrypoint and `/health` endpoint

## Release cleanup
- Generated `.pyc` bytecode and `__pycache__` directories are excluded from the release ZIP.
- No deployment secrets are bundled.
