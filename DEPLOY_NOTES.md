# Deploy Notes

## Render commands

Build Command:
```
python -m pip install -r requirements.txt
```

Start Command:
```
python bot.py
```

Keep these the same as above.

## Required indexing settings

```
INDEX_ON_START=false
AUTO_INDEX_NEW_POSTS=true
```

`INDEX_ON_START=false` does **not** disable the manual `/reindex` command. `/reindex` and `/index` explicitly force a historical backfill.

## Admin

Set `ADMIN_IDS` to the numeric Telegram user ID(s), comma-separated. Use `/id` in the bot to see the current account's ID. If an admin command is not authorized, the bot now tells you which ID it received instead of silently doing nothing.

## Database channel

The bot must be a member of the configured `DATABASE_CHANNEL_ID` so Telegram can deliver channel-post updates. Keep `AUTO_INDEX_NEW_POSTS=true`. After posting a new media file, use `/stats`; the movie count should increase before testing a search in the request group.

## Existing/old files

Old channel history requires a valid Telethon `SESSION_STRING`. If the session is malformed/expired/unauthorized, `/reindex` will report that instead of pretending the job completed.
