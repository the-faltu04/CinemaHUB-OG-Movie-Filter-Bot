# Cinema HUB OG — Search UX Update

## Scope
This update preserves the existing search, filter, pagination, F-Sub, Linkpays, deep-link and delivery behaviour while adding the requested search-result UX improvements.

## Added

### 1. Fast cinematic first reveal
- First search-result rendering performs one short intermediate message edit followed by the final result view.
- Total intentional delay is approximately 100 ms between the two edits.
- Pagination and filter-result renders skip the animation and remain immediate.
- No external animation library or heavy dependency was added.

### 2. Bold, bright clickable result text
- Every result line remains a Telegram HTML hyperlink to the same `file_<movie_id>` deep-link as before.
- Result text is now wrapped in `<b>...</b>` for stronger readability.
- Existing filename cleaning, real Telegram file-size display and blank-line separation are unchanged.

### 3. Ten-minute result-message expiry
- New environment variable: `SEARCH_RESULT_DELETE_AFTER_SECONDS`.
- Default value: `600` seconds (10 minutes).
- The countdown starts once the final result/correction message first appears.
- Pagination and filter edits do not reset the countdown.
- The user's original search message is never targeted by this cleanup.
- Expiry metadata is stored in MongoDB so the cleanup remains effective after a Render restart.
- A lightweight Telegram JobQueue worker checks every 30 seconds.
- Delivered movie-file deletion remains controlled by the existing `DELETE_AFTER_SECONDS` setting and is not changed.

### 4. Smarter typo correction
The existing SequenceMatcher-only suggestion stage was upgraded to combine:
- whole-query similarity
- word-order-insensitive similarity
- per-word similarity
- word coverage
- small prefix/containment bonuses

Candidates are taken from distinct database titles, so duplicate file versions do not overwhelm the suggestion stage. The normal practical typo cases now targeted include: `Kabri Singh` → `Kabir Singh`, `Kakli` → `Kalki`, `Conjring` → `The Conjuring`, and `Avatr` → `Avatar`.

When a strong correction exists, the bot shows a clean inline title button (Telegram's native box-style button) instead of reporting the movie as unavailable. When multiple genuinely close candidates exist, up to three are shown.

### 5. Existing search lifecycle preserved
- User search message: preserved.
- Bot result message: auto-deleted after 10 minutes.
- User's original search message: auto-deleted on the same fixed 10-minute expiry.
- Search expiry is persisted in MongoDB, so pagination/filter edits and Render restarts do not reset the countdown.
- F-Sub / Linkpays: unchanged.
- Individual file deep-links: unchanged.
- Send All Files: unchanged.
- Quality / Language / Season filters: unchanged.
- Prev / Next pagination: unchanged.

## Validation performed
- `bot.py` and `ingest.py` compile successfully with `py_compile`.
- No new runtime dependency was introduced.
- Existing delivered-file deletion continues to use the original `DELETE_AFTER_SECONDS` job.
- Production Telegram, MongoDB and Linkpays calls were not executed here because they require the deployment's live secrets and environment.

### 6. Fuzzy-search performance guard
- Distinct title candidates are cached in memory for 60 seconds.
- Newly indexed titles are added to the live cache when the cache is active.
- This keeps repeated typo searches from issuing a full MongoDB `distinct` query on every request.
- No third-party fuzzy-matching package was introduced.

## Search backend upgrade

- Removed the hard 100-result ceiling from search retrieval.
- Search now counts the complete matching set and fetches only the current page from MongoDB.
- Search terms can occur anywhere in the stored title + original caption + filename, and word order no longer matters.
- Filter option discovery uses MongoDB `distinct()` across the complete matching set.
- Result cards preserve a compact preview of the original database-channel caption instead of showing filename-derived metadata only.
- Artificial 250ms + 100ms reveal delays were removed so the search result appears in a single fast edit.
