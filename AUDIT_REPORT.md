# Premium UI / UX build audit

This build is based on the working Render single-file bot and keeps the existing MongoDB, Force-Subscribe, Linkpays, delivery, expiry, and historical-indexing logic intact.

## Added

- Premium `/start` welcome interface with random nature image support.
- Time-based greeting using configurable `GREETING_TIMEZONE` (default `Asia/Kolkata`).
- Clickable `Cinema HUB OG` bot identity.
- `ADD ME TO YOUR GROUP`, `HELP`, `ABOUT`, `TOP SEARCHING`, `UPGRADE` menu.
- Persistent top-search counters stored in `search_counts`.
- Request-group animated/big emoji reaction attempt via Telegram message reactions.
- Premium “searching” acknowledgement edited in-place into results.
- Button-based result rows with pagination.
- Quality / Language / Season filter menus.
- Send All Files deep-link flow preserved.
- Existing movie-result deep-link → Force Subscribe → Linkpays → delivery flow preserved.
- Private-chat F-Sub gate followed by the premium Request Group redirect message.

## Safety / compatibility

- `INDEX_ON_START=false` remains safe by default.
- Manual `/reindex` and `/index` behavior remains independent of `INDEX_ON_START`.
- Existing database channel indexing is unchanged except for the new search statistics collection.
- Existing user migration and Render startup protections are preserved.

## Validation

- `python -m py_compile bot.py` passed.
- AST/function-structure checks passed.
- Distribution ZIP excludes Python bytecode caches and backup files.

Live Telegram/Mongo/Linkpays calls still require the user's deployment credentials and external services.
