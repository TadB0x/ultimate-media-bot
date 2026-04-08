# Ultimate Media Bot

Telegram bot for downloading media from YouTube, Spotify, SoundCloud, TikTok, Instagram, Twitter/X, Pinterest, Deezer, and 1000+ other sites via yt-dlp. Also compresses and converts audio/video files sent directly.

## Features

- Download audio or video from 1000+ supported sites
- Spotify & SoundCloud playlist support
- SoundCloud search by name
- Audio/video compression and format conversion for uploaded files
- Inline button UI for format, quality, and speed selection
- ACRCloud-powered song recognition (Shazam-style)
- MTProto for uploading files beyond the Bot API size limit
- Stats tracking

## Setup

```bash
cp .env.example .env
# fill in your tokens in .env
pip install python-telegram-bot telethon yt-dlp
python bot.py
```

## Environment Variables

| Variable | Description |
|---|---|
| `BOT_TOKEN` | Telegram bot token from @BotFather |
| `SP_CLIENT_ID` | Spotify app client ID |
| `SP_CLIENT_SECRET` | Spotify app client secret |
| `ACRCLOUD_ACCESS_KEY` | ACRCloud access key (for song recognition) |
| `ACRCLOUD_ACCESS_SECRET` | ACRCloud access secret |

## Notes

- `ffmpeg` must be installed on the machine.
- For Spotify downloads, `spotdl` must be installed.
- Large file uploads use Telethon (MTProto) — set API credentials inside the script.
