# CoverFrame

**Turn your old iPad or other display device into a living album cover artwork frame**

![Stargazer time synced](screenshots/stargaizer%20time%20synced.png)

CoverFrame transforms any old display device—iPad, Android tablet, old iPhone, or spare monitor—into a beautiful, living album artwork frame that syncs with your music playback. Watch your album covers come alive with synchronized lyrics, track information, and stunning visuals.

## What It Does

- **Living Album Art**: Displays high-quality album artwork that updates as your music plays
- **Synchronized Lyrics**: Shows time-synced lyrics that scroll with the music
- **Touch Control**: Control playback with simple touch gestures on your display device
- **Works with Old Devices**: Supports devices as old as iOS 9.3 and macOS 10.9
- **Local Music Focus**: Designed for your manually organized music collection, not streaming services

## What You Need

### macOS Server (Music Player)
- **macOS**: 10.9 Mavericks or later
- **Processor**: Intel or Apple Silicon
- **Music Player**: Vox Player (recommended) or any music player
- **Dependencies**: 
  - Python 3 (built-in on macOS 10.15+)
  - ImageMagick (for album art processing)
  - kid3-cli (for reading ID3 tags)
  - **Lyrics sources** (all free, no registration):
    - lrclib.net (time-synced lyrics) - included
    - Genius.com (optional) - requires free API key
    - Other services - included

### Display Device (Remote Frame)
- **iOS Device**: iOS 9.3 or later (iPad, iPhone, iPod Touch)
- **Android Device**: Any modern browser
- **Computer**: Any device with a web browser
- **Connection**: Same Wi-Fi network as your macOS server

## Installation

### Quick Start

1. **Download** the latest release
2. **Extract** the archive
3. **Open Terminal** and navigate to the folder:
   ```bash
   cd ~/Downloads/CoverFrame
   ```
4. **Run the installer**:
   ```bash
   bash install.sh
   ```
5. **Follow the prompts** - the installer handles everything:
   - Checks for dependencies
   - Installs missing components (ImageMagick, kid3-cli)
   - Handles architecture-specific issues (ARM64/Intel)
   - Deploys files to `~/Sites`

### Package Managers

The installer supports:
- **MacPorts** (recommended for older macOS < 11)
- **Homebrew**
- **Manual installation** (if no package manager available)

### After Installation

1. **Organize your music** in `~/Sites/music`
   - Manual organization required
   - Not designed for iTunes/Music.app database
   - Artist folders → Album folders → Music files

2. **Start CoverFrame**:
   ```bash
   ~/Sites/CoverFrame_starter
   ```

3. **Open on your display device**:
   - Connect to the same Wi-Fi network
   - Open browser to: `http://YOUR_MAC_IP:8000`
   - For pre iOS 9.3: Use `http://YOUR_MAC_IP:8000/indexios4.html`

## Lyrics Sources

CoverFrame fetches lyrics from **multiple free sources** - no registration required for basic use!

### Included (No Setup Needed)
- **lrclib.net**: Time-synced lyrics database (recommended, works out of the box)
- **Other services**: Additional free lyrics sources

### Optional Enhancement
- **Genius.com**: Popular lyrics website
  - Provides additional lyrics coverage
  - Requires free API key (one-time setup)
  - See `genius_api_key.txt.template` after installation
  - **Not required** - CoverFrame works great without it!

Most users won't need Genius - the included sources cover most songs with time-synced lyrics.

## Features

### Album Artwork Display
- High-resolution cover art
- Smooth transitions
- Automatic scaling for any display size
- Background blur effects

### Lyrics Engine
- Fetches lyrics from ID3 tags or local cache folder
- Time-synchronized display
- Manual calibration tools
- Page navigation for long lyrics

### Touch Gestures
- Play/Pause: Tap center
- Next/Previous: Swipe left/right
- Volume: Swipe up/down
- Skip 15 seconds: Two-finger swipe
- See full gesture documentation in `/mnt/project/CoverFrame_touch_zones_16-12-2025.md`

### Playback Control
- Shuffle mode
- Single song repeat
- Volume control
- Track information display

## Compatibility

### Tested On
- **macOS**: 10.14 Mojave, 11 Big Sur, 12 Monterey, 13 Ventura, 14 Sonoma, 15 Sequoia
- **iOS**: 9.3, 10, 11, 12, 13, 14, 15, 16, 17, 18
- **Processors**: Intel x86_64, Apple Silicon ARM64
- **Display Devices**: iPad (all generations), iPhone, Android tablets, desktop browsers

### Special Handling
- **macOS < 11**: Installer detects Qt6 incompatibility and links Qt5 kid3-cli
- **Apple Silicon**: Installer prefers ARM64 binaries for native performance
- **Old iOS**: Special interface (`indexios4.html`) for iOS older than 9.3

## Architecture

```
┌─────────────────┐         ┌──────────────────┐
│  macOS Server   │         │  Display Device  │
│  (Music Player) │◄────────┤   (Web Browser)  │
│                 │  HTTP   │                  │
│  • Vox Player   │  :8000  │  • Album Art     │
│  • CoverFrame   │────────►│  • Lyrics        │
│  • HTTP Server  │         │  • Touch Control │
└─────────────────┘         └──────────────────┘
```

## Project Structure

```
CoverFrame/
├── install.sh                 # Main installer
├── Sites/
│   ├── CoverFrame_starter     # Launch script
│   ├── CoverFrame_server      # HTTP server
│   ├── index.html             # Main interface
│   ├── indexios4.html         # iOS 9.3-11 interface
│   └── browser.html           # Desktop browser interface
├── scripts/                   # Control scripts (play, pause, etc.)
├── tools/                     # Utilities (position adjuster, shadowcopy)
└── usr-local-bin/             # Helper binaries
```

## Documentation

- **Installation**: This README
- **Touch Gestures**: `CoverFrame_touch_zones_16-12-2025.md`
- **Web Interface**: `CoverFrame_Web_Interface_Documentation_14-12-2025.md`
- **Changelog**: `CoverFrame_server_and_browser_html_Changelog_20-12-2025.md`
- **Technical Analysis**: `CoverFrame_Complete_Technical_Analysis.md`

## Development Philosophy

CoverFrame is designed with these principles:

1. **Simplicity**: No complex setup, just music and artwork
2. **Longevity**: Works with devices from 2015 onwards
3. **Local-First**: Your music, your control, no cloud dependencies
4. **Manual Curation**: Designed for people who organize their music
5. **Lightweight**: Minimal dependencies, runs on old hardware

## Known Limitations

- **macOS Only**: Server requires macOS (music player integration)
- **Manual Organization**: No iTunes/Music.app database integration
- **Same Network**: Requires server and display on same Wi-Fi
- **Vox Player**: Best experience with Vox (other players via scripts)

## Troubleshooting

### kid3-cli not found
```bash
# Check status
which kid3-cli

# MacPorts (recommended for macOS < 11)
sudo port install kid3

# Homebrew
brew install kid3

# Manual (for macOS < 11, use Qt5 version)
# Download from: https://sourceforge.net/projects/kid3/files/kid3/3.9.7/
```

### ImageMagick not found
```bash
# MacPorts
sudo port install ImageMagick

# Homebrew
brew install imagemagick

# Manual
# Download from: https://imagemagick.org/script/download.php#macosx
```

### Display device won't connect
1. Verify same Wi-Fi network
2. Check firewall settings on macOS
3. Try `http://IP:8000` instead of hostname
4. For pre iOS 9.3-11, use `indexios4.html`

## License

[Your License Here]

## Author

Macschrauber (known for Macschrauber's Rom Dump, the major firmware tool for Intel Macs of that era)

## Acknowledgments

- Built with love for music lovers who appreciate album art
- Inspired by the desire to give old devices new life
- Thanks to the open-source community for ImageMagick, kid3, and jq

## Support

For issues, questions, or contributions:
- **Issues**: [GitHub Issues](your-repo-url/issues)
- **Discussions**: [GitHub Discussions](your-repo-url/discussions)

---

**Enjoy your living album art frame!** 🎵✨
