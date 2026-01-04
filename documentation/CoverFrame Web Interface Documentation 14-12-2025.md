# CoverFrame Web Interface Documentation

## Overview

CoverFrame is a web-based remote control interface for a music player server. It displays album artwork and provides intuitive touch/mouse/keyboard controls for playback, navigation, volume adjustment, and lyrics management.

**Version:** v30h  
**Compatibility:** Modern browsers, iOS 10+ (iPad 4 tested), touch and desktop devices

---

## Features

### Core Functionality
- **Album Cover Display** - Full-screen album artwork with smooth transitions
- **Playback Control** - Play/pause, skip tracks, seek within songs
- **Volume Control** - Adjust volume via top/bottom zones
- **Lyrics Management** - View, navigate, calibrate, and renew synced lyrics
- **Multi-Input Support** - Touch gestures, mouse clicks, and keyboard shortcuts
- **Dynamic Polling** - Adaptive update rate (200ms for synced lyrics, 2s otherwise)
- **Server Health Monitoring** - Automatic detection and reconnection prompts

### Visual Feedback
- **VERBOSE Mode** - Toggle on-screen command overlays for debugging (`VERBOSE = true`)
- **Brightness Dimming** - Visual feedback when commands are sent (70% dim for 250ms)
- **Cover Flow Animation** - Smooth slide transitions when tracks change

---

## Control Methods

### 1. Touch Controls (Mobile/Tablet)

#### Swipe Gestures (Middle 60% of Screen)
| Gesture | Action |
|---------|--------|
| **Swipe Left** | Skip to next track or lyrics page* |
| **Swipe Right** | Previous track or lyrics page* |
| **Swipe Up** | Show lyrics |
| **Swipe Down** | Show track info |

*Automatically switches between track/lyrics navigation when lyrics are displayed

#### Tap Gestures (Middle 60% of Screen)
| Gesture | Action |
|---------|--------|
| **Short Tap** (< 400ms) | Play/Pause (500ms delay to prevent accidental triggers) |
| **Long Press** (400-2000ms) | Show track info |
| **Extra Long Press** (≥ 2000ms) | Renew lyrics from source |

#### Zone Taps
| Zone | Location | Action |
|------|----------|--------|
| **Top 20%** | Full width | Volume up |
| **Bottom 20%** | Full width | Volume down |
| **Mid-Left** | Left edge (5-25%) | Skip back 15 seconds |
| **Mid-Right** | Right edge (75-95%) | Skip forward 15 seconds |

**Note:** Swipe gestures in the middle 60% area (Y: 20%-80%) take priority. The mid-left/mid-right zones are temporarily disabled during swipes to prevent conflicts.

---

### 2. Mouse Controls (Desktop)

#### Click Zones
| Zone | Location | Action |
|------|----------|--------|
| **Top 20%** | Full width | Volume up |
| **Bottom 20%** | Full width | Volume down |
| **Left 25%** | Middle 60% height | Previous track or lyrics page* |
| **Right 25%** | Middle 60% height | Next track or lyrics page* |
| **Center** | Album cover | Play/Pause |

*Automatically switches between track/lyrics navigation when lyrics are displayed

---

### 3. Keyboard Shortcuts

#### Playback & Navigation
| Key | Action |
|-----|--------|
| **Space** | Play/Pause |
| **←** | Previous track or lyrics page* |
| **→** | Next track or lyrics page* |
| **↑** | Volume up |
| **↓** | Volume down |
| **Shift + ←** | Skip back 15 seconds |
| **Shift + →** | Skip forward 15 seconds |
| **Alt + ←** | Skip back 15 seconds |
| **Alt + →** | Skip forward 15 seconds |

#### Lyrics Management
| Key | Action |
|-----|--------|
| **L** | Toggle lyrics display |
| **N** | Next lyrics page |
| **B** | Previous lyrics page (Back) |
| **R** (Shift+R) | Renew lyrics from source |
| **U** (Shift+U) | Undo lyrics renewal |
| **C** (Shift+C) | Calibrate lyrics timing |
| **F** (Shift+F) | Fix lyrics synchronization |

#### System
| Key | Action |
|-----|--------|
| **I** | Show track info |
| **Ctrl + C** | Quit server (with confirmation) |

*Automatically switches between track/lyrics navigation when lyrics are displayed

---

## Technical Architecture

### Server Communication

#### Polling System
The interface uses **dynamic polling** to balance responsiveness and resource usage:

```javascript
POLL_FAST = 200ms    // When synced lyrics are active
POLL_NORMAL = 2000ms // Default state
```

**Polling Endpoints:**
1. **`cover.tag`** - Track change detection (returns unique track identifier)
2. **`brightness.tag`** - Volume level feedback (0-200 scale)
3. **`scripts/lyrics_synced.tag`** - Determines polling rate (HEAD request)

**Additional Endpoints:**
- **`coverflow.tag`** - Triggers cover flow animation
- **`scripts/lyrics_skip.tag`** - Checks if lyrics navigation mode is active
- **`message.html`** - Custom status messages (checked every 2s when stopped)

#### Command Transmission
Commands are sent via GET requests to `/command?cmd={command}&t={timestamp}`

**Anti-Duplicate Protection:**
- Pause commands have a 350ms cooldown to prevent double-triggers
- Commands during cover animations are blocked (skip/back only)

---

### State Management

#### Player States
| State | Trigger | UI Changes |
|-------|---------|------------|
| **Playing** | `cover.tag` returns 200 | Show album cover, hide message, enable controls |
| **Stopped** | `cover.tag` fails | Hide cover, show "Player is stopped" message |
| **Server Offline** | Initial connection fails | Show reconnection modal |

#### Visual State
- **`currentTag`** - Tracks current song to detect changes
- **`lastBrightnessValue`** - Caches volume level (prevents flicker)
- **`isAnimating`** - Blocks commands during cover transitions
- **`lastCommand`** - Stores last sent command (for debugging)

---

### Animation System

#### Cover Flow Transition
When a new track is detected:
1. Load new cover into `next-cover` element (off-screen right)
2. Slide `current-cover` out to the left (scale down to 0.8, fade out)
3. Slide `next-cover` in from the right (scale up to 1.0, fade in)
4. Swap element references and reset positions
5. **Re-attach event listeners** to the new current cover

**Timing:** 800ms total (50ms delay + 750ms transition)

---

### Touch Gesture Detection

#### Swipe Recognition Algorithm
```javascript
minSwipeDistance = 50px       // Minimum distance to register
maxSwipeTime = 1000ms         // Maximum duration
maxVerticalDeviation = 100px  // Max Y-axis drift for horizontal swipes
```

**Horizontal Swipe:**
- `|deltaX| ≥ 50px` AND `|deltaY| ≤ 100px`

**Vertical Swipe:**
- `|deltaY| ≥ 50px` AND `|deltaX| ≤ 100px`

**Tap/Press Detection:**
- Duration < 400ms → Short tap (pause after 500ms delay)
- Duration 400-2000ms → Long press (info)
- Duration ≥ 2000ms → Extra long press (lyrics renew)

---

### Device Detection

```javascript
hasTouchSupport = ('ontouchstart' in window) || (navigator.maxTouchPoints > 0)
```

**Touch Devices:**
- Enable swipe area, mid-left/mid-right zones
- Disable mouse click zones (left/right 25%)

**Desktop:**
- Enable mouse click zones
- Disable swipe detection

---

## Advanced Features

### Lyrics Context Switching
The interface automatically detects when lyrics are displayed and switches navigation behavior:

1. Check for `scripts/lyrics_skip.tag` file
2. If present: Arrow keys/swipes navigate lyrics pages
3. If absent: Arrow keys/swipes navigate tracks

**Function:** `checkLyricsSkipTagAndSend(backCmd, skipCmd)`

---

### Brightness Feedback
Volume changes are reflected visually via CSS brightness filter:

```javascript
brightness.tag → 0-200 scale → CSS filter: brightness(0-200%)
```

**Dim Effect on Commands:**
- Briefly dims cover to 70% brightness for 250ms
- Provides tactile feedback that command was sent

---

### Server Offline Handling
On startup or connection loss:
1. Check `/cover.tag` endpoint
2. If timeout (2s) or error → Show reconnection modal
3. User clicks "Retry" → Re-run startup check
4. On success → Hide modal and start polling loop

---

## Configuration

### Adjustable Parameters

#### Swipe Sensitivity
```javascript
var minSwipeDistance = 50;        // Minimum pixels to trigger
var maxSwipeTime = 1000;          // Maximum milliseconds
var maxVerticalDeviation = 100;   // Max Y drift for horizontal swipes
```

#### Polling Rates
```javascript
var POLL_FAST = 200;    // Fast polling (synced lyrics)
var POLL_NORMAL = 2000; // Normal polling
```

#### Visual Feedback
```javascript
var VERBOSE = true;     // Show command overlays
var DIM_FACTOR = 0.7;   // Dim to 70% on command
```

#### Pause Protection
```javascript
var pauseCooldownMs = 350;  // Prevent duplicate pause commands
```

---

## Browser Compatibility

### Tested Platforms
- ✅ iOS 10+ (iPad 4)
- ✅ Modern Chrome/Firefox/Safari
- ✅ Touch and desktop devices

### Compatibility Features
- Manual XHR timeouts (older iOS doesn't support native timeout)
- Fallback keyCode detection for legacy keyboards
- Touch event passive flag handling
- Prevent iOS text selection and callouts

---

## Debugging

### VERBOSE Mode
Enable command overlay for visual feedback:
```javascript
var VERBOSE = true;  // Set to false to disable
```

Shows command name in large overlay for 1 second after each action.

### Console Logging
Key events are logged:
- `sendCommand(cmd)` - Every command transmission
- Animation state transitions
- Duplicate pause suppressions
- Touch event coordinates (for gesture tuning)

---

## Security & Performance

### Request Optimization
- Timestamped URLs prevent caching (`?t={Date.now()}`)
- HEAD requests for flag files (no body transfer)
- Batch polling (2 concurrent requests max)
- Abort pending requests on timeout

### Input Sanitization
- No user input is accepted (read-only interface)
- Commands are predefined strings (no injection risk)

### Resource Management
- Single polling loop (no interval conflicts)
- Cleanup of timeouts on state changes
- Event listener cleanup on cover swaps

---

## Customization Guide

### Adding New Commands

1. **Add keyboard shortcut:**
```javascript
else if (key === 'x') sendCommand('your_command');
```

2. **Add touch zone** (in HTML):
```html
<div id="zone-custom" class="zone"></div>
```

3. **Style the zone** (in CSS):
```css
#zone-custom {
  top: 40%; left: 40%; width: 20%; height: 20%;
}
```

4. **Attach event listener:**
```javascript
zoneCustom.addEventListener('click', function() {
  sendCommand('your_command');
});
```

### Adjusting Touch Zones
Modify percentage values in CSS:
```css
#zone-up {
  height: 20% !important;  /* Change this value */
}
```

---

## Known Limitations

1. **Browser Navigation Conflicts** - Some Ctrl+Arrow combinations may be intercepted by browsers for navigation (tested: doesn't work reliably)
2. **Alt+Arrow Fallthrough** - Alt+Arrow may trigger plain arrow handlers on some browsers (fixed with early returns)
3. **iOS Safari Quirks** - Long press may trigger text selection (prevented via CSS)

---

## License & Credits

**CoverFrame Web Interface**  
Album cover display and remote control for music player servers.

Built with vanilla JavaScript for maximum compatibility and minimal dependencies.