# Visible Feedback System: Quick Debug Checklist

## ✅ What Was Added

1. **Status Panel (bottom-left corner)**
   - Shows: Camera, Marker, Detection, Confidence, Last Event
   - Appears when system ready (after loading screen)
   - Updates in real-time during marker detection

2. **Marker Detection Popup (center screen)**
   - Green popup: "MARKER DETECTED ✓"
   - Red popup: "MARKER LOST ✗"
   - Auto-displays for 2.5 seconds

3. **Event Handlers**
   - markerFound event: Updates status + shows green popup
   - markerLost event: Updates status + shows red popup

## 🔍 If Status Panel Doesn't Appear

**Checklist:**

- [ ] Open browser console (F12)
- [ ] Look for: "Camera ready. Point at the article marker to start verification."
- [ ] If NOT there → Camera not ready → Jump to "Camera Issues" below
- [ ] If YES there → Check that status panel div exists:
  ```javascript
  // Paste in console:
  document.getElementById("status-panel").style.display
  // Should return: "block"
  // If returns "none" → Status panel hidden, test markerFound event
  ```

- [ ] If still not visible → Check CSS is loaded:
  ```javascript
  // Paste in console:
  window.getComputedStyle(document.getElementById("status-panel")).background
  // Should return: "rgba(6, 7, 10, 0.9)" or similar
  ```

## 🔍 If Green/Red Popup Doesn't Show

**Checklist:**

- [ ] Point camera at printed marker
- [ ] Wait 2 seconds for detection
- [ ] Check console for: "★★★ MARKER FOUND - Tracking Active ★★★"
- [ ] If NOT there → Marker not detecting → Jump to "Marker Issues" below
- [ ] If YES there → Popup should have shown. Check:
  ```javascript
  // Paste in console:
  document.getElementById("marker-detection-event").style.display
  // Should return: "block" while popup is showing
  // Should return: "none" after 2.5 seconds (auto-hides)
  ```

- [ ] Test popup manually:
  ```javascript
  // Paste in console to trigger green popup:
  (window.showMarkerDetectionEvent || (() => {})("MARKER DETECTED ✓", false))
  
  // Or if function exists:
  showMarkerDetectionEvent("MARKER DETECTED ✓", false)
  ```

## 🔍 Camera Issues

If status panel shows "❌ Camera Error" or camera won't start:

- [ ] Check browser console for: "📷 Requesting HD camera"
- [ ] Check permissions: Does browser ask for camera access?
- [ ] If YES → Allow it
- [ ] If NO → Check index.html has getUserMedia interceptor (lines 800-815)
- [ ] Try incognito window (clears cache + permissions)
- [ ] Check browser supports WebRTC (Chrome, Firefox, Edge, Safari 14.1+)

**Console Test:**
```javascript
// Paste in console:
navigator.mediaDevices.getUserMedia({video: {width: {ideal: 1280}}})
  .then(stream => {
    console.log("✅ Camera works! Resolution:", 
                stream.getVideoTracks()[0].getSettings().width,
                "x", 
                stream.getVideoTracks()[0].getSettings().height);
    stream.stop();
  })
  .catch(err => console.error("❌ Camera error:", err.message));
```

## 🔍 Marker Issues

If marker loads but won't detect (status shows "⏳ Waiting" always):

- [ ] Check console for: "✓✓✓ arjs-nft-loaded global event fired"
- [ ] If NOT there → NFT files not loading → Check markers/ folder has:
  - `article.fset`
  - `article.fset3`
  - `article.iset`
  - `README.md` (optional)

- [ ] If YES → Marker loaded but not detecting. Check:
  - Is printed marker sharp / high contrast?
  - Is lighting good?
  - Does marker have distinctive features (text, patterns)?
  - Try rotating: does detection change?

**Feature Analysis Test:**
Use test-images.html to check marker image quality:
1. Open `test-images.html` in browser
2. See "Detection Score" for each test image
3. If your marker image is AI-generated → Score will be <40 (expected)
4. If marker is real photo → Score should be >60

## 🔍 Status Panel Sections

### Camera Status
```
⏳ Initializing → ✅ Ready → ❌ Error
```
- Initializing = camera permissions pending
- Ready = camera active, video streaming
- Error = camera denied or not available

### Marker Status  
```
⏳ Loading → ✅ Loaded → ⚠️ Missing
```
- Loading = NFT descriptor files downloading
- Loaded = NFT ready, waiting for marker in frame
- Missing = .fset/.fset3 files not found

### Detection Status
```
⏳ Waiting → 🟢 FOUND → ⏸ Tracking
```
- Waiting = camera + marker ready, no marker in frame
- Found = marker detected in current frame
- Tracking = marker was detected, currently tracking

### Confidence
```
0% → 35% → 75% → 100%
```
- Real photo with good lighting: 70-90%
- AI-generated image: 5-25%
- Printed paper: 40-60%

## 📋 Event Timing

**Normal Sequence:**
```
T=0s    "Initializing camera and tracker"
T=1s    "📷 Requesting HD camera"
T=2s    "Camera ready. Verifying marker assets..."
T=3s    "NFT num. of ImageSet: 18"
T=4s    "Camera and marker assets ready. Point at..."
        [Status panel appears]
        
T=5s    [Point camera at marker]
        "Worker process: Processing frame"
        [< 100ms later]
        "★★★ MARKER FOUND"
        [Green popup shows]
        [Status panel: Detection = "🟢 FOUND"]
        
T=10s   [Move marker away]
        "★★★ MARKER LOST"
        [Red popup shows]
        [Status panel: Detection = "⏸ Tracking"]
```

## 🐛 Advanced Debugging

**Enable Verbose Logging:**
```javascript
// Paste in console BEFORE refreshing page:
window.DEBUG_VERBOSE = true;
// Then refresh - more detailed logs will appear
```

**Check All Events Firing:**
```javascript
// Paste in console:
const markerEl = document.querySelector('[arjs-nft-markerhandler]');
console.log("Marker element found:", !!markerEl);
console.log("markerFound listeners:", markerEl?.listeners?.markerFound?.length || 0);
console.log("markerLost listeners:", markerEl?.listeners?.markerLost?.length || 0);
```

**Inspect Status Panel Values:**
```javascript
// Paste in console:
const row = document.getElementById("detection-status");
console.log({
  text: row.textContent,
  classes: row.className,
  visibility: row.offsetParent !== null ? "visible" : "hidden"
});
```

**Test Animation:**
```javascript
// Paste in console:
const popup = document.getElementById("marker-detection-event");
popup.textContent = "TEST POPUP";
popup.style.display = "block";
popup.classList.add("found");
// Popup should appear in center, green
// After 2.5s should fade out
```

## 📊 Expected Console Output

**During Normal Operation (with green popup):**
```
📊 Marker locked. Spatial threat overlays are active.
★★★ MARKER FOUND - Tracking Active ★★★
```

**During Normal Operation (with red popup):**
```
📊 Marker lost. Re-frame the printed article.
★★★ MARKER LOST - Tracking Stopped ★★★
```

## ✨ Verification Checklist

- [ ] Open index.html
- [ ] See loading screen with spinner
- [ ] Wait for spinner to disappear
- [ ] See status panel appear (bottom-left)
- [ ] Panel shows: Camera ✅, Marker ✅, Detection ⏳
- [ ] Point camera at printed article
- [ ] See large green popup: "MARKER DETECTED ✓"
- [ ] Panel updates: Detection = "🟢 FOUND"
- [ ] See overlay entities on marker (DEEPFAKE/VERIFIED labels)
- [ ] Move marker away
- [ ] See large red popup: "MARKER LOST ✗"
- [ ] Panel updates: Detection = "⏸ Tracking"

If all ✅ → System working as designed!

## 🆘 Still Having Issues?

1. **Check browser console:** Is there an error message?
2. **Try different browser:** Chrome works best for AR.js
3. **Check file permissions:** Can you access index.html?
4. **Verify marker files:** Do markers/ folder have .fset files?
5. **Test camera directly:** Can you use camera in other web app?
6. **Clear cache:** Ctrl+Shift+Del → Clear all → Reload

If problem persists, capture:
- [ ] Browser console output (F12 → Console tab → Ctrl+A → Ctrl+C)
- [ ] Screenshot of status panel (if visible)
- [ ] Screenshot of what you're pointing at with camera
- [ ] Browser + OS info (Help → About in Chrome)
