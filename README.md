# AutoFilter Movie Bot — Final Render Build

This is a single-file Telegram bot designed for an authorized movie/series database channel. The web service stays available while MongoDB and Telegram are connected in background workers with retries, so transient network timeouts do not kill the Render process.

## Main flow

`/start` → premium welcome menu → private-chat request-group handoff → Request Group search → animated reaction + premium search acknowledgement → bold bright clickable text results with a fast cinematic reveal, Quality/Language/Season filters and pagination → result deep-link → Force Subscribe → Linkpays short-link → user-bound one-time Telegram return token → authorized database-channel message copied to the user → expiry notice + auto-delete.

The welcome menu includes a random nature image, time-based greeting, clickable bot identity, Help/About/Top Searching/Upgrade sections, and an optional Add Me To Your Group button. Top Searching is ranked from actual search counts recorded since deployment.

## Render

Build command:
`python -m pip install -r requirements.txt`

Start command:
`python bot.py`

The application binds to `0.0.0.0:$PORT` and exposes `/health`. Render requires web services to bind on `0.0.0.0`; `/health` returns 200 even while the Telegram/Mongo workers are still connecting. This prevents a transient external API timeout from causing the web process itself to exit.

## Environment variables

Required: `BOT_TOKEN`, `API_ID`, `API_HASH`, `MONGO_URI` (or legacy `DATABASE_URI`), `DATABASE_CHANNEL_ID` (or `BIN_CHANNEL`), and `REQUEST_GROUP_ID` (or `REQUEST_GROUP_USERNAME`).

`BOT_USERNAME` is optional; the bot reads its real username from Telegram during startup. If an incorrect username is supplied, the Telegram value wins.

Optional UI settings: `DEVELOPER_USERNAME`, `DEVELOPER_NAME`, `GREETING_TIMEZONE`, and `START_IMAGE_URLS` (comma-separated image URLs). If `START_IMAGE_URLS` is blank, a small built-in set of landscape/nature image URLs is used and one is selected at random for `/start`.

`LOG_CHAT_ID` is optional. Leave it absent/blank; never use the literal word `Blank`.

For Force Subscribe, `FSUB_CHANNELS` and `FSUB_INVITE_LINKS` are paired in order. Public channels may use `@username` and can have their join URL generated automatically. Private channels should use their numeric chat ID such as `-100...` plus the matching invite link. If an F-Sub entry is stale/inaccessible, the bot skips that entry instead of crashing startup; if none remain usable, Force Subscribe is disabled and the bot continues running.

## MongoDB migration safety

The bot automatically repairs the common legacy `users` schema problem that caused a unique-index crash: if a document has no `user_id`, it derives it from the old numeric `id`; numeric string user IDs are normalized to integers; duplicate legacy user records are collapsed before the unique sparse index is recreated. This removes the `E11000 ... user_id: null` deployment failure.

## Telegram networking

The bot uses `HTTPXRequest` with configurable connect/read/write/pool timeouts and an exponential retry loop around startup calls such as `getMe` and `getChat`. Telegram's official bot docs expose these timeout controls, and Render health checks are served independently of the Telegram worker.

## Linkpays

The supplied Linkpays developer documentation defines the GET shortening endpoint at `https://linkpays.in/api` with required `api` and `url` parameters. The project expects the JSON response field `shortenedUrl`. The shortener destination is a user-bound `t.me/<bot>?start=sv_<token>` deep-link. When the user returns, the token is checked for the correct Telegram user, expiry, and one-time use. The supplied Linkpays material did not document a server-to-server completion webhook, so the implementation does not invent one.

## Existing database channel

Historical indexing is optional. `INDEX_ON_START=false` is the safe default, so a missing/broken Telethon session can never prevent the bot from running. A manual `/reindex` (or `/index`) now **ignores `INDEX_ON_START`** and performs the historical backfill when a valid Telethon `SESSION_STRING` is present. The build normalizes common whitespace/quote/missing-padding issues and disables only historical backfill if the session is malformed or unauthorized. The bot itself does not download database media to Render; delivery uses Telegram's server-side message copy from the configured database channel. New channel posts are indexed automatically when `AUTO_INDEX_NEW_POSTS=true` and the bot is a member of the database channel.

Admin commands require the numeric Telegram user ID in `ADMIN_IDS`. If an admin command is sent by an unconfigured user, the bot now replies with that user's ID instead of silently ignoring the command. `/id` is also available.

`/stats` reports the current movie count and indexing flags. This lets you verify that a newly posted database file actually reached MongoDB before testing search in the request group.

## Search-group experience

When a user posts a movie/series title in the configured Request Group, the bot tries to set one animated/big emoji reaction on that message, then posts a short premium “searching” card and edits that same card into the results view. The results view uses bold, bright, fully clickable text links for each file, a Send All Files deep-link, Quality/Language/Season filter controls, and database-backed Prev/Next pagination across the full matching result set (not a 100-file cap). Search terms are matched anywhere in the indexed title, original caption, or filename, without requiring the same word order. Each result also preserves a compact preview of the operator-written caption. The original user search message and the bot result message share one fixed 10-minute expiry; cleanup is persisted in MongoDB so pagination/filter edits and Render restarts do not reset the timer. Clicking a file still uses the existing verification + Linkpays delivery flow.

## Permissions

- Bot can access the database channel.
- Bot can read the request group (disable BotFather privacy mode if necessary).
- Bot can check membership in every Force Subscribe channel; administrator permissions are recommended/required for reliable `getChatMember` checks in channels.
- The Telethon account used by `SESSION_STRING` must be able to read the database channel history.

## Testing performed on the distributed source

- Python syntax compilation
- Structural checks for single-file Render entrypoint
- Static checks of startup retry and Mongo migration paths

Live Telegram/Mongo/Linkpays tests cannot be performed from this build environment because the environment has no outbound package/network access and no user secrets.

## Authorized / Public-Domain Automatic Ingestion

The project now includes an optional Internet Archive ingestion worker. It is
**disabled by default** and only imports items whose metadata advertises a
supported public-domain or non-commercial Creative Commons license class
configured by `IA_QUERY`.

### Render variables

Set:

- `IA_INGEST_ENABLED=true`
- `IA_INITIAL_BACKFILL=true` for the first controlled backfill, or leave it false and use `/iasync`
- `IA_BATCH_SIZE=5` to keep uploads gentle while testing
- `IA_MAX_FILE_MB=1800` to cap the temporary download size
- `IA_SCAN_INTERVAL_SECONDS=1800` for a 30-minute scan interval
- `IA_QUERY=...` to narrow the source search to a collection or license set you are allowed to redistribute

Automatic uploads use the configured Telethon `SESSION_STRING`. The Telegram
account represented by that session must have permission to post in the
configured `DATABASE_CHANNEL_ID`.

The worker stores each source identifier in MongoDB `ingest_jobs` so the same
source item is not uploaded repeatedly. After a successful Telegram upload it
also writes the corresponding movie record into the existing `movies`
collection, so the existing search/index flow can use it immediately.

Admin commands:

- `/iasync` — run a small ingestion batch now
- `/iastats` — show ingestion status

The Internet Archive API supports metadata search and item metadata retrieval.
The project intentionally does not scrape or bypass access controls on third-
party download sites. Only use source items that you are legally entitled to
redistribute. Internet Archive itself notes that it does not guarantee the
copyright status of uploader-provided items, so operators must verify the
rights/license of content they import.
