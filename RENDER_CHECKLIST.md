# Render deployment checklist

Build Command:
`python -m pip install -r requirements.txt`

Start Command:
`python bot.py`

Do not make Build Command and Start Command identical: the build command installs dependencies; the start command runs the web service.

Required Render environment variables:
- BOT_TOKEN
- API_ID
- API_HASH
- MONGO_URI (or DATABASE_URI)
- DATABASE_CHANNEL_ID (or BIN_CHANNEL)
- REQUEST_GROUP_ID (or REQUEST_GROUP_USERNAME)

Optional:
- FSUB_CHANNELS
- FSUB_INVITE_LINKS
- LINKPAYS_API
- ADMIN_IDS
- LOG_CHAT_ID
- SESSION_STRING
- INDEX_ON_START

Recommended first deployment:
- `INDEX_ON_START=false`
- Leave `SESSION_STRING` blank unless historical backfill is needed.

The bot can still index new database-channel posts when `AUTO_INDEX_NEW_POSTS=true`.
