# PoseMate

A free, mobile-first photo pose coach. Pick Solo, Couple or Group, choose a place, follow a reference pose with a live camera guide, and capture. Body detection runs on the phone with MediaPipe. There is no backend, login, database, analytics or upload.

**React + Vite + TypeScript, Tailwind CSS, `@mediapipe/tasks-vision`.** No router, state, UI or icon libraries.

## Status

> This code was written without being able to install packages or run a browser, so it has **not been compiled or tested yet**. Run the steps below and test on a real Android phone before publishing. Treat anything under "Needs device testing" as unverified.

### Completed (implemented in code)

| Feature | Notes |
| --- | --- |
| Home with Solo / Couple / Group | `src/screens/Home.tsx` |
| Scenes: College Campus, Cafe, Bedroom, Beach, Tourist Places | 5 scenes per mode |
| 26 pose cards with illustrations and 3-step instructions | 13 solo, 7 couple, 5 group. Illustrations are filled body silhouettes generated from the same joint data as the live guide |
| Hash routing (Android back button works, no server rewrites) | `src/lib/route.ts` |
| Camera opens and asks permission only on the camera screen | `src/screens/Camera.tsx` |
| Front / rear switching, self-timer (3s, 10s), flash feedback | |
| Capture, preview, retake, save locally (IndexedDB), download, share, delete | Gallery at `#/gallery` |
| Highest practical resolution | Requests 4:3 at up to 4032×3024; uses `ImageCapture.takePhoto` for full-sensor stills where supported, otherwise the video frame. Real size is shown in the preview |
| Original photo kept separate from the overlay | Photo blob is never modified. Landmarks and pose id are stored as metadata; the overlay is a separate canvas you can toggle |
| Real MediaPipe Pose Landmarker | `src/lib/pose.ts`, GPU with CPU fallback, loaded on demand |
| Live skeleton from detected landmarks | Drawn from actual `PoseLandmarker` output, up to 5 people |
| Reference pose guide on the camera | Body-shaped outline with a soft tint (`src/lib/silhouette.ts`), toggleable. The live skeleton stays as lines so you can tell detected landmarks from the guide |
| Match score (solo poses) | Real calculation: limb directions compared with the reference |
| Error messages | Camera permission, no camera, camera busy, insecure context, missing model, unsupported browser, storage blocked |

### Needs device testing

- Full-resolution capture on your phone (some browsers reject `takePhoto` at max size; the code then falls back to the video frame).
- Overlay alignment in the preview. It is only shown when the photo has the same aspect ratio as the live view.
- Detection speed on low-end Android phones (the lite model is used and frames are downscaled to 480px).
- Front-camera photos are saved **un-mirrored** (as other people would see you). The live view is mirrored.

### Placeholders / not built

- The body outline is a generic mannequin shape built from the joints, not a real person's contour. Real traced outlines would need the pose-from-photo creator (not built).
- Match score is skipped for couple and group poses (it would need person-to-figure assignment).
- No installable PWA or offline mode (no service worker or icons yet).
- No "export with overlay baked in" (by design the original is kept clean).
- No pose editing, no user-added poses, no landscape-specific layout.

## Run it

Requires Node 18+.

```bash
npm install        # also copies the MediaPipe WASM and downloads the ~6 MB pose model into public/
npm run dev
```

If the download fails (offline install), save the model yourself as `public/models/pose_landmarker_lite.task`:
https://storage.googleapis.com/mediapipe-models/pose_landmarker/pose_landmarker_lite/float16/latest/pose_landmarker_lite.task
then run `npm run setup:assets`.

### Testing on a phone

Browsers only allow the camera on **HTTPS or localhost**, so `http://192.168.x.x:5173` will not work. Options:

1. USB debugging: on the phone enable Developer options → USB debugging, open `chrome://inspect/#devices` on your computer, add port forwarding `5173 → localhost:5173`, then open `http://localhost:5173` on the phone.
2. Deploy to any HTTPS host (below) and open that URL.

## Build and publish

```bash
npm run build      # type-checks, then writes the static site to dist/
npm run preview    # serve dist/ locally
```

`dist/` is plain static files with relative paths and hash routing, so it works at a domain root or in a sub-folder with no rewrite rules.

- **Netlify / Cloudflare Pages / Vercel:** connect the repo; build command `npm run build`, output directory `dist`. Or drag `dist/` onto Netlify Drop.
- **GitHub Pages:** push `dist/` to a `gh-pages` branch (or use a Pages workflow that runs `npm ci && npm run build`).
- **Any static host:** upload the contents of `dist/`. Make sure `.wasm` is served as `application/wasm` (all major hosts do this).

If you build in CI, the build needs internet access once to download the model. Alternatively commit `public/models/pose_landmarker_lite.task` and remove it from `.gitignore`.

## Privacy

- Camera frames and photos are never uploaded. Detection runs in your browser using a WASM runtime and model served from your own site.
- The camera is requested only when the camera screen opens.
- Saved photos live in the browser's IndexedDB on that device. Clearing site data deletes them.
- No analytics, cookies, accounts or third-party requests at runtime.
- Optional hardening: add a Content-Security-Policy that restricts `connect-src` to `'self'`. Test it on a device first, because MediaPipe needs `'wasm-unsafe-eval'` in `script-src`.

## Project layout

```
src/
  data/poses.ts          pose library (joints, steps, scenes)
  lib/camera.ts          getUserMedia, errors, best-resolution capture
  lib/pose.ts            MediaPipe loader and detector
  lib/draw.ts            overlay drawing (guide + skeleton)
  lib/match.ts           pose match score
  lib/db.ts, files.ts    local storage, download, share
  screens/               Home, Scenes, Poses, Camera, Gallery
  components/            PoseArt, Preview, PhotoWithOverlay, Header, Icon
scripts/setup-assets.mjs copies WASM + downloads model to public/
```

## Adding poses

Add an entry to `POSES` in `src/data/poses.ts`. Start from `STAND` or `SIT`, override joints with `mk(...)`, and place figures with `place(...)`. Coordinates are in a 0..1 square as seen by the camera, so the person's left side is on the image right.
