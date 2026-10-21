# PoseMate

A free photo pose coach that runs entirely on your phone. Pick Solo, Couple or Group and a place, line yourself up with a live ghost guide on the camera, and shoot. Body detection runs on the device with MediaPipe. There is no backend, account, analytics or upload.

**Stack:** React 18, Vite, TypeScript, Tailwind CSS, `@mediapipe/tasks-vision`. No router, state, UI or icon libraries.

## What it does

**Posing**
- 40 built-in poses (19 solo, 12 couple, 9 group) across 8 places: College Campus, Cafe, Street, Stairs and steps, Bedroom, Beach, Tourist Places, Formal and events. Each has 3 steps and mood tags (Confident, Playful, Cozy, Elegant, Candid).
- Make your own pose from a photo (`#/create`): joints, body outline and part lines are found on-device. Optional "photo ghost" keeps a faded cut-out of the person.
- Favorites, and "Pick up where you left off" (your last 6 poses) on Home.

**Camera**
- Live skeleton for up to 5 people, a ghost figure and outline guide, and a match score with one short tip ("Lift your left arm") for solo poses.
- **Fit to me:** the guide scales and moves onto your body. **Drag and pinch** to place it yourself; Reset guide puts it back.
- Auto-snap (hold the pose and it shoots), self-timer, front/rear switch, level line, lighting tips (too dark, too bright, light behind you).
- **Candid burst:** 6 quick frames, then pick the best (the sharpest is marked).
- Highest practical resolution, original photo never modified, overlay kept separate. Save, download, share, before/after Compare slider.

**Make it yours (`#/customize`)**
- Colors (5 presets and all 11 theme colors), fonts (7 built in, or your own font file), text size, corner style.
- Replace any icon, including the camera buttons and the shutter, with a picture or an emoji.
- **Tap anything to change it:** rename text, recolor, add a picture to, hide, or swap the icon of any button or heading. Undo each change from "Your changes". Save your look to a file and load it back.

**Built for older phones**
- Performance setting (Auto / Light / Full). Light mode limits tracking to 15 per second, reads a smaller image, drops blur and animations, and takes smaller burst frames. Auto turns it on for phones with 4 or fewer cores or 2 GB of memory or less.
- Adaptive frame rate (20, 15, 10, 6 per second) that slows itself down if the phone cannot keep up, and a Battery saver (Off / Auto / On).
- Accuracy: Lite (default, ~6 MB), Full (~9 MB), Heavy (~30 MB). Larger models are saved on the phone the first time and freed when you leave the camera.

**Works offline and installs like an app** (PWA): after one online visit the app, the Lite model and the WASM runtime are cached.

## Run it

Requires Node 18 or newer.

```bash
npm install     # also copies the MediaPipe WASM and downloads the pose models into public/
npm run dev
```

If the model download fails (offline install), save the model yourself as `public/models/pose_landmarker_lite.task` (and `_full.task` / `_heavy.task` for the other accuracy levels), then run `npm run setup:assets`. A failed Full or Heavy download only prints a warning; the app falls back to Lite. To build a slimmer site: `POSEMATE_MODELS=lite npm run build`.

The URL pattern is `https://storage.googleapis.com/mediapipe-models/pose_landmarker/pose_landmarker_lite/float16/latest/pose_landmarker_lite.task`.

### Trying it on a phone

Browsers allow the camera only on **HTTPS or localhost**, so a plain `http://192.168.x.x:5173` will not work. Either:

1. **USB debugging:** enable Developer options and USB debugging on the phone, open `chrome://inspect/#devices` on your computer, add port forwarding `5173 → localhost:5173`, then open `http://localhost:5173` on the phone.
2. **Deploy** to an HTTPS host (below) and open that link.

## Build and publish

```bash
npm run build     # downloads assets if missing, type-checks, builds dist/, writes dist/sw.js
npm run preview   # serve dist/ locally
```

Use `npm run build`, not `vite build` alone: the last step writes the service worker that makes offline work.

`dist/` is plain static files with relative paths and hash routing, so it runs at a domain root or in a sub-folder with no rewrite rules.

- **Netlify:** drag the `dist` folder onto app.netlify.com/drop, or connect the repo (build command `npm run build`, output `dist`).
- **Cloudflare Pages / Vercel:** same build command and output directory.
- **GitHub Pages:** publish `dist/` to a `gh-pages` branch, or use a workflow that runs `npm ci && npm run build`.
- Make sure `.wasm` is served as `application/wasm` (all major hosts do this).

If you build in CI, the build needs internet access once to download the models. Alternatively, commit the model file(s) under `public/models/`.

### Installing it as an app

Open the hosted link in Chrome on your phone, let it load once while online, then use the menu's **Install app** (Home also shows an Install button when the browser allows it). It then opens full screen from your app drawer and works offline.

For a Play Store listing or a standalone `.apk`, wrap the hosted site with PWABuilder or Bubblewrap (a Trusted Web Activity). New personal Play developer accounts currently need a closed test with at least 12 testers for 14 days before production access, so check Google's current rules first. For personal use, the Chrome install is enough.

After you deploy an update, an installed app picks it up the next time it is opened with a connection.

## Settings

| Setting | Default | What it does |
| --- | --- | --- |
| Pose guide, Ghost strength, Ghost style, Body-part lines | On, Medium | What is drawn on the camera. Ghost strength, body-part lines and skeleton are remembered |
| Live skeleton | On | Your own body as the camera sees it |
| Fit guide to me | Off | Guide eases onto your size and position |
| Move guide with fingers | On | One finger drags, two fingers pinch |
| Level line | On | Horizon line that stays level as the phone tilts |
| Auto-snap | Off | Solo poses: match reaches 80% and stays above 65% for 1.5 s, then it shoots |
| Candid burst | Off | Each shutter press takes 6 frames (4 in light mode) to pick from |
| Sounds and vibration | On | Timer ticks, shutter click, small buzz (Android) |
| Accuracy | Lite | Lite / Full / Heavy pose model |
| Performance | Auto | Auto / Light / Full |
| Battery saver | Auto | Off / Auto (on at 20% battery or less, unplugged) / On |
| Pose tips, Lighting tips | On | Short advice on the camera |
| Keep screen on | On | Screen wake lock while the camera is open |
| Lighting readout | Off | Shows measured light numbers, with Copy log, for tuning |

## Privacy

- Camera frames and photos are never uploaded. Detection runs in your browser, using a WASM runtime and model served from your own site.
- The camera is requested only when the camera screen opens.
- Saved photos and your own poses live in the browser's IndexedDB on that device. Settings, favorites and your custom look live in localStorage. Clearing site data deletes them. `#/privacy` shows what is stored and can delete each kind.
- **Photo ghost** is the one exception to "only outlines are stored". It is off by default; if you turn it on for a pose, a small cut-out of each person in that photo is saved on the device next to the pose, and deleted with it.
- No analytics, cookies, accounts or third-party requests at runtime.
- Optional hardening: a Content-Security-Policy limiting `connect-src` to `'self'`. MediaPipe needs `'wasm-unsafe-eval'` in `script-src`, so test it on a device first.

## Project layout

```
src/
  App.tsx, main.tsx       routes; theme and customization applied at start
  data/poses.ts           pose library (joints, steps, scenes, moods)
  data/custom.ts          poses you made from photos
  screens/                Home, Scenes, Poses, Camera, Gallery, Create, Favorites, Privacy, Customize
  camera/                 camera screen parts
    useCameraStream       camera and permission
    useDetector           model loading, accuracy fallback
    useDetectionLoop      detect, draw, light, match, tips, auto-snap, guide fit
    useGuideOptions       guide, skeleton, ghost options
    useGuideGestures      drag and pinch
    GuideSettings, PoseSheet, LightDebugPanel, constants
  components/             PoseArt, PoseCard, HeartButton, Preview, BurstPicker, CompareSlider,
                          PhotoWithOverlay, LevelLine, Header, BottomNav, Controls, Icon
  ui/                     theme (CSS variables, fonts, images), engine (applies renames and
                          element tweaks), EditMode (tap to change), editing state
  lib/
    camera, pose, draw    getUserMedia and capture, MediaPipe, overlay drawing
    match, fit, guideAdjust  match score and tips, guide fitting, drag/pinch maths
    smooth, perf, accuracy   One Euro smoothing, frame governor and light mode, model levels
    light, burst          lighting check and log, burst capture and scoring
    outline, parts, vision, cutout, silhouette   pose-from-photo and guide art
    prefs, ui             settings (localStorage), customization store
    db, files, route, device, install, tilt   storage, share, routing, sounds, PWA, level
scripts/
  setup-assets.mjs        copies WASM, downloads models to public/
  make-sw.mjs             writes dist/sw.js after the build
```

## Adding poses

Add an entry to `POSES` in `src/data/poses.ts`. Start from `STAND` or `SIT`, override joints with `mk(...)`, and place figures with `place(...)`. Coordinates are in a 0..1 square as seen by the camera, so the person's left side is on the image right. Give it `scenes`, `moods` and 3 `steps`.

## Tuning

First-guess numbers you may want to adjust after real use:

| Where | What |
| --- | --- |
| `src/lib/light.ts` | Dark, bright and backlit thresholds (use the Lighting readout and Copy log) |
| `src/camera/constants.ts` | Auto-snap enter, release and hold time |
| `src/lib/perf.ts` | Frame rate steps, light mode cap, what counts as a low-end phone, burst size in light mode |
| `src/lib/burst.ts` | Burst frame count, gap between frames, scoring weights |
| `src/lib/match.ts` | Match score and the angle at which a tip appears |

## Known limits

- Pose advice is 2D: it cannot tell whether a limb points toward the camera.
- Match score, tips and auto-snap work for solo poses only (couple and group need person-to-figure assignment).
- Burst frames are the live-view size, not full sensor, and there is no eyes-open or smile check (the pose model has no face data).
- The built-in guide body is a generic mannequin shape; poses made from a photo have real outlines.
- No true dark theme: many places use fixed white and black. Colors drawn on the camera (skeleton, ghost) are not themeable.
- Rename in Customize matches whole pieces of text; labels that change with state ("Burst on" / "Burst off") are separate items.
- Front-camera photos are saved un-mirrored (as others would see you); the live view is mirrored.
- Wake lock, battery status, vibration and motion sensors depend on the browser.
- There is no automated test suite in the repo; logic was checked with ad hoc scripts, and the screens by hand on a phone.

## Ideas for next

- Couple and group match score and tips.
- More poses, in packs by place or occasion, and sharing poses between phones.
- Saving photos straight to the gallery (would need a native wrapper such as Capacitor).
- Splitting the remaining capture and burst code out of `Camera.tsx`, and adding automated tests.

## Design notes

- **One idea:** *frame yourself.* Viewfinder corner marks frame every pose illustration, the way the camera frames you. Flat colour, plain type, no shadows.
- **Colour means something:** Solo is yellow-green, Couple is pink, Group is aqua, and each place has its own colour. Cobalt marks the main action on a screen. All of these can be changed in Customize.
- **Navigation:** a thumb-reach bottom bar (Home, Camera, Make a pose, My photos) on every screen except the camera. The camera has four top controls; everything about the guide lives in one Guide settings panel.
