# TyrQuake Libretro Core for DataFrog SF2000 & GB300 (FrogUI)

Optimized and customized build of **TyrQuake (Libretro)** specifically tuned for the **DataFrog SF2000** and **GB300** handheld retro gaming consoles running standard Multicore or **FrogUI**.

---

## 🎮 Overview

The DataFlyer SF2000 and GB300 are ultra-budget handhelds powered by an Actions Semiconductor ATS2825 / HCW5880 single-core MIPS32 CPU with **no hardware FPU (soft-float only)**, **32 MB total RAM**, a 16 KB L1 data cache, and a **320x240 LCD**.

Running 3D software rendering on this platform requires extreme optimizations. This fork adapts TyrQuake to run smoothly and stably on SF2000 and GB300 hardware.

---

## 🚀 Key Optimizations & Features

1. **Tailored Memory Footprint (Zero OOM / Freeze Protection):**
   - **Hunk Size:** Fixed at `16 MB` (`DEFAULT_MEMSIZE_MB = 16`). This prevents `Hunk_Alloc` out-of-memory crashes when loading full maps (`start.bsp`, `e1m1.bsp`), monster models, entity scripts, and sound banks.
   - **Surface Cache (Surfcache):** Dynamically sized with a `2 MB` minimum on SF2000 (saving 8 MB compared to the upstream 10 MB default) avoiding memory exhaustion while preventing cache thrashing.
   - **Rover End-of-Memory Protection:** Robust safety fallback in `D_SCAlloc` that flushes caches (`D_FlushCaches()`) instead of crashing when surface cache wraps around.
   - Total runtime RAM consumption stays around **~18.3 MB**, leaving plenty of headroom in the console's 32 MB space.

2. **32-Bit Packed Screen Blitting:**
   - Rewritten `VID_Update` palette blitter: batches and writes pairs of 16-bit RGB565 pixels as single 32-bit words (`uint32_t`) across the MIPS bus, reducing framebuffer presentation time by over 50%.

3. **Optimized Soft-Float Math & Compiler Flags:**
   - Built with MIPS32 GCC using `-O3`, `-ffast-math`, `-fsingle-precision-constant`, `-fno-math-errno`, `-fno-trapping-math`, `-fomit-frame-pointer`, `-finline-functions`, and `-funroll-loops`.

4. **Lower Audio Overhead (22.05 kHz):**
   - Default audio mixing rate is set to **22,050 Hz** on SF2000/GB300. This halves audio mixer CPU usage and prevents sound stuttering/crackling during heavy 3D scenes.

5. **Optimal Default CVar Performance Profile:**
   - Automatically enables performance-friendly defaults at launch:
     - `r_fastsky 1` (flat sky rendering, eliminates 2-layer cloud animation overhead)
     - `r_dynamic 0` (disables dynamic surface lighting recalculations on explosions)
     - `r_shadows 0` (disables polygon entity shadows)
     - `r_waterwarp 0` / `d_warp 0` (disables heavy trigonometric underwater screen warping)
     - `r_wateralpha 1.0` (opaque water, avoids 2-pass translucent liquid rendering)

6. **Stripped Bloat:**
   - Removed unneeded external audio decoders (MP3/FLAC/Ogg/MikMod) and CD audio emulation (replaced with lightweight stubs).
   - Removed unused networking drivers while preserving the internal loopback driver for singleplayer.

---

## 🛠️ How to Build

This core is built using the MIPS toolchain inside the `gb300_multicore` environment.

### Requirements:
- Linux / WSL (Ubuntu/Debian) or Docker
- MIPS32 toolchain (`mips-mti-elf-gcc` / `mips-mti-elf-g++`)
- `make`, `python3`

### Build Steps:
1. Place this `tyrquake` folder inside `gb300_multicore/cores/tyrquake`. and use https://github.com/Trademarked69/sf2000_multicore as buildsystem.
2. Run the build command for your target:
   ```bash
   # Build static core library
   make -C cores/tyrquake platform=sf2000

   # Build core.elf and generate SF2000 / GB300 binaries
   make core_87000000 TARGET_CORE=tyrquake
   ```
3. Use the multicore python packing script to produce `.sf2k` files:
   - `tyrquake_sf2000.sf2k`
   - `tyrquake_gb300.sf2k`

---

## 📁 Installation & SD Card Setup

### 1. Copy the Core
- **FrogUI / GB300:** Copy `tyrquake_gb300.sf2k` (or `tyrquake_sf2000.sf2k`) to your SD card under `cores/` or configure it in your FrogUI core list.

### 2. Add Quake Game Data
1. On your SD card, create a folder for Quake (e.g. `ROMS/quake/` or `ROMS/ports/quake/`).
2. Create an `id1` subfolder:
   ```
   sdcard/
   └── ROMS/
       └── quake/
           └── id1/
               ├── pak0.pak
               └── pak1.pak   (optional, full version)
   ```
   > **Note:** `pak0.pak` (Shareware Quake Episode 1) is freely downloadable. For the full campaign, add `pak1.pak` from your retail Quake installation (Steam, GOG, or CD).

3. Launch the game from FrogUI or the SF2000 menu by selecting `pak0.pak` or your custom `.gba` / ROM launcher stub.

---

## 🕹️ Controls (SF2000 / GB300 Layout)

| Handheld Button | Quake Action |
| :--- | :--- |
| **D-Pad Up / Down** | Move Forward / Backward |
| **D-Pad Left / Right** | Turn Left / Right |
| **A** | Jump |
| **B** | Attack / Fire Weapon |
| **X** | Next Weapon (`impulse 10`) |
| **Y** | Previous Weapon (`impulse 12`) |
| **L** | Strafe Left |
| **R** | Strafe Right |
| **Start** | Menu / Escape |
| **Select** | Toggle Status Bar / Scores |

---

## 📜 License

This project is licensed under the **GNU General Public License v2 (GPLv2)**, matching the original Id Software Quake source and TyrQuake project licenses.
