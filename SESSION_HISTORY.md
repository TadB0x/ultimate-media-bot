# Session History — Full Context

## What Was Built / Fixed in This Session

### 1. Computer Control Setup
- Installed `scrot` for screenshots
- Created `~/misc-scripts/computer_control.sh` — wraps xdotool/wmctrl with automatic before/after screenshots saved to `~/screenshots/`
- Display is `:10.0` (X11), resolution 1920x1080
- Used it to open Telegram, navigate to Audio/Video @ TadBox group, send a YouTube link, and interact with the bot

### 2. Spotify Download Fix (`bot.py`)
**Problem:** `spotdl 4.2.10` crashed with `KeyError: 'genres'` — Spotify API stopped returning `genres`, `label`, `popularity` fields reliably.

**Fixes applied:**
- Upgraded spotdl to 4.4.3
- Patched `~/.local/lib/python3.13/site-packages/spotdl/types/song.py`:
  - `raw_album_meta["genres"]` → `raw_album_meta.get("genres", [])`
  - `raw_artist_meta["genres"]` → `raw_artist_meta.get("genres", [])`
  - `raw_album_meta["label"]` → `raw_album_meta.get("label")`
  - `raw_track_meta["popularity"]` → `raw_track_meta.get("popularity")`
- Patched `~/.local/lib/python3.13/site-packages/spotdl/utils/metadata.py`:
  - Guarded `audio_file[tag_preset["encodedby"]] = song.publisher` with `if song.publisher is not None`

### 3. SoundCloud Search Fix (`bot.py`)
**Problem:** `_sc_search_sync()` used `entry["url"]` which returns internal API stub URLs (`api.soundcloud.com/tracks/soundcloud%3Atracks%3A...`) — not downloadable.

**Fix:** Changed to `entry.get("webpage_url") or entry.get("url", "")` — returns real `soundcloud.com/artist/track` URLs.

### 4. Artist + Thumbnail in Telegram Audio Messages (`bot.py`)
**Problem:** `reply_audio()` only passed `audio` and `filename` — Telegram ignores embedded ID3 tags, so no artist/title/thumbnail showed.

**Fix:** Added `extract_audio_meta(path)` helper that reads ID3 tags via `mutagen` and returns `(title, performer, thumb_bytes)`. These are now passed explicitly to `reply_audio()`.

### 5. Spotify Search Added (`bot.py`)
- Command: `/search_sp <query>`
- Uses Spotify Web API directly via `requests` (NOT spotdl — spotdl is broken on linuxbrew Python 3.14)
- `_sp_get_token()` — client credentials OAuth flow
- `_sp_search_sync()` — hits `https://api.spotify.com/v1/search`
- Results shown as inline buttons, selecting one goes straight to audio quality picker then downloads via spotdl

### 6. Video Quality Fix (`bot.py`)
**Problem:** Android player client only provides combined streams up to 360p — 4K requests silently fell back to 360p.

**Fix:** Changed video downloads to use `tv_embedded` player client which provides proper separate video+audio streams.

**Also:** Removed video quality picker entirely — all videos now download at 1080p max (no user prompt).

### 7. File Organization
Cleaned up `/home/debian/` into:
```
trading/          ← MT5 scripts, backtest.py, trade logs, dow-trading-agent
web-projects/     ← horizonglow, HTML files, design system
instagram-tools/  ← scrapers, downloaders, debug files, instagram_media
telegram-tools/   ← pyro scripts, .session files
installers/       ← .deb, .apk, .exe, pulseaudio source
misc-scripts/     ← vpn-monitor, computer_control.sh, test scripts
apps/             ← buttermax, flappy-bird, messenger-lite
audiobot/         ← THIS project
```

---

## Critical Technical Notes

### Two Python Environments
| Environment | Path | Used For |
|-------------|------|----------|
| Python 3.14 | `/home/linuxbrew/.linuxbrew/bin/python3` | Bot runtime |
| Python 3.13 | system python3.13 | spotdl (called via subprocess) |

**Never import spotdl directly in bot.py** — linuxbrew Python 3.14 has broken spotdl (`pkg_resources` missing).

### Spotify Credentials
```python
SP_CLIENT_ID     = "57bd4e27a02543e69231328957fb3f88"
SP_CLIENT_SECRET = "3fda9a99e45b41cf9ce53bde528acc6d"
```

### YouTube Player Clients
- `android` → combined streams only, max 360p — use for **audio only**
- `tv_embedded` → separate streams, up to 4K — use for **video**

### SoundCloud Search
Always use `entry.get("webpage_url")` not `entry.get("url")` from yt-dlp extract_flat results.

---

## Bot Commands Reference
| Command | Function |
|---------|----------|
| `/search <q>` | YouTube search |
| `/search_sc <q>` | SoundCloud search |
| `/search_sp <q>` | Spotify search |
| `/merge` | Merge audio files |
| `/stats` | Bot statistics |
| `/help` | Help |

## Service Management
```bash
sudo systemctl restart audiobot
sudo systemctl status audiobot
tail -f /home/debian/audiobot/bot.log
```
