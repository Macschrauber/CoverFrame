# CoverFrame - Message Display System (Sentinel Method)
## Updated: December 29, 2025

---

## **Overview**

The message display system allows scripts to show text messages to users in cover view (index.html) instead of album covers. This uses a **sentinel value system** where `cover.tag = "-1"` signals message mode.

---

## **How It Works**

### **The Sentinel System:**

```
index.html checks cover.tag:
├─ cover.tag = "-1" → Show message.html (message mode)
└─ cover.tag = "/path" → Show cover.jpg (normal mode)
```

**Key Insight:** The value "-1" in cover.tag is the signal, not the absence of the file!

---

## **Why Sentinel vs. File Deletion?**

### **Old Method (broken):**
```bash
rm -f cover.tag              # Delete file
# Problem: Can't lock a file that doesn't exist!
# Background scripts recreate cover.tag → race condition
```

### **New Method (sentinel - works):**
```bash
echo "-1" > cover.tag        # Write sentinel value
lock cover.tag               # Now it CAN be locked!
# Background scripts blocked from overwriting → no race!
```

---

## **The Three Key Files**

Located in `~/Sites/`:

### **1. cover.tag**
- **Purpose:** Controls what index.html displays
- **Values:**
  - `"-1"` → Message mode (show message.html)
  - `"/path/to/album"` → Normal mode (show cover.jpg)
- **Can be locked:** YES (this is the key feature!)

### **2. message.html**
- **Purpose:** Contains the HTML message to display
- **Used when:** cover.tag = "-1"
- **Format:** Any HTML content
- **Can be locked:** YES (prevents accidental overwrite)

### **3. cover.jpg**
- **Purpose:** The album cover image
- **Used when:** cover.tag ≠ "-1"
- **Ignored when:** cover.tag = "-1"

---

## **To Display a Message**

### **Required Steps:**

```bash
# 1. Create message.html
echo "<h2 style=\"color: orange;\">Your Message</h2>" > ~/Sites/message.html

# 2. Set sentinel value in cover.tag
echo "-1" > ~/Sites/cover.tag

# 3. (Optional) Lock files to prevent overwrites
lock ~/Sites/cover.tag
lock ~/Sites/message.html

# Message is now visible! ✅
```

---

## **To Clear the Message**

### **Method 1: Just change cover.tag (message.html can stay)**
```bash
echo "/music/album" > ~/Sites/cover.tag
# Message disappears, cover appears
# message.html still on disk (ignored)
```

### **Method 2: Full cleanup**
```bash
unlock ~/Sites/cover.tag
unlock ~/Sites/message.html
echo "/music/album" > ~/Sites/cover.tag
rm -f ~/Sites/message.html
# Message deleted, cover appears
```

**Note:** Deleting message.html is optional! cover.tag value controls display.

---

## **The lock/unlock Functions**

### **Required in scripts:**
```bash
lock() {
    [ -f "$1" ] && chflags uchg "$1" && echoverbose "$1 locked" && return 0
    return 1
}

unlock() {
    [ -f "$1" ] && chflags nouchg "$1" && echoverbose "$1 unlocked" && return 0
    return 1
}
```

### **What they do:**
- `lock`: Sets macOS immutable flag (uchg) - file becomes read-only
- `unlock`: Removes immutable flag (nouchg) - file becomes writable

### **Why lock?**
Prevents background processes (like cover_slideshow) from overwriting the sentinel value or message during display.

---

## **Current Implementation**

### **1. play Script (v29-12-2025)**

**When folder is too big for playback:**

```bash
# Create message
unlock "$message_site"
echo "<h2 style=\"color: orange;\">Folder too big<br>Starting cover slideshow instead</h2>" > "$message_site"
lock "$message_site"

# Set sentinel and lock
unlock "$cover_tag"
echo "-1" > "$cover_tag"
lock "$cover_tag"

# Start slideshow (it will handle the timing)
cover_slideshow "$folder" &
exit 0
```

**Clean handoff:** play sets up message and exits, slideshow handles timing.

---

### **2. cover_slideshow Script (v29-12-2025)**

**At startup, checks for message mode:**

```bash
# Check if cover.tag contains "-1" sentinel
if [ -f "$cover_tag" ]; then
    cover_tag_content=$(cat "$cover_tag" 2>/dev/null | tr -d '[:space:]')
    if [ "$cover_tag_content" = "-1" ] && [ -f "$message_site" ]; then
        echo_yellow "Message display mode detected - waiting 3 seconds..."
        sleep 3
        
        # Clear message and unlock for slideshow
        unlock "$cover_tag"
        unlock "$message_site"
        rm -f "$message_site"
        echo_green "Message cleared, starting slideshow"
    fi
fi

# Continue with normal slideshow...
```

**Responsibility separation:** Slideshow owns the 3-second timing, not play.

---

### **3. index.html (v29-12-2025)**

**Polling logic:**

```javascript
var tag = cover.tag content

if (tag === "-1") {
    // Sentinel detected - show message
    if (currentTag !== "-1") {
        currentTag = "-1";
        setPlaying();
        showMessageHtml();  // Create overlay with message
    }
} else {
    // Normal cover path
    if (currentTag === "-1") {
        hideMessageHtml();  // Hide message overlay first
    }
    currentTag = tag;
    setPlaying();
    refreshCover();
}
```

**Display method:**
- Creates overlay div on top of cover images
- Hides cover images (display: none)
- Shows message in overlay
- When switching back: hides overlay, shows covers
- **Preserves DOM structure** (no innerHTML replacement)

---

## **Timeline Example**

### **User plays big folder:**

```
0.0s: User right-clicks folder → "Play Folder"
0.0s: Browser.html switches to cover mode
0.1s: play creates message.html + sets cover.tag = "-1" + locks both
0.1s: play starts cover_slideshow &
0.1s: play exits
0.2s: index.html polls, sees cover.tag = "-1"
0.2s: index.html displays message: "Folder too big..." ✅
0.2s: cover_slideshow starts, detects sentinel
0.2s: cover_slideshow: "Message display mode detected - waiting 3 seconds..."
3.2s: cover_slideshow unlocks files, deletes message
3.2s: cover_slideshow writes first album to cover.tag
3.3s: index.html polls, sees new path
3.3s: index.html hides message, shows first cover ✅
3.3s: Slideshow continues normally
```

---

## **Message Styling**

### **Standard Template:**

```html
<h2 style="color: orange;">
    Main Message<br>
    Secondary Line<br>
    <span style="font-size: 0.8em;">Detail/Path</span>
</h2>
```

### **Color Convention:**
- **Orange:** Status messages (shuffling, slideshow starting)
- **Red:** Warnings/errors
- **Green:** Success messages
- **Yellow:** Information/busy indicators

### **Example Messages:**

```html
<!-- Folder too big -->
<h2 style="color: orange;">
    Folder too big<br>
    Starting cover slideshow instead<br>
    <span style="font-size: 0.8em;">/music/Blues/B.B._King</span>
</h2>

<!-- Busy indicator -->
<div style="text-align: center;">
  <h1 style="color: #ffcc00; font-size: 3em;">⏳</h1>
  <h2 style="color: white;">Processing...</h2>
  <p style="color: #888;">Please wait</p>
</div>

<!-- Error message -->
<h2 style="color: red;">
    Error: Operation failed<br>
    <span style="font-size: 0.8em;">Check logs for details</span>
</h2>
```

---

## **Troubleshooting**

### **Message not appearing?**

**Check 1:** Is cover.tag set to "-1"?
```bash
cat ~/Sites/cover.tag
# Should show: -1
```

**Check 2:** Does message.html exist?
```bash
ls -la ~/Sites/message.html
# Should exist
```

**Check 3:** Is index.html loaded?
```
Message only shows in cover view, not directory view
```

---

### **Message doesn't clear?**

**Check 1:** Is cover.tag locked?
```bash
ls -lO ~/Sites/cover.tag | grep uchg
# If "uchg" present → locked
```

**Check 2:** Unlock it manually
```bash
chflags nouchg ~/Sites/cover.tag
echo "/music" > ~/Sites/cover.tag
```

---

### **Message stuck after changing cover.tag?**

**This was a bug in early versions!** Fixed in v29-12-2025.

**Problem:** Old version replaced cover-container innerHTML, destroying img elements.

**Solution:** New version uses overlay div, preserves img elements.

**Verify you have:** index.html v29-12-2025 or later.

---

## **Advanced Use Cases**

### **1. Simple Busy Indicator**

```bash
# Create once:
echo "<h2 style='color: yellow;'>⏳ Busy...</h2>" > ~/Sites/message_busy.html

# Use anytime:
cp ~/Sites/message_busy.html ~/Sites/message.html
echo "-1" > ~/Sites/cover.tag
lock ~/Sites/cover.tag

# Do work...

# Clear:
unlock ~/Sites/cover.tag
echo "/music" > ~/Sites/cover.tag
```

---

### **2. Progress Updates**

```bash
# Start
echo "<h2>Step 1/5: Scanning...</h2>" > ~/Sites/message.html
echo "-1" > ~/Sites/cover.tag
lock ~/Sites/cover.tag

# Update (cover.tag stays -1)
echo "<h2>Step 2/5: Processing...</h2>" > ~/Sites/message.html
sleep 2

echo "<h2>Step 3/5: Finalizing...</h2>" > ~/Sites/message.html
sleep 2

# Done
unlock ~/Sites/cover.tag
echo "/music/result" > ~/Sites/cover.tag
```

---

### **3. Multiple Message Templates**

```bash
# Create library:
echo "<h2 style='color: orange;'>Shuffling...</h2>" > ~/Sites/msg_shuffle.html
echo "<h2 style='color: yellow;'>Loading...</h2>" > ~/Sites/msg_loading.html
echo "<h2 style='color: red;'>Error!</h2>" > ~/Sites/msg_error.html

# Use different ones:
show_message() {
    cp ~/Sites/msg_$1.html ~/Sites/message.html
    echo "-1" > ~/Sites/cover.tag
}

show_message "loading"
# ... do work ...
echo "/music" > ~/Sites/cover.tag
```

---

### **4. Time-based Messages**

```bash
# Show for specific duration
show_timed_message() {
    local duration=$1
    local message=$2
    
    echo "<h2>$message</h2>" > ~/Sites/message.html
    echo "-1" > ~/Sites/cover.tag
    lock ~/Sites/cover.tag
    
    sleep $duration
    
    unlock ~/Sites/cover.tag
    echo "/music" > ~/Sites/cover.tag
    rm -f ~/Sites/message.html
}

# Usage:
show_timed_message 5 "Processing complete!"
```

---

## **Integration Pattern**

### **Standard pattern for any script:**

```bash
#!/bin/bash

site="$HOME/Sites"
message_site="$site/message.html"
cover_tag="$site/cover.tag"

lock() {
    [ -f "$1" ] && chflags uchg "$1" && return 0
    return 1
}

unlock() {
    [ -f "$1" ] && chflags nouchg "$1" && return 0
    return 1
}

# Cleanup on exit
trap cleanup EXIT

cleanup() {
    unlock "$cover_tag"
    unlock "$message_site"
    rm -f "$message_site"
}

# Main script
main() {
    # Show message
    echo "<h2>Processing...</h2>" > "$message_site"
    echo "-1" > "$cover_tag"
    lock "$cover_tag"
    lock "$message_site"
    
    # Do work...
    
    # Cleanup happens automatically via trap
}

main "$@"
```

---

## **Files Using Message System**

### **1. play (v29-12-2025)**
- **Message:** "Folder too big / Starting cover slideshow instead"
- **When:** Folder exceeds playback threshold
- **Duration:** 3 seconds (handled by cover_slideshow)
- **Cleanup:** cover_slideshow handles it

### **2. cover_slideshow (v29-12-2025)**
- **Role:** Detects message mode, waits 3 seconds, clears message
- **Message:** None (handles messages from other scripts)
- **Integration:** Checks for sentinel at startup

### **3. shuffle (future update needed)**
- **Current:** Uses old rm cover.tag method
- **Should update:** To use sentinel system
- **Message:** "shuffling /folder"

---

## **Best Practices**

### **DO:**
✅ Use sentinel value (-1) instead of deleting cover.tag  
✅ Lock cover.tag when displaying message (prevents overwrites)  
✅ Lock message.html when displaying (prevents overwrites)  
✅ Unlock before changing values  
✅ Use color conventions for message types  
✅ Keep messages concise and readable  

### **DON'T:**
❌ Delete cover.tag (can't lock what doesn't exist)  
❌ Leave files locked permanently  
❌ Assume message displays instantly (polling has delay)  
❌ Forget to unlock in cleanup/error handlers  
❌ Use complex HTML that won't render in iOS 9  

---

## **Technical Details**

### **index.html Display Logic:**

```javascript
// Overlay approach (v29-12-2025)
function showMessageHtml() {
    // Hide cover images (preserve in DOM)
    var images = coverContainer.getElementsByTagName('img');
    for (var i = 0; i < images.length; i++) {
        images[i].style.display = 'none';
    }
    
    // Create overlay on top
    var overlay = document.createElement('div');
    overlay.id = 'message-overlay';
    overlay.style.zIndex = '1000';
    // ... style as fullscreen, centered ...
    overlay.innerHTML = messageContent;
    coverContainer.appendChild(overlay);
}

function hideMessageHtml() {
    // Hide overlay
    overlay.style.display = 'none';
    
    // Show cover images again
    var images = coverContainer.getElementsByTagName('img');
    for (var i = 0; i < images.length; i++) {
        images[i].style.display = 'block';
    }
}
```

**Preserves DOM structure** - no innerHTML replacement!

---

## **File Locations**

```
~/Sites/
├── cover.jpg           # Current album cover image
├── cover.tag           # Controls display: "-1" or "/path"
├── message.html        # Message content (when cover.tag = -1)
├── scripts/
│   ├── play            # Creates messages for big folders (v29-12-2025)
│   ├── cover_slideshow # Handles message timing (v29-12-2025)
│   └── shuffle         # Not yet updated (still uses old method)
└── index.html          # Displays based on cover.tag value (v29-12-2025)
```

---

## **Quick Reference**

### **Show message:**
```bash
echo "<h2>Message</h2>" > ~/Sites/message.html
echo "-1" > ~/Sites/cover.tag
```

### **Show message (protected):**
```bash
echo "<h2>Message</h2>" > ~/Sites/message.html
echo "-1" > ~/Sites/cover.tag
lock ~/Sites/cover.tag
lock ~/Sites/message.html
```

### **Clear message:**
```bash
echo "/music/path" > ~/Sites/cover.tag
# message.html can stay - it's ignored!
```

### **Clear message (full cleanup):**
```bash
unlock ~/Sites/cover.tag
unlock ~/Sites/message.html
echo "/music/path" > ~/Sites/cover.tag
rm -f ~/Sites/message.html
```

### **Check current mode:**
```bash
cat ~/Sites/cover.tag
# Shows "-1" → message mode
# Shows "/path" → cover mode
```

---

## **Version History**

- **Pre-v27:** No messaging system
- **v27-v28:** Old method (delete cover.tag) - unreliable with races
- **v29-12-2025:** Sentinel system (-1 method) - reliable, lockable
  - index.html: Overlay-based display (preserves DOM)
  - play: Creates message + sentinel, clean handoff
  - cover_slideshow: Detects message mode, handles 3s timing

---

## **Summary**

**The sentinel messaging system provides:**
- ✅ Lockable file-based signaling (no race conditions)
- ✅ Simple flip-flop control (change one file value)
- ✅ Flexible message content (any HTML)
- ✅ Clean architecture (separation of concerns)
- ✅ Easy integration (standard pattern)
- ✅ Multiple use cases (busy, progress, errors, status)

**Key insight:** cover.tag value = display mode. "-1" = magic sentinel!

---

*Updated: December 29, 2025*  
*Sentinel system implemented and tested on iOS 9.3 and macOS Desktop*
