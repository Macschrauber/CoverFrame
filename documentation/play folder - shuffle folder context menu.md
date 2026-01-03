# CoverFrame Update - December 25, 2024

## 🎯 New Feature: Folder Context Menus

Added right-click/long-press context menus for folders in the browser view, enabling multi-device playback control.

---

## ✨ What's New

### Folder Context Menu
- **Right-click (desktop)** or **long-press (touch)** any folder to show context menu
- **Two actions available:**
  - `Play Folder` - Plays folder contents in order
  - `Shuffle Folder` - Plays folder contents shuffled
- Works on folders at any level (including "..parent directory")
- Commands sent directly to server via XMLHttpRequest (iOS 9 compatible)

### Multi-Device Control
- Any browser can now control playback without needing `browser.html`
- Use old iPhones/iPads as dedicated remote controls
- Navigate to folder → long-press → play/shuffle
- Independent of main display device

---

## 🔧 Technical Changes

### Core Improvements
1. **JavaScript always generated** - Fixed issue where folders without audio files had no JavaScript/event handlers
2. **Dual event handling** - Context menus work on both desktop (click) and touch devices (touchend)
3. **Folder detection** - Added `data-folder="true"` attribute to all folder links
4. **Null safety** - Wrapped player-specific code in existence checks to prevent errors on folder-only pages

### Files Modified
- `CoverFrame_server` - Main HTTP server with embedded HTML/JavaScript generation

### Key Code Additions
- `createFolderContextMenu()` - Creates folder menu with permanent handlers
- `showFolderContextMenu()` - Displays menu at click/touch position  
- `callPlayFolder()` - Sends `play <path>` command via XMLHttpRequest
- `callShuffleFolder()` - Sends `shuffle <path>` command via XMLHttpRequest
- Global `contextmenu` event handler prevents default browser menu on folders
- Touch handlers (touchstart/touchmove/touchend) for long-press detection

---

## 🐛 Bugs Fixed

1. **Empty playlist folders** - JavaScript wasn't generated when folders contained no audio files
2. **Null reference errors** - `audio.onended` crashed when no player element existed
3. **Desktop clicks ignored** - Context menu handlers only accepted touchend, not click events
4. **Orphaned duplicate code** - Removed leftover code from incomplete refactoring
5. **Syntax errors** - Fixed `var currentIndex++` and other JavaScript syntax issues

---

## 🎨 User Experience

### Before
- Had to navigate into folders to see/play contents
- Required `browser.html` for remote control
- Only audio files had context menus

### After  
- Right-click folders to instantly play/shuffle
- Any browser = instant remote control
- Consistent UX across folders and audio files
- Works on iOS 9.3+, modern iOS, and desktop browsers

---

## 📱 Compatibility

- ✅ **iOS 9.3+** - Uses XMLHttpRequest (no fetch API)
- ✅ **Modern iOS/iPadOS** - Full touch gesture support
- ✅ **Desktop Safari/Chrome/Firefox** - Right-click context menus
- ✅ **macOS 10.9+** - Bash 3.2 compatible server
- ✅ **Intel & Apple Silicon** - Universal compatibility

---

## 🚀 Use Cases Enabled

1. **Quick folder playback** - Navigate, right-click, play (3 clicks vs many)
2. **Multiple control points** - iPad displays cover art, iPhone controls playback
3. **Guest access** - Give visitors simple URL, they can browse and play
4. **Repurpose old devices** - iPhone 5/6/7 as dedicated music remotes
5. **Kitchen/bedroom control** - Control music from any room with a browser

---

## 📝 Known Limitations

- Requires JavaScript enabled
- Context menu positioned at click/touch point (may go off-screen on edges)
- Debug logging still active (can be removed later)
- HTML/JavaScript embedded in Python (refactoring to separate files recommended)

---

## 🔜 Future Enhancements (Ideas)

- [ ] Extract HTML/JS to separate template files for easier editing
- [ ] Add "Add to Queue" folder option
- [ ] Remember last played position in folders
- [ ] Folder thumbnail previews in context menu
- [ ] Keyboard shortcuts (e.g., Space to play selected folder)

---

## 🙏 Credits

Developed through collaborative debugging on **December 25, 2024**  
Tested on macOS (desktop) and iOS 9 (touch device)