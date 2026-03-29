<div align="center">

# silex

**A from-scratch Super Nintendo emulator built on documented hardware behavior.**

![Version](https://img.shields.io/badge/version-v0.3.2--alpha-blueviolet?style=flat-square)
![Language](https://img.shields.io/badge/language-C%2B%2B17-0ea5e9?style=flat-square)
![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20Windows-555555?style=flat-square)
![Status](https://img.shields.io/badge/status-active%20development-f59e0b?style=flat-square)
![License](https://img.shields.io/badge/license-see%20LICENSE-10b981?style=flat-square)

</div>

---

silex is a from-scratch SNES emulator written in C++17. Every subsystem is built directly from hardware documentation — no borrowed cores, no pasted logic, no close-enough guesses. The goal is a clean, maintainable codebase that reaches hardware-accurate behavior through disciplined, documented implementation.

The frontend is built for real use: immediate boot-to-game, low input latency, a retro-inspired feel, and a full toolset to validate behavior against real commercial software.

## Contents

- [What's New](#whats-new)
- [Status](#status)
- [Platform & Renderer Support](#platform--renderer-support)
- [Frontend Features](#frontend-features)
- [Implemented Systems](#implemented-systems)
- [Building](#building)
- [Tooling](#tooling)
- [Compatibility & Bug Hunting](#compatibility--bug-hunting)
- [Roadmap](#roadmap)
- [Philosophy](#philosophy)
- [Contact](#contact)

---

## What's New

### `v0.3.2-alpha` — SRAM Persistence

Game saves now survive across sessions. Save data is loaded automatically when a ROM opens, written to disk in the background every 60 seconds, and flushed on exit so no progress is ever lost on a clean shutdown.

Writes are **atomic** — data goes to a temp file, then renames over the destination. A **rolling backup** preserves the previous good save before each new write. The flush interval is adjustable in Settings. Use `--no-srm` to disable persistence entirely for headless or scripted runs.

### `v0.3.1-alpha` — Shaders, Scrub Controls, CPU Fixes

Post-processing shader pipeline (OpenGL), Guide/Home controller scrub-pause binding, corrected BCD decimal-mode arithmetic, SETINI register, H-IRQ timing correction, and expanded PPU tracing.

---

## Status

### Core Subsystems

| Subsystem | Status | Notes |
|---|:---:|---|
| **65C816 CPU** | ✅ | Full opcode table, all addressing modes, NMI/IRQ/BRK/COP, correct BCD decimal mode |
| **Cartridge / Bus** | ✅ | LoROM, HiROM, WRAM, hardware register windows, SRAM persistence |
| **DMA / HDMA** | ✅ | General DMA channels, multiple transfer modes, HDMA state and execution |
| **PPU** | 🔧 | BG modes, OBJ/OAM, windowing, color math, Mode 7 — ongoing correctness work |
| **SPC700 APU** | ✅ | From-scratch core, full opcode table, hardware-accurate cycle counts, all three timers |
| **DSP** | ✅ | BRR decode, 8 voices, ADSR/GAIN, echo/reverb, stereo master output |
| **Audio Pipeline** | ✅ | SDL2 at 32 kHz stereo S16, ring buffer, volume control |
| **Interrupt Path** | ✅ | NMI, IRQ, auto-joypad wired |
| **Joypad / Input** | ✅ | Keyboard + SDL GameController, auto-joypad register, Guide/Home scrub binding |

> ✅ Implemented and validated &nbsp;|&nbsp; 🔧 Active correctness work

### Frontend

| Feature | Status |
|---|:---:|
| SDL2 desktop shell + Dear ImGui | ✅ |
| First-run wizard | ✅ |
| ROM picker + recent ROM list | ✅ |
| Fullscreen mode | ✅ |
| Save / load state slots | ✅ |
| Rewind + fast-forward + scrub | ✅ |
| Guide/Home controller scrub binding | ✅ |
| SRAM auto-save (atomic writes, rolling backup) | ✅ |
| Post-process shader pipeline | ✅ OpenGL only |
| RetroAchievements integration | ✅ Standard play |
| Performance HUD | ✅ |
| Toast notification system | ✅ |

---

## Platform & Renderer Support

### Renderers

| Renderer | macOS | Linux | Windows | Notes |
|---|:---:|:---:|:---:|---|
| `OpenGL3` | ✅ | ✅ | ✅ | Primary renderer; full shader pipeline available |
| `Software` | ✅ | ✅ | ✅ | Optional threaded presentation worker |
| `Vulkan` | — | ✅ | ✅ | Requires a Vulkan-capable driver |
| `D3D11` | — | — | ✅ | Windows-native path |

### Shader Presets (OpenGL only)

| Preset | Description |
|---|---|
| **Scanlines** | Classic horizontal scanline overlay |
| **CRT Simple** | Light curvature and scanline blending |
| **CRT Geom** | Barrel distortion, aperture-grille mask, phosphor bleed, vignette |
| **Phosphor Glow** | Multi-tap bloom with a warm phosphor tone |

---

## Frontend Features

### Controls

- keyboard and SDL GameController merged into a single unified input path
- SDL GameController covers the vast majority of real-world pads automatically
- **Guide / Home binding:** hold Guide/Home to pause at the current frame; D-pad Left / Right steps backward through rewind history or advances at fast-forward speed

### Save System

| Feature | Detail |
|---|---|
| Save states | Multiple slots, save and load at any time |
| Rewind | Continuous history buffer, scrub-accessible |
| SRAM auto-save | Background flush every 60 s by default (adjustable: 10–3600 s) |
| Atomic writes | Temp file + rename — the `.srm` file is never partially written |
| Rolling backup | Previous good save preserved as `.srm.bak` before each write |
| `--no-srm` | Disables disk persistence for headless and scripted use |

### Notifications

Each category is individually toggleable in Settings:

| Category | Default | Covers |
|---|:---:|---|
| System | ✅ On | Renderer, PPU mode, shader, and video events |
| Save States | ✅ On | Save and load state confirmations |
| Achievements | ✅ On | Unlocks, mastery, and session events |
| SRAM | Off | Background save confirmations and write failures |

---

## Implemented Systems

<details>
<summary><strong>CPU — WDC 65C816</strong></summary>
<br>

- reset, native, and emulation mode behavior
- full opcode table wired
- all addressing modes: direct page, absolute, long, indirect, stack-relative, native-mode variants
- NMI / IRQ / BRK / COP handling
- correct BCD decimal-mode ADC and SBC for 8-bit and 16-bit accumulator widths
- instruction tracing and disassembly support

</details>

<details>
<summary><strong>Memory / Cartridge / Bus</strong></summary>
<br>

- LoROM and HiROM cartridge loading
- WRAM mapping and mirrors
- hardware register windows for PPU / CPU I/O / DMA
- multiplication / division CPU registers
- joypad serial and auto-joypad register reads
- APU I/O port handshake fully wired
- battery-backed SRAM persistence: dirty tracking, atomic writes, rolling backup, configurable flush interval

</details>

<details>
<summary><strong>DMA / HDMA</strong></summary>
<br>

- general DMA channels with multiple transfer modes
- channel stepping and register updates per hardware spec
- HDMA state and execution path

</details>

<details>
<summary><strong>PPU</strong></summary>
<br>

- VRAM, CGRAM, and OAM storage
- scanline-based framebuffer generation
- BG layer rendering across modes exercised by the active validation set
- OBJ / OAM rendering with priority handling
- color math and fixed color support
- main / sub screen window masking and color-window control
- Mode 7 multiply register reads
- SETINI (0x2133) write path

</details>

<details>
<summary><strong>APU / SPC700</strong></summary>
<br>

- SPC700 CPU core implemented against the SPC700 reference manual
- full opcode table with hardware-accurate cycle counts
- three-timer subsystem (T0/T1/T2) with prescaler and output counter behavior
- SPC I/O port handshake wired between the main CPU bus and the APU
- per-instruction CPU/SPC interleaving for correct port timing during data uploads

</details>

<details>
<summary><strong>DSP</strong></summary>
<br>

- DSP register model
- BRR (Bit Rate Reduction) decoder: block headers, all four filter modes, loop/end flags
- eight-voice sample engine: key-on, key-off, ENDX, per-voice pitch accumulator, BRR lookahead and loop
- voice mixing and stereo master output
- ADSR and GAIN envelope paths
- echo / reverb buffer
- master volume, per-voice volume, pitch modulation

</details>

<details>
<summary><strong>Audio Pipeline</strong></summary>
<br>

- SDL2 audio device at 32 kHz stereo S16
- ring buffer from DSP output to SDL callback
- volume control with integer scaling

</details>

---

## Building

### Prerequisites

- CMake 3.20+
- C++17 compiler (Clang, GCC, or MSVC)
- SDL2, MiniZip, OpenGL (platform-specific; see CMake config for details)

### macOS

```bash
# Release
cmake --preset macos-release
cmake --build --preset macos-release -j8

# Debug
cmake --preset macos-debug
cmake --build --preset macos-debug -j8
```

### Linux / Windows

```bash
cmake -S . -B build
cmake --build build -j4
ctest --test-dir build --output-on-failure
```

### Running

```bash
# Launch with a ROM
./build/silex path/to/game.sfc

# Headless with CPU trace
./build/silex --headless path/to/game.sfc --trace

# Headless screenshot capture at frame 280
./build/silex --headless path/to/game.sfc --max-frames 300 --screenshot-frame 280 --screenshot-out /tmp/frame.png

# Disable SRAM persistence
./build/silex path/to/game.sfc --no-srm
```

---

## Tooling

silex ships with a practical diagnostic toolset for validating hardware behavior. The primary development workflow is: **reproduce → trace → confirm from docs → fix → re-validate**.

### Environment Variables

| Variable | Effect |
|---|---|
| `SILEX_SPC_TIMER_TRACE=1` | SPC700 timer tick trace to stdout |
| `SILEX_DSP_TRACE=1` | DSP register and voice state trace |
| `SILEX_OAM_ACTIVE_TRACE_FRAME=N` | OAM write activity dump on frame N |
| `SILEX_PIXEL_TRACE_FRAME=N` | Pixel-level composition trace on frame N |
| `SILEX_PPU_MODE_TRACE=1` | PPU mode register write trace |
| `SILEX_PPU_SCROLL_TRACE=1` | BG scroll register write trace |

### CLI Flags

| Flag | Effect |
|---|---|
| `--headless` | Run without a display window |
| `--trace` | Enable CPU instruction trace |
| `--max-frames N` | Stop after N emulated frames |
| `--screenshot-frame N` | Capture a screenshot on frame N |
| `--screenshot-out path` | Output path for the captured screenshot |
| `--no-srm` | Disable SRAM persistence for this session |

---

## Compatibility & Bug Hunting

Full compatibility status is tracked in **[COMPATIBILITY.md](COMPATIBILITY.md)**.

### Current Validation Set

| Title | Status |
|---|---|
| Chrono Trigger | 🟡 Partial Gameplay |
| Donkey Kong Country | 🟡 Partial Gameplay |
| F-Zero | 🟡 Partial Gameplay |
| Final Fantasy IV | 🟡 Partial Gameplay |
| The Legend of Zelda: A Link to the Past | 🟡 Partial Gameplay |
| Super Mario World | 🟡 Partial Gameplay |
| Super Metroid | 🟡 Partial Gameplay |
| PeterLemon CPU suite | ✅ Validated |

See [COMPATIBILITY.md](COMPATIBILITY.md) for per-title notes, known remaining issues, and the status legend.

---

### Helping Find Bugs

If a game doesn't run correctly, the tools below help isolate what's wrong. A good bug report saves a lot of time — the ideal one tells you exactly which subsystem to look at.

#### Step 1 — Reproduce headlessly

```bash
./build/silex --headless game.sfc --max-frames 600 --screenshot-frame 590 --screenshot-out /tmp/result.png
```

Run without a display and capture a screenshot at a known frame. This gives a clean, reproducible baseline independent of the live UI.

#### Step 2 — Enable full CPU trace

```bash
./build/silex --headless game.sfc --trace > trace.txt 2>&1
```

Full CPU instruction trace. Output is large — pipe to a file and search for unexpected register state, wrong opcode sequences, or incorrect branch behavior near the failure point.

#### Step 3 — Narrow with targeted traces

**Audio issues (silence, wrong pitch, crackling):**
```bash
SILEX_SPC_TIMER_TRACE=1 SILEX_DSP_TRACE=1 \
  ./build/silex --headless game.sfc --max-frames 300
```

**PPU / rendering issues (wrong tiles, scrambled backgrounds, color errors):**
```bash
SILEX_PPU_MODE_TRACE=1 SILEX_PPU_SCROLL_TRACE=1 SILEX_PIXEL_TRACE_FRAME=120 \
  ./build/silex --headless game.sfc --max-frames 125
```

**Sprite problems (missing or corrupted OBJ):**
```bash
SILEX_OAM_ACTIVE_TRACE_FRAME=120 \
  ./build/silex --headless game.sfc --max-frames 125
```

#### What makes a useful report

| Field | What to include |
|---|---|
| **Title + region** | e.g. `Super Mario World (USA)` |
| **Scene / frame** | Where the issue first appears |
| **Expected vs actual** | What hardware does vs. what silex does |
| **Screenshot** | Captured with `--screenshot-out` at the failing frame |
| **Save state** | Save state at or just before the failing scene (use the in-app save state slots, then attach the file) |
| **Trace excerpt** | The relevant window of CPU or subsystem trace output |

Send reports to **`dev@hitpoint.pro`** with the subject line `silex bug — <Title>`. Attaching a save state that lands right at the problem frame cuts diagnosis time significantly.

> **Hardware reference:** [fullsnes](https://problemkaputt.de/fullsnes.htm) is the primary register-level reference used to validate silex behavior. If something looks wrong, it's the first place to check expected hardware output.

---

## Roadmap

### Near-term

| # | Work Item |
|:---:|---|
| 1 | PPU correctness — LttP file-name screen, DKC gameplay gaps |
| 2 | APU / DSP edge cases and additional title coverage |
| 3 | Input and progression path stability |
| 4 | Expand curated compatibility set |

### Mid-term

| # | Work Item |
|:---:|---|
| 1 | Broader commercial compatibility coverage |
| 2 | Frontend usability and controller support depth |
| 3 | Save / run-state tooling for debugging and speedrun workflows |
| 4 | Modding and randomizer-friendly workflow support |

### Later

| # | Work Item |
|:---:|---|
| 1 | Backend-assisted features where justified |
| 2 | LAN-oriented experiments |
| 3 | Richer library and workflow tooling |

---

## Philosophy

silex is built strictly from documentation.

- No borrowed runtime cores
- No pasted emulator logic
- No close-enough hardware guesses when the behavior is already documented
- No hardcoded per-game hacks as substitutes for missing architecture

### Reference Material

| Document | Covers |
|---|---|
| `fullsnes` | Full SNES hardware register map, timing, and subsystem behavior |
| anomie's register docs | PPU, DMA, and I/O register details |
| WDC 65C816 manual | CPU instruction set, addressing modes, and timing |
| SPC700 reference manual | APU instruction set, timers, and I/O port behavior |

### Implementation Priority

1. correct structure
2. reproducible debugging
3. stable progression through real commercial software
4. performance tuning after correctness

---

## Author

silex is a solo project by **HitPointX**.

- Website: [hitpoint.pro](https://hitpoint.pro)
- Email: `dev@hitpoint.pro`

## License

See [LICENSE](LICENSE).
