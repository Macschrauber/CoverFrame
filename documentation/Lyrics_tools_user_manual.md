# Lyrics Tools User Manual

**CoverFrame Lyrics Fetcher & Scanner**  
Automated lyrics retrieval and management for your music library

---

## Overview

The CoverFrame lyrics tools consist of two Bash scripts that work together to automatically fetch, organize, and embed song lyrics for your music collection:

- **`lyrics_fetch`** - Core lyrics fetching engine (supports multiple sources)
- **`lyrics_scan`** - Batch processor for scanning music directories

### Key Features

✅ **Two Lyrics Sources** (no API keys required!)
- [lrclib.net](https://lrclib.net) - Time-synced lyrics (LRC format)
- QQ Music - Chinese lyrics database with international coverage

✅ **Genius** (API key required, you can get it for free)
- Genius (optional) - Popular lyrics website

✅ **Time-Synced Lyrics** - Prefers `.lrc` format with millisecond timestamps  
✅ **Smart Album Matching** - Jaccard similarity for accurate results  
✅ **Parallel Fetching** - Queries all sources simultaneously for speed  
✅ **Organized Archive** - Stores lyrics by `Artist/Album - Track.lrc`  
✅ **Quality Verification** - Trust level scoring to detect incorrect lyrics  
✅ **ID3 Tag Support** - Optional embedding via `kid3-cli`  
✅ **Live Album Detection** - Falls back to mainstream lyrics for live recordings  
✅ **Bash 3.2 Compatible** - Works on macOS without upgrades

---

## Requirements

### System Requirements
- **macOS** (tested on Bash 3.2+)
- **curl** - HTTP requests
- **jq** - JSON parsing
- **kid3-cli** (optional) - For embedding lyrics in ID3 tags
- **bc** - Floating point math (standard on macOS)

### Installation

part of CoverFrame, gets installed with `install.sh`

---

## Configuration

### Directory Structure

Edit these paths at the top of `lyrics_scan`:

```bash
LYRICS_FETCH="$HOME/Sites/scripts/lyrics_fetch"       # Path to lyrics_fetch
LYRICS_DIR="$HOME/Sites/lyrics"                       # Lyrics archive
NO_LYRICS_DIR="$HOME/Sites/no_lyrics"                 # Tracks without lyrics
SUSPICIOUS_LYRICS_DIR="$HOME/Sites/suspicious_lyrics" # Low-quality results
```

### Optional: Genius API Setup

**Note:** Genius is completely optional! The tools work great without it using free sources like lrclib.net and QQ Music.

If you want to add Genius as an additional source:

1. Create an account at [genius.com](https://genius.com)
2. Get your API credentials from: [https://genius.com/api-clients](https://genius.com/api-clients)
3. Create `genius_api_key.txt` in the same directory as the scripts:

```bash
# genius_api_key.txt
GENIUS_API_KEY="your_api_key_here"
GENIUS_CLIENT_ID="your_client_id_here"
```

Both scripts will automatically load this file if present.

---

## Usage

### lyrics_scan - Batch Processor

**Syntax:**
```bash
./lyrics_scan [OPTIONS] [PATHS...]
```

**Options:**

| Option | Description |
|--------|-------------|
| `-skip_existing_lyrics` | Don't re-scan files that already have lyrics |
| `-dont_skip_existing_lyrics` | Re-scan plain text lyrics to find time-synced versions |
| `-writetags` | Embed lyrics in ID3 tags using kid3-cli |
| `-only_live_albums` | Only process albums with "Live" in the name |
| `-renew` | Force refresh (moves old lyrics to `lyrics_delete/`) |
| `-undo_renew` | Restore lyrics from `lyrics_delete/` backup |
| `-recheck_link_only_lyrics` | Re-fetch lyrics that are stored as Genius URLs |
| `-rescan_lyrics_with_no_chapters` | Re-scan lyrics without LRC timestamps |

**Examples:**

```bash
# Scan a directory, skip existing lyrics
./lyrics_scan -skip_existing_lyrics "/Volumes/Music/New Albums"

# Scan and upgrade plain text to time-synced lyrics
./lyrics_scan -dont_skip_existing_lyrics "/Volumes/Music/AC DC"

# Scan and embed lyrics in ID3 tags
./lyrics_scan -writetags "/Volumes/Music/Iron Maiden"

# Scan multiple directories
./lyrics_scan "/Volumes/Music/2024" "/Volumes/Music/2025"

# Force refresh all lyrics (creates backup)
./lyrics_scan -renew "/Volumes/Music/Various Artists"

# Restore from backup
./lyrics_scan -undo_renew "/Volumes/Music/Various Artists"
```

### lyrics_fetch - Direct Fetcher

**Syntax:**
```bash
./lyrics_fetch [OPTIONS] "Artist" "Title" ["Album"]
```

**Options:**

| Option | Description |
|--------|-------------|
| `-t` | Prefer time-synced lyrics (LRC format) |
| `-a` | Album must match (strict mode) |
| `-r` | Accept mainstream lyrics if album doesn't match |
| `-v` | Verbose output (debug mode) |
| `-m` | Test mode - run all fetchers and compare |

**Environment Variables:**

| Variable | Description | Example |
|----------|-------------|---------|
| `FETCHERS_OVERRIDE` | Force specific fetcher | `fetch_from_lrclib_v3` |

**Examples:**

```bash
# Fetch time-synced lyrics with album matching
./lyrics_fetch -t "AC/DC" "Let There Be Rock" "Let There Be Rock"

# Fetch with strict album matching
./lyrics_fetch -t -a "Iron Maiden" "The Number Of The Beast" "The Number Of The Beast"

# Test mode - compare all sources
./lyrics_fetch -m "Pink Floyd" "Comfortably Numb" "The Wall"

# Force specific fetcher
FETCHERS_OVERRIDE="fetch_from_lrclib_v3" ./lyrics_fetch "AC/DC" "Thunderstruck"

# Accept mainstream lyrics for live albums
./lyrics_fetch -r "AC/DC" "Let There Be Rock" "Live At River Plate"

# Verbose debugging
./lyrics_fetch -v -t "Metallica" "Master Of Puppets" "Master Of Puppets"
```

---

## How It Works

### Fetching Process

1. **Parallel Execution**: All enabled fetchers run simultaneously
2. **Early Exit**: First time-synced result stops other fetchers
3. **Quality Scoring**: Calculates "trust level" based on title word matching
4. **Smart Fallback**: Uses best available source if no perfect match

### Fetcher Priority

**Time-Synced Mode** (`-t`):
```
1. LRCLIB (time-synced + plain text, no registration)
2. QQ Music (time-synced + plain text, no registration)
3. Genius (time-synced + plain text, optional)
```

**Plain Text Mode** (default):
```
1. Genius (best plain format, optional)
2. LRCLIB (also has plain lyrics)
3. QQ Music (if required, lyrics_scan is transcoding)
```

### Album Matching

The tools use **Jaccard similarity** to compare album names:

```
Similarity = (Words in common) / (Total unique words)
```

**Example:**
- Search: "Let There Be Rock"
- Found: "Let There Be Rock (Live)"
- Similarity: 80% ✅ (4 words match out of 5 total)

Thresholds:
- **100%** = Perfect match (high confidence)
- **80%+** = Good match (used with verification)
- **50%+** = Acceptable match (flagged for review)
- **<50%** = Rejected (unless `-r` flag used)

### QQ Music Validation

QQ Music often returns search results in Chinese, Korean, or other languages, which can make verification challenging. Instead of attempting translation, the tool uses a clever validation approach:

**The Problem:**
```
Search: "Iron Maiden - Fear of the Dark"
QQ Returns: "铁娘子乐队 - 黑暗恐惧" (Chinese characters)
Question: Is this the correct song?
```

**The Solution:**

1. Each QQ Music result has a unique `songmid` (song ID)
2. Fetch the actual **lyrics** for that songmid
3. Extract the album name from the LRC metadata tag: `[al:Album Name]`
4. Compare this metadata album with your search album
5. If they match → Accept! If not → Reject and try next candidate

**Why This Works:**

- The `songmid → lyrics` mapping is authoritative
- LRC metadata tags often contain the original album name (frequently in English)
- No translation needed - QQ Music validates itself
- Catches wrong songs, bad uploads, and search algorithm errors
- Works for any language combination

**Example:**
```
Search result: "아이언 메이든 - 어둠의 공포" (Korean)
Fetch lyrics for that songmid...
Metadata shows: [al:Fear of the Dark]
Compare: "Fear of the Dark" = 100% match ✅
Result: Validated!
```

This prevents fetching lyrics for the wrong song when QQ Music returns mixed-language results or search algorithm mistakes.

### Quality Verification

Every fetched lyric gets a **trust level** (0-100):

```
Trust = (Title words found in lyrics) / (Total title words) × 100
```

**Trust Levels:**
- **100** = All title words present (high confidence)
- **50-99** = Partial match (acceptable)
- **0-49** = Low match → saved to `suspicious_lyrics/`
- **0** = No title words found → offers alternatives

**Example:**
```
Title: "The Rolling Stones – Sympathy for the Devil"
Fill words removed: "for", "the"
Lyrics contain: "Sympathy" (1/2 words)
Trust Level: 50%
```

### Live Album Handling

For albums containing "Live", "Concert", or "Bootleg":

1. Try exact live album match
2. If no match, fall back to **mainstream studio version**
3. Prefix lyrics with: `[mainstream lyrics for a live album]`

---

## Output & Logging

### Statistics

After each run, `lyrics_scan` displays:

```
------------------------
no_lyrics:         45   ← No lyrics available from any source
plain_text_lyrics: 23   ← Plain text lyrics saved
mainstream_lyrics: 8    ← Studio versions for live albums
time_code_lyrics:  134  ← Time-synced LRC files
suspicious_lyrics: 3    ← Low trust level (flagged for review)
------------------------
```

### Color Coding

- 🔵 **Blue** - Time-synced lyrics found/saved
- 🟢 **Green** - Plain text lyrics found/saved
- 🟡 **Yellow** - Warnings, fallbacks, or processing info
- 🔴 **Red** - No lyrics found or errors

### Archive Organization

```
~/Sites/
├── lyrics/
│   ├── AC_DC/
│   │   └── Let There Be Rock - 01 Go Down.lrc
│   ├── Iron Maiden/
│   │   └── The Number Of The Beast - 06 The Number Of The Beast.lrc
│   └── Pink Floyd/
│       └── The Wall - 13 Comfortably Numb.lrc
│
├── no_lyrics/
│   └── AC_DC/
│       └── 12 Deep in the Hole.mp3 → /Volumes/Music/...
│
└── suspicious_lyrics/
    └── Metallica/
        └── Master Of Puppets - 03 Master Of Puppets_suspicious_genius.lrc
```

---

## Advanced Features

### Source Attribution

Lyrics files include source attribution in the first line:

**Time-Synced:**
```
[re:lrclib]
[00:12.34] First line of lyrics...
```

**Plain Text:**
```
# genius
First line of lyrics...
```

### Alternative Lyrics

When multiple sources disagree, better matches are saved as:
```
Song Name_lrclib.lrc     ← Alternative from lrclib
Song Name_qqmusic.lrc    ← Alternative from QQ Music
Song Name_genius.lrc     ← Alternative from Genius
```

Original low-quality version moved to `suspicious_lyrics/`.

### Instrumental Detection

If Genius URL contains "this song is an instrumental":
```
[Instrumental]
```

No further fetching attempted.

### Symlink Support

`lyrics_scan` resolves:
- Filesystem symlinks
- macOS Finder aliases (via `alias_resolve` if available)

---

## Troubleshooting

### Common Issues

**Problem:** "lyrics_fetch not found"  
**Solution:** Edit `LYRICS_FETCH` path in `lyrics_scan` to point to the script location

**Problem:** "jq: command not found"  
**Solution:** `brew install jq`

**Problem:** All lyrics showing as "no lyrics found"  
**Solution:** Check internet connection. Try verbose mode: `./lyrics_fetch -v -t "Artist" "Title"`

**Problem:** Wrong lyrics fetched  
**Solution:** Use `-a` flag for strict album matching: `./lyrics_fetch -t -a "Artist" "Title" "Album"`

**Problem:** Getting lyrics for studio version instead of live  
**Solution:** Use `-r` flag to accept mainstream lyrics for live albums

**Problem:** Genius not working  
**Solution:** Genius is optional! Check if `genius_api_key.txt` exists and has valid credentials. Or just use the free sources (lrclib, QQ Music).

### Debug Mode

Run with verbose flag to see detailed fetching process:

```bash
./lyrics_fetch -v -t "Artist" "Title" "Album" 2>&1 | tee debug.log
```

Check the debug output for:
- API responses
- Similarity calculations
- Trust levels
- Source selection logic

---

## Performance

### Speed

- **Parallel fetching**: All sources queried simultaneously
- **Early exit**: Stops on first high-quality time-synced result
- **Typical fetch time**: 2-5 seconds per song
- **Batch processing**: ~200-500 songs/hour (depending on hit rate)

### Caching

Results are cached in the lyrics archive:
- Re-running `lyrics_scan` with `-skip_existing_lyrics` is instant
- Use `-dont_skip_existing_lyrics` to upgrade plain → time-synced

---

## Integration with CoverFrame

These tools are part of the **CoverFrame** music management system:

- **lyrics_scan** - Populates lyrics archive
- **CoverFrame** - Displays time-synced lyrics on album covers
- **cover_music_player** - Shows scrolling lyrics during playback

The `.lrc` format enables frame-accurate lyric display synchronized with audio.

---

## Technical Details

### Bash 3.2 Compatibility

Written for **macOS Bash 3.2** (the version included with macOS):

- No associative arrays (Bash 4+ feature)
- Uses indexed arrays with safe `${array[@]:-}` syntax
- Compatible with `set -euo pipefail` strict mode
- Works on macOS without requiring Bash upgrade

### Error Handling

Scripts use **strict mode** for reliability:

```bash
set -euo pipefail
```

- `-e` - Exit on error
- `-u` - Error on unbound variables
- `-o pipefail` - Catch pipe failures

All array operations protected against "unbound variable" errors.

### UTF-8 Safety

macOS `sed` requires special handling:

```bash
# Incorrect (may corrupt UTF-8)
sed 's/pattern/replacement/'

# Correct (preserves UTF-8)
LC_ALL=C sed 's/pattern/replacement/'
```

Scripts handle international characters correctly.

---

## FAQ

**Q: Do I need a Genius API key?**  
A: No! Genius is optional. The tools work great with just lrclib.net and QQ Music (both free, no registration). However, Genius delivers high quality lyrics in plain and time-synced mode"

**Q: Why prefer time-synced lyrics?**  
A: LRC format enables frame-accurate display in music players like CoverFrame.

**Q: Can I use this with iTunes/Music.app?**  
A: Yes! Use the `-writetags` option to embed lyrics in ID3 tags.

**Q: What if lyrics are in the wrong language?**  
A: QQ Music sometimes returns Chinese lyrics. Use `FETCHERS_OVERRIDE="fetch_from_lrclib_v3"` to prefer English sources.

**Q: How do I handle duplicate albums (Deluxe, Remaster, etc.)?**  
A: Use `-a` flag for strict album matching to ensure correct version.

**Q: Can I run this on my entire music library?**  
A: Yes! Use `-skip_existing_lyrics` to avoid re-processing: `./lyrics_scan -skip_existing_lyrics "/Volumes/Music"`

**Q: What happens to lyrics I already have?**  
A: With `-skip_existing_lyrics`, existing files are untouched. Without it, plain text lyrics may be upgraded to time-synced.

---

## Version History

**lyrics_scan v317** (Jan 6, 2026)
- Array safety fixes for `set -euo pipefail` compatibility
- Fixed JSON truncation in QQ Music fetcher
- Improved Jaccard similarity calculation
- Better handling of array operations in Bash 3.2

**lyrics_fetch v313** (Jan 6, 2026)
- Major stability improvements
- Fixed unbound variable errors
- Enhanced parallel fetcher reliability

---

## Credits

**Author:** CoverFrame Project  
**License:** MIT (implied by open source nature)  
**Dependencies:**
- [lrclib.net](https://lrclib.net) - Free lyrics API
- QQ Music - International lyrics database
- [Genius](https://genius.com) - Optional lyrics source
**AI connection:**
- for those who care:
- made with help of AIs, Claude, Gemini, Perplexity, chatgpt, whatever was helpful. We are in 2026.

---

## Contributing

Found a bug? Have a feature request?

1. Check existing issues
2. Provide sample artist/title that fails
3. Include verbose debug output
4. Note your OS version and Bash version

---

## Support

For issues or questions:
- Check this manual first
- Run with `-v` verbose flag for debugging
- Check the `suspicious_lyrics/` folder for quality issues
- Review the colored terminal output for hints

Happy lyrics hunting! 🎵

