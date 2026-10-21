# Turnip Dark v1.2 (Custom Vulkan Driver)

This is a customized Vulkan driver based on Mesa 24.3.4, modified and compiled specifically for the Snapdragon 675 (Adreno 612) chipset. The goal of this project is to squeeze out maximum performance and bypass standard translation layer bottlenecks in heavy emulators like Suyu or Yuzu.

## IMPORTANT: Read Before Installing
- **Snapdragon 675 ONLY:** This driver is hardcoded exclusively for the Adreno 612 GPU. 
- Do NOT use this on any other Snapdragon processor (like SD 870, SD 8 Gen 1, etc.). It will cause immediate crashes or unstable rendering.
- Best used in rooted Android environments running stable custom ROMs (like crDroid or LineageOS) with proper kernel tweaks.

## Features & Specific Optimizations
The core source code (`src/freedreno/`) has been updated to v1.2 with custom performance profiles hardcoded via `drirc`. It automatically applies fixes when launching specific games inside Suyu:

### 1. Dark Souls 1 Remastered
- **Disabled Robustness:** Strips away background error-checking to gain raw execution speed.
- **Forced Shader Caching:** Forces the driver to cache shaders aggressively, reducing game stutter and frame drops.
- **Target:** Aiming for a stable 25-30 FPS at 544p resolution without using heavy upscalers like FSR.

### 2. Assassin's Creed IV: Black Flag
- **Texture Alignment Fix:** Fixes the infamous Adreno 612 graphics bug that causes black screen rendering and broken textures on the Switch port.
- **Stability Tweaks:** Improves memory pacing to prevent crashes during physics-heavy collisions.

## How it was compiled
Since compiling directly on a smartphone can lead to annoying environment or linking errors with Android system libraries (`liblog`, `libcutils`, `libhardware`), this build is processed through GitHub Actions using a clean Linux environment.

- **Base:** Mesa 24.3.4
- **Modifications applied:** `turnip_dark_v1.2_source.zip`
- **Build tool:** Meson + Ninja

## Installation
1. Go to the Actions or Releases tab and download `vulkan.dark.so`.
2. Open Suyu / Yuzu on your Android device.
3. Go to Settings -> GPU Driver Manager -> Install, and choose the file.
4. For best results, long-press Dark Souls or Assassin's Creed inside the emulator, go to Game Properties, and force this driver to run on them specifically.

Feel free to fork the repo, report issues, or contribute if you know how to optimize older Adreno chips.

**Maintained by:** Dark (Dark1199)
