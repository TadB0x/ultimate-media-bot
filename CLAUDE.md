# Audiobot Project Context

## What This Is
A Telegram bot (`@TadBoxAudioBot`) that downloads audio/video from YouTube, Spotify, SoundCloud, TikTok, Instagram, and 1000+ sites. It lives in a group called **Audio/Video @ TadBox**.

## File Structure
```
/home/debian/audiobot/
├── bot.py              ← main bot code (Python, python-telegram-bot library)
├── bot.log             ← live log file
└── audiobot.service    ← systemd service definition
```

## How to Run / Restart
```bash
sudo systemctl restart audiobot
sudo systemctl status audiobot
tail -f /home/debian/audiobot/bot.log
```

Service runs as user `debian`, uses `/home/linuxbrew/.linuxbrew/bin/python3` (Python 3.14).

## Key Technical Details

### Two Python environments — IMPORTANT
- **Bot runtime**: `/home/linuxbrew/.linuxbrew/bin/python3` → Python 3.14
- **spotdl**: installed under `python3.13` (`~/.local/lib/python3.13/site-packages/spotdl`)
- The bot calls spotdl via `subprocess` with `python3.13 -m spotdl ...` — never imports it directly
- linuxbrew's Python 3.14 has a broken spotdl install (`pkg_resources` missing) — do NOT import spotdl in bot.py

### Spotify downloads
- Uses `spotdl` (python3.13) via subprocess
- Credentials: client_id `57bd4e27a02543e69231328957fb3f88`, secret `3fda9a99e45b41cf9ce53bde528acc6d` (constants `SP_CLIENT_ID`, `SP_CLIENT_SECRET` in bot.py)
- spotdl's `song.py` was patched to use `.get()` for `genres`, `label`, `popularity` (Spotify API stopped returning these reliably)
- spotdl's `metadata.py` was patched to guard against `None` publisher

### Spotify search
- Uses Spotify Web API directly via `requests` (no spotdl) — `_sp_get_token()` + `_sp_search_sync()`
- Command: `/search_sp <query>`

### SoundCloud search
- Uses yt-dlp `scsearch` — must use `entry["webpage_url"]` not `entry["url"]` (the latter returns internal API stubs that don't download)

### YouTube video download
- Uses `tv_embedded` player client — gives real separate video+audio streams up to 1080p
- `android` client only gives combined streams up to 360p — do NOT use for video
- Audio still uses `android` client (works fine for audio-only)
- All videos download at **1080p max** (no 4K)

### Thumbnail + metadata in Telegram
- `reply_audio()` needs explicit `title`, `performer`, `thumbnail` params — Telegram ignores embedded ID3 tags
- `extract_audio_meta(path)` helper reads ID3/MP4 tags via mutagen and returns `(title, performer, thumb_bytes)`

## Bot Commands
- `/search` — Search YouTube
- `/search_sc` — Search SoundCloud
- `/search_sp` — Search Spotify
- `/merge` — Merge audio files
- `/stats` — Bot statistics
- `/help` — Help

## Common Fixes Reference
- **Bot not responding**: `sudo systemctl restart audiobot`
- **Spotify KeyError**: check `~/.local/lib/python3.13/site-packages/spotdl/types/song.py` for hardcoded dict key access
- **Wrong SoundCloud results**: ensure `webpage_url` is used not `url` in `_sc_search_sync`
- **Video too small / wrong quality**: ensure `tv_embedded` client is used for video, not `android`

## Other Projects on This Machine
```
/home/debian/
├── audiobot/          ← THIS project
├── trading/           ← MT5 forex trading scripts
├── web-projects/      ← horizonglow website, design system
├── instagram-tools/   ← Instagram scrapers
├── telegram-tools/    ← Pyrogram scripts, session files
├── apps/              ← buttermax, flappy-bird, messenger-lite
├── installers/        ← .deb, .apk, .exe packages
└── misc-scripts/      ← vpn-monitor, computer_control.sh, test scripts
```

## System Info
- OS: Debian 13 (bookworm)
- Display: `:10.0` (X11)
- Screenshot tool: `scrot`, screenshots go to `~/screenshots/`
- Computer control: `~/misc-scripts/computer_control.sh` (wraps xdotool with before/after screenshots)
