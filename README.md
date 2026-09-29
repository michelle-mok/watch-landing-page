# Scroll-Driven 3D Product Page

A landing page for a fictional limited-edition collector's watch.

**Live Demo:** https://3d-watch-landing-page.vercel.app/

![watch mid-rotation beside "Made, not Moulded" section](docs/screenshot.png)

## What it does

- **Scroll-driven product rotation.** The watch rotates through 1 turn full continuously as the user scrolls down the page, allowing them to see the watch from every side. The motion is eased so that it glides instead of jumping.

- **Three flowing sections.** _Presence_, _Made, not Moulded_ and _Three Hundred_ fade in and out beside the watch, ending with a **Register** call to action.

- **Built for both phones and desktops.** On mobile, the copy moves above the watch rather than beside it, with its own fade timing.

- **Clean first impression.** A loading overlay covers the page until the watch is rendered, so users have a premium experience (no half-built pages presented to them).

## Performance

The source model is 7.4MB. After optimisation, the whole deployed site (code and model) is 4.1MB.

|                           | before | after             |
| ------------------------- | ------ | ----------------- |
| 3D model                  | 7.4MB  | **2.8MB** (-63%)  |
| Texture memory on the GPU | 71.7MB | **21.3MB** (-70%) |

- **First load:** under 2 seconds on 5G, about 1 second on home wifi.
- **Smoothness:** holds 60fps at rest and while scrolling on a Xiaomi 14 (Snapdragon 8 Gen 3, Chrome).

## Technical decisions

- **Meshopt over Draco for geometry.** Both compress well but drei's useGLTF fetches the Draco decoder from Google's CDN at runtime, while Meshopt's decoder is already bundled with the app, resulting in one less third party request.

- **Lossless WebP.** Lossy WebP was measured at a saving of a further 1.2MB but was rejected because it visibly flattened the normal and roughness maps, resulting in a less realistic render of the watch model.

- **Textures capped at 1024px.** Cuts GPU texture memory by 70%. Reviewed the model rendered on screen and kept.

- **The glass costs a second render.** The watch crystal uses real transmission, which makes Three.js redraw the whole scene into an offscreen buffer every frame. That buffer renders at 75% resolution; at 50% the dial behind the glass appears blurry.

- **The loading screen waits for the first frame to be drawn, not the model to be downloaded.** After the files are downloaded, the model is not immediately visible as the browser still has to decode textures and compile shaders. A flag set after the watch's first rendered frame (shared via a Zustand store) hides the overlay only when the model has been rendered.

- **Reproducible pipeline.** `npm run model:optimise` rebuilds the optimised model from the original source.

## Architecture

```
src/
├── components/
│   ├── canvas/     everything mounted inside <Canvas>
│   └── dom/        everything in the browser DOM
├── stores/         Zustand - state shared between <Canvas> and DOM
└── config/         shared constants
assets-src/         original model - not shipped
public/models/      optimised model
```

- **Folders follow the renderer.** A component lives in `canvas/` or `dom/` depending on which React renderer mounts it.

- **Per-frame work stays out of React.** Values that change every frame, such as the backdrop's animated glow, are written onto three.js objects inside `useFrame`, and the scroll rotation is tweened by GSAP, so they do not cause React re-renders.

- **Only one store.** The only shared state is the flag that the canvas writes after the watch is first drawn, and the loading overlay subscribes to it. Nothing else requires a store.

# Running locally

Developed on Node.js 24.

```bash
npm install
npm run dev     # development server
npm run build   # type-check, then production build to dist/
npm run preview # serve the production build locally
npm run lint    # ESLint
```

Add `?stats` to the URL to show a live FPS panel.

```bash
npm run model:optimise
```

This writes intermediate files to `/tmp`, so it runs on macOS, Linux and WSL, not native Windows.

## Credits

**3D model** - _Chronograph Watch_, from the [Khronos glTF Sample Assets](https://github.com/KhronosGroup/glTF-Sample-Assets/tree/main/Models/ChronographWatch). Attribution, verbatim from the model file:

> Asset © 2025 Darmstadt Graphics Group GmbH, CC BY 4.0 International, adapted by Eric Chadwick. Original asset Chronograph Watch Mudmaster (https://skfb.ly/oAsPA) by graphiccompressor is licensed under Creative Commons Attribution (http://creativecommons.org/licenses/by/4.0/). Khronos logo © 2015, Khronos Group. DGG logo © 2020, Darmstadt Graphics Group GmbH.

**Changes made for this project:** textures resized to a maximum of 1024 px and re-encoded as lossless WebP; unused data removed; geometry quantised and compressed with Meshopt, using [glTF Transform](https://gltf-transform.dev/). Used under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

**Fonts** - Gilda Display and Inter, served from Google Fonts under the SIL Open Font License.
