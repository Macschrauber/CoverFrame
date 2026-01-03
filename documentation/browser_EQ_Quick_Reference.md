# browser.html EQ - Quick Reference

**Updated:** December 24, 2025

---

## ✅ Does EQ Work on My Device?

### YES - EQ Available:
- ✅ Desktop Safari (macOS)
- ✅ iOS 14, 15, 16, 17, 18+
- ✅ iPad Air 2 or newer (can run iOS 14+)
- ✅ iPad 5th generation or newer
- ✅ iPad Mini 4 or newer
- ✅ iPad Pro (all models)

### NO - EQ Not Available:
- ❌ iOS 9, 10, 11, 12, 13
- ❌ iPad 3, 4
- ❌ iPad Air 1
- ❌ iPad Mini 1, 2, 3

---

## 🎛️ How to Use EQ:

1. Open browser.html in Safari
2. Navigate to music folder
3. Click "Show Player"
4. **If compatible:** EQ controls appear below player
5. Adjust sliders or select preset
6. Toggle "EQ: ON/OFF" to hear difference

---

## 🎚️ Controls:

### Sliders:
- **Treble:** -12 to +12 dB (high frequencies @ 5kHz)
- **Bass:** -12 to +12 dB (low frequencies @ 200Hz)

### Presets:
- **Off (0/0)** - Flat response
- **Hearing Assist (+9/-8)** - Boost treble, cut bass
- **Bass Fix (0/-10)** - Reduce bass (for corner speakers)

### Settings:
- Saved automatically in browser
- Persist across sessions
- Per-device settings

---

## ⚙️ Configuration (Optional):

Edit top of browser.html:

```javascript
var EQ_ENABLED = true;   // false to disable EQ completely
var EQ_VERBOSE = true;   // false for clean logs
```

---

## ❓ Why No EQ on My iPad?

### iOS 9-13 Web Audio Bug:
Safari on these versions has broken Web Audio API:
- Creates filters successfully
- Reports "running" state
- **BUT: Audio bypasses all processing!**
- Apple fixed this in iOS 14

### Check Your iOS Version:
Settings → General → About → Software Version

---

## 🔧 Alternative Solutions:

### If Your Device Doesn't Support EQ:

**Option 1: Mac + AirPlay**
- Apply EQ on Mac (VOX, Music app)
- Stream via AirPlay to speaker
- iPad for display/control only
- Works on ANY iOS version! ✅

**Option 2: Native Apps**
- Use apps with native EQ (Denon Audio, etc.)
- These use AudioUnits, not Web Audio
- Work on older iOS versions

**Option 3: Upgrade Device**
- iPad Air 2+ supports iOS 14+
- Used models: €150-250
- Full CoverFrame + EQ support

**Option 4: iOS Accessibility (iOS 14+)**
- Settings → Accessibility → Audio/Visual
- Headphone Accommodations
- System-wide EQ

---

## 📱 What You'll See:

### Compatible Device (iOS 14+):
```
[Show Player button]
  ↓
[Audio player appears]
[EQ controls appear below]
- Treble slider
- Bass slider
- Presets
- EQ ON/OFF toggle
```

### Incompatible Device (iOS 9-13):
```
[Show Player button]
  ↓
[Audio player appears]
[No EQ controls]
(Clean interface, everything else works)
```

---

## 🎯 Best Use Cases:

### EQ is Perfect For:
- ✅ Hearing loss compensation
- ✅ Tinnitus frequency adjustment
- ✅ Room acoustic compensation
- ✅ Speaker placement correction
- ✅ Personal preference tuning

### When to Use Each Preset:
- **Hearing Assist (+9/-8):** High frequency hearing loss
- **Bass Fix (0/-10):** Corner-placed speakers
- **Custom:** Adjust to taste

---

## 📊 Technical Specs:

**Processing:**
- Web Audio API BiquadFilter
- 32-bit float precision
- Real-time DSP
- No added latency

**Filters:**
- Treble: Highshelf @ 5000 Hz
- Bass: Lowshelf @ 200 Hz
- Q factor: 0.707 (default)

**Output:**
- Works with ALL audio outputs
- Analog, Lightning, Bluetooth, USB DAC
- Digital processing before output

---

## 🐛 Troubleshooting:

### "I don't see EQ controls"
→ Check iOS version (Settings → General → About)  
→ Must be iOS 14 or newer

### "EQ controls appear but have no effect"
→ Make sure "EQ: ON" (button should be green)  
→ Try extreme settings (+12/-12) to test  
→ Toggle on/off to hear difference

### "I want EQ but have old iPad"
→ Use Mac VOX + AirPlay solution  
→ Or upgrade to iOS 14+ device  
→ Or use native app with EQ

---

## 📚 More Information:

**Full documentation:**
- browser_html_EQ_update_24-12-2025.md
- iOS_WebAudio_EQ_Compatibility_Report.md

**Testing results:**
- Tested on iOS 12, iOS 18, Desktop
- All compatible devices confirmed working
- All incompatible devices properly hidden

---

## ✅ Quick Checklist:

Before using EQ:
- [ ] Check iOS version (need 14+)
- [ ] Open browser.html in Safari
- [ ] Navigate to music folder
- [ ] Click "Show Player"
- [ ] Look for EQ controls below player
- [ ] If no controls: iOS too old (see alternatives)

Using EQ:
- [ ] Start with preset (Hearing Assist or Bass Fix)
- [ ] Toggle "EQ: ON" (should turn green)
- [ ] Play music
- [ ] Toggle EQ on/off to hear difference
- [ ] Adjust sliders to taste
- [ ] Settings save automatically

---

**That's it! Enjoy your music with proper EQ!** 🎵

**Questions?** Check the full documentation files or test on compatible device.
