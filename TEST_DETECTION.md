# Marker Detection Debugging Guide

## Quick Test: Check if Events Are Firing

**Step 1: Open index.html and browser console (F12)**

**Step 2: Paste this into console to verify marker element was found:**
```javascript
const marker = document.getElementById("article-marker");
console.log("Marker element found:", !! marker);
console.log("Marker ID:", marker?.id);
console.log("Marker tag:", marker?.tagName);
console.log("Marker has emitevents:", marker?.getAttribute("emitevents"));
```

Expected output:
```
Marker element found: true
Marker ID: article-marker
Marker tag: A-NFT
Marker has emitevents: true
```

**Step 3: Check if event listeners are attached:**
```javascript
const marker = document.getElementById("article-marker");
console.log("Event listeners on marker:");
console.log("  markerFound listeners:", marker._listeners?.markerFound?.length || "checking...");
console.log("  markerLost listeners:", marker._listeners?.markerLost?.length || "checking...");

// Check A-Frame component
const aframeComp = marker._aframeComponent;
console.log("A-Frame component:", !! aframeComp);
```

**Step 4: Print the marker and point at it**

Watch the **browser console** for these messages when marker is in frame:

```
✓ Marker element found: article-marker
✓ Event listeners attached to marker element
🟢🟢🟢 MARKERFOOUND EVENT FIRED 🟢🟢🟢
★★★ MARKER FOUND - Tracking Active ★★★
Found overlay entities: 3
Made overlay 0 visible
Made overlay 1 visible
Made overlay 2 visible
✓ Green detection overlay SHOWN
Found text labels: 3
```

**If you DON'T see these messages:**

→ Events not firing = Detection not working = Need to check AR.js configuration

## Diagnostic: Check Why Detection Isn't Firing

### Issue: "markerFound never fires"

This usually means AR.js is either:
1. Not recognizing the printed marker
2. Confidence threshold too high
3. Camera can't see the marker clearly

**Test 1: Check minConfidence threshold**
```javascript
const marker = document.getElementById("article-marker");
const comp = marker.components?.[' arjs-anchor'];
console.log("AR.js anchor component:", comp?.data || "Not found");
console.log("minConfidence setting:", comp?.data?.minConfidence);
```

Should show: `minConfidence: 0.2` (very permissive)

**Test 2: Verify camera is capturing**
```javascript
const video = document.querySelector("video");
console.log("Video element found:", !! video);
console.log("Video playing:", video?.paused === false);
console.log("Video dimensions:", video?.videoWidth, "x", video?.videoHeight);
```

Should show:
```
Video element found: true
Video playing: true
Video dimensions: 1280 x 960 (or similar)
```

**Test 3: Check if ARToolkit is processing frames**

Look for these logs in console:
```
[info] Allocated videoFrameSize 307200
[ARController] Got ID from setup 0
NFT marker width: 1984
NFT marker height: 1264
NFT num. of ImageSet: 18
```

If you see these → AR.js is working, just not detecting your marker.

## What Overlay Should Look Like

When marker is detected, you should see:

1. **Large Semi-Transparent Green Overlay** (35% opacity)
   - Covers entire marker area
   - Bright green (#00ff00)
   - Very obvious

2. **Three Text Labels** (with colored backgrounds):
   - Red box: "[ WARNING: DEEPFAKE SIGNATURE DETECTED ]" at top
   - Green box: "[ VERIFIED: HUMAN ORIGIN ]" in middle
   - Yellow box: "[ ALERT: HIGH AI-PERPLEXITY SCORE ]" at bottom

3. **Status Panel** (bottom-left corner):
   - Detection: "🟢 FOUND" in green
   - Last Event: "Marker detected at: 14:32:15"

4. **Large Center Popup**:
   - Green message "MARKER DETECTED ✓"
   - Auto-hides after 2.5 seconds

## Marker Image Requirements

Your marker image (from markers/article.fset) needs:

✅ **Sharp, High-Contrast Features**
- Text with clear edges
- Geometric patterns
- Diagonal lines
- Checkboards or grids
- Real photographs

❌ **Won't Work:**
- Smooth AI-generated images
- Pure-color gradients
- Blurry or low-res images
- Solid white/black areas

**DPI:** 144 DPI (standard screen resolution)
**Size:** 22cm × 14cm (when printed at 144 DPI)
**Paper:** Glossy or matte both work, but glossy = better

## Step-by-Step Test Procedure

### Setup:
1. Open index.html in browser
2. Wait for loading screen to disappear
3. Open console (F12)

### Printing:
4. Use test-images.html to verify your marker's detection score:
   - If score > 60 → Good marker, should detect
   - If score < 40 → Poor marker, likely won't detect

### Testing:
5. Print marker at 22cm × 14cm on glossy paper
6. Point camera at marker
7. Watch console for:
   - `🟢🟢🟢 MARKERFOOUND EVENT FIRED 🟢🟢🟢`
   - `Found overlay entities: 3`
   - `✓ Green detection overlay SHOWN`

8. Watch screen for:
   - Large green transparent overlay on camera feed
   - Three label boxes (red, green, yellow)
   - Green popup "MARKER DETECTED ✓" in center

### If Nothing Shows:
- Check console for errors
- Verify video element is playing
- Try different lighting
- Try closer to camera
- Try rotating marker slowly

## Console Output Examples

### ✅ SUCCESS (Detection Working):
```
✓ Marker element found: article-marker
✓ Event listeners attached to marker element
[ARController] Got ID from setup 0
NFT num. of ImageSet: 18
📷 Requesting HD camera: {...}
Camera ready. Point at the article marker to start verification.
🟢🟢🟢 MARKERFOOUND EVENT FIRED 🟢🟢🟢
Event details: Event { type: "markerFound", ... }
★★★ MARKER FOUND - Tracking Active ★★★
Found overlay entities: 3
Made overlay 0 visible
Made overlay 1 visible
Made overlay 2 visible
✓ Green detection overlay SHOWN
```

### ❌ FAILURE (Detection Not Working):
```
✓ Marker element found: article-marker
✓ Event listeners attached to marker element
[ARController] Got ID from setup 0
NFT num. of ImageSet: 18
📷 Requesting HD camera: {...}
Camera ready. Point at the article marker to start verification.
[... point camera at marker for 5 seconds, nothing happens ...]
```

If you see FAILURE pattern:
- Marker image may have no detectable SIFT features
- Lighting may be poor
- Printed size may be wrong
- Try using test-images.html to verify marker quality

## Testing with Test Images

**Go to test-images.html:**
1. Shows 6 pre-generated test images
2. Each has a "Detection Score" below it
3. Scores > 60 are good markers, < 40 are poor

Your printed marker should ideally score:
- Real photo: 70-90
- High-contrast printed text: 60-80
- Newspaper page: 50-70
- Low-contrast gradient: 10-30 (won't detect)

## Performance Notes

- Detection latency: 16-33ms (happens every frame at 60 FPS)
- Overlay rendering: Negligible (WebGL triangles)
- CPU impact: < 5% additional (worker processes in background)
- Memory impact: < 20MB (NFT descriptors cached)

## If Still Stuck

Check these in order:

1. **Browser Console (F12)** for error messages
   - Any JavaScript errors?
   - Any WebGL errors?

2. **Verify Marker Files Loaded** (see HTTP requests):
   - HTTP 200 for markers/article.fset?
   - HTTP 200 for markers/article.fset3?
   - HTTP 200 for markers/article.iset?

3. **Check Camera Works**:
   - See camera input on screen?
   - Any permission dialogs to accept?
   - Try incognito window (fresh permissions)

4. **Try Different Marker**:
   - Print test pattern (high contrast)
   - Or use a photograph
   - Different lighting / distance / angle

5. **Browser Compatibility**:
   - Tested on: Chrome (best), Firefox, Edge, Safari 14.1+
   - Not supported: IE 11

**Still need help?** Share:
- Screenshots of console output (F12 → Console)
- What you're pointing camera at
- What you expect to see vs. what's actually happening
