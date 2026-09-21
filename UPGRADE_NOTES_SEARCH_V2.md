# Cinema HUB OG — Search & Request UX v2

This release is built directly from the provided working project ZIP. Existing delivery, F-Sub, Linkpays, indexing, admin, and Render architecture are preserved.

## Changes

1. **Shared 10-minute cleanup**
   - The original Request Group user message and the bot result message use one fixed expiry.
   - Expiry state is stored in MongoDB.
   - Pagination/filter edits do not extend the timer.
   - Cleanup retries when Telegram temporarily refuses deletion instead of immediately marking the message complete.

2. **Unlimited-result search architecture**
   - The old `MAX_SEARCH_RESULTS` hard cap is no longer used by search retrieval.
   - Search counts the complete matching set with MongoDB and retrieves only the current page.
   - Prev/Next navigation works across the full result set.

3. **Anywhere / any-order matching**
   - Search words are matched independently anywhere inside the indexed `normalized` field.
   - `normalized` already contains title + original caption + filename.
   - Therefore caption-only matches and reordered multi-word queries are supported.

4. **Caption-preserving result display**
   - Original stored captions remain unchanged.
   - Each result shows a compact preview of the important caption lines (episode/audio/quality/source text) to stay safely inside Telegram's message size limits.

5. **Faster UI**
   - Removed the artificial search and reveal sleeps.
   - First-page search, pagination, and filters use direct result edits.

## Compatibility

- `MAX_SEARCH_RESULTS` remains accepted as a legacy environment variable but is no longer used as a search-result ceiling.
- `SEND_ALL_LIMIT` remains unchanged to avoid altering the existing bulk-delivery behavior.
- `ingest.py` remains intact and is not removed or made optional.
