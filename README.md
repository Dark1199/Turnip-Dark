# Turnip Dark

**Turnip Dark** is an unofficial, experimental build of the open-source
[Mesa](https://www.mesa3d.org/) **Turnip** Vulkan driver (Mesa 24.3.4),
adjusted for one device family: phones with the **Snapdragon 675** and its
**Adreno 612** GPU. It is meant to be loaded by Switch emulators such as Suyu
or Yuzu through their GPU driver manager.

Current version: **v0.0.3** (`Turnip Dark v0.0.3 by @Dark (Adreno 612 / Snapdragon 675)`).

> **Status:** hobby project, tested on one device (Samsung Galaxy A70, SM6150).
> Results vary by game, scene, emulator build and phone temperature. Nothing
> here is a guaranteed frame-rate target.

---

## Why this exists

Stock Turnip supports many Adreno 6xx/7xx GPUs and is tuned for the broadest
possible set of devices. The Adreno 612 is a low-end part (small on-chip
memory, limited Vulkan features), and emulators push it hard. This project
narrows the driver to that one GPU and tries different low-risk settings to
see what keeps it stable and smooth.

## What is different from upstream Mesa

| Area | Change |
|---|---|
| Identity | Driver name `Turnip Dark`, author `@Dark`, version `0.0.3` in the Vulkan properties. |
| Hardware lock | The driver only accepts **Adreno 612**. Any other GPU is rejected at start-up with `VK_ERROR_INCOMPATIBLE_DRIVER`. The Adreno 7xx device path is removed. |
| Vulkan level | Reports **Vulkan 1.1**. `VK_KHR_multiview` is exposed (required for 1.1), using Turnip's software multiview path because the 612 has no hardware multiview. |
| Conformance | The driver does **not** claim Khronos conformance (conformance version reported as 0.0.0.0). |
| Command streams | Larger initial command-stream chunks, so the driver grows its buffers less often. Inline hints on the hot emit functions. |
| Emulator profile | When the app name looks like Suyu / Yuzu / Sudachi / Citron, a combined profile with larger command chunks is applied automatically. App names containing the game titles also match the per-game profiles. `TU_DARK_PROFILE=darksouls\|ac4\|emulator\|none` overrides it. |
| Stable defaults (`[stable]` build) | Direct rendering (`sysmem`) with UBWC disabled, as defaults. In testing, builds with these settings (or per-draw synchronisation) ran without the GPU device-loss crashes seen with the baseline. The exact cause is not yet isolated. |

### What it does *not* do

- It does not disable robustness, bypass validation, or fake another GPU vendor.
- It does not decode video or change how an emulator handles the network.
- The per-game profiles only change buffer sizes, not the GPU commands, so they
  are safe but modest.
- The Adreno 612 hardware limits remain: no 16-bit / 8-bit storage, 16 vertex
  attributes, one viewport. Emulators will warn about these.

## Games and testing notes

- **Dark Souls: Remastered:** tested. Character creation reaches the 30 FPS cap;
  cutscenes, early gameplay and the first boss fight ran roughly 10-26 FPS,
  depending on the scene. Much of the stutter came from the emulator compiling
  hundreds of pipelines during play, so enabling **asynchronous shaders** and
  playing with a warm shader cache matters more than the driver.
- **Assassin's Creed IV: Black Flag:** profile present, **not yet tested**.

## Installing

1. Download `vulkan.dark.zip` from the latest GitHub Actions run (artifact
   `turnip-dark`) or from Releases.
2. In your emulator, open the GPU driver manager and install the ZIP.
3. Select it, start a game, and check the log for `Turnip Dark`.
4. Keep the same driver build while testing: a new build starts a new, empty
   pipeline cache.

## Building

The workflow in `.github/workflows/build.yml`:

1. Verifies the custom source ZIP in `src/` (set `SRC_ZIP` at the top of the file).
2. Downloads Mesa 24.3.4 from a mirror with retries, falling back to the
   official release tarball.
3. Replaces Mesa's `src/freedreno` with the custom sources.
4. Cross-compiles for Android with the NDK (r27c, API 29, arm64) via Meson and Ninja.
5. Packages `vulkan.dark.so` into `vulkan.dark.zip`.

### How the build avoids Android linking errors

- A fixed NDK cross file means the compiler and linker are always the Android ones.
- `pkg-config` is pointed at an empty directory so Meson never picks up the
  runner's **host** libraries (for example SPIRV-Tools headers), which cannot
  be used in an Android build and caused an earlier failure. Dependencies such
  as zlib and expat come from Meson wraps built for the target.
- Only the single target library (`libvulkan_freedreno.so`) is built, with
  `-static-libstdc++`, so the result has no loose C++ runtime dependency.
- A small retry script reads the compiler log and applies a known fix (such as
  defining `_GNU_SOURCE`) and rebuilds, up to three times. Anything else stops
  with the captured errors uploaded as a `build-log` artifact.

## Credits and license

Turnip and Mesa are the work of the Mesa project and its contributors (Rob
Clark, Google, Valve, Igalia and many others), licensed under the **MIT
license**. Their copyright notices are kept in the source files, as the license
requires. Turnip Dark's modifications are by **@Dark**. This project is not
affiliated with Mesa, Qualcomm, or any emulator project.
