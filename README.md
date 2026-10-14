# FrameTube

youtube's tv app as a flat linux app for the steam frame. launch it from your steam library, it floats over passthrough and you just watch stuff with your controllers, no laser pointing at tiny buttons.

it's basically youtube.com/tv in a fullscreen electron window, plus a couple fixes so it actually works on the frame.

## install

[![Install with FramePort](https://cdn.jsdelivr.net/gh/spoopyghosty0/frameport@main/docs/badges/install-with-frameport-animated.svg)](https://frameport.app/install?manifest=https://github.com/coah80/frametube/releases/latest/download/manifest.json)

1. grab `FrameTube-linux-arm64.tar.gz` from the [releases](../../releases) and unpack it
2. in [FramePort](https://github.com/spoopyghosty0/frameport), go to add games, then "Add a Linux app… folder", and pick the unpacked folder
3. install it to the frame, it shows up in your steam library as FrameTube

no frameport? copy the folder to the frame, add `frametube` (the small launcher, NOT `frametube-electron`) as a non-steam game, done.

on a steam deck or a pc grab `FrameTube-linux-x64.tar.gz` instead, same deal. distros that block user namespaces (ubuntu 23.10+ does) need `--no-sandbox` in the launch options, steamos doesnt.

every release is built by github actions from the tagged commit (`.github/workflows/release.yml`), each download has a `.sha256` next to it.

signing in works like any tv, pick sign in and enter the code at youtube.com/activate on your phone. it stays signed in.

## controls

| controller | does |
|---|---|
| d-pad or left stick | move around |
| A | select |
| B or the view button | back |
| X | play / pause |

## how it works

- youtube only serves the tv app to tvs, so the window says its a tv (cobalt, youtube's own tv runtime). the tv app wants remote control keys, arrows plus enter plus escape, it doesnt read gamepads at all.
- on the frame, steam gives non-steam apps its "keyboard (wasd) and mouse" controller layout, so your controllers show up as keys like w a s d, 1 2 3 4, space, e. FrameTube swaps those for the keys the tv app reads (`src/steam-keys.js`). if steam ever hands the app a real gamepad instead, that works too (`src/gamepad-map.js`).
- steam starts games with its overlay in `LD_PRELOAD` plus its runtime libraries in `LD_LIBRARY_PATH`, and with both chromium dies at start ("Zygote process exited prematurely"). `frametube` is a tiny static launcher (`launcher/launcher.c`, no libc) that drops those two and starts `frametube-electron`.
- electron's setuid sandbox helper gets left out of the build, copied to the frame it isnt owned by root so chromium would refuse to start. without it chromium uses its user namespace sandbox, which steamos allows, so the sandbox stays on.

## build

needs node 22+ and clang with lld (the android ndk's works fine, point `CLANG` at it).

```
npm install
CLANG=/path/to/clang npm run build:launcher
npm test
npm run build          # dist/linux-arm64-unpacked
npm run build:x64      # dist/linux-x64-unpacked, for trying it on a pc
```

`npm start` runs it fullscreen on a desktop, `FRAMETUBE_WINDOWED=1` gives you a normal window, `FRAMETUBE_DEBUG=1` logs every key it translates.

## not affiliated

not made by or affiliated with google, youtube or valve. youtube is a trademark of google.

## license

MIT
