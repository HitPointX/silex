# Changelog

All notable silex release milestones are tracked here.

## v0.3.4-alpha - 2026-03-29

Auto save-state and multiplayer lobby scaffold.

### Save States

- added **Auto Save-State** toggle in the File menu, above the State Slot submenu
- when enabled, silex silently saves to your active slot every 5 minutes of real play time
- timer pauses during rewind and scrub; disabled automatically in Hardcore mode
- on/off state persisted across sessions
- toast confirmation on toggle

### Multiplayer (scaffold)

- added **Multiplayer** top-level menu between Settings and Debug
- **Multiplayer > Lobby...** opens a modal with Host and Join tabs
- Host tab: room code display, 4 player slot list, Start Session button
- Join tab: room code input (persisted), slot indicators, Connect button
- Display Name field persisted across sessions
- all network-dependent features are clearly marked as in-development
- concept: shared-world co-op where each connected player gets their own copy of the host's player character; world state synced to host; up to 4 players per session

---

## v0.3.3-alpha - 2026-03-29

Input mapping overhaul, two-player controller support, Nintendo Switch button fix, and multi-monitor DPI window scaling fix.

### Input

- added **Settings > Input** modal with per-port configuration for Port 1 and Port 2
- device dropdown auto-selects the currently connected controller; falls back to keyboard when no controller is detected
- **Automatic Input Mapping** is on by default - SDL GameController mapping applies with no user action required
- **Remapping wizard** walks through all 12 bindings in order: A, B, X, Y, D-Pad Left/Right/Up/Down, Start, Select, L, R
- wizard captures a single keypress or gamepad button per step; any step can be skipped
- on completion the port switches to **custom** mode and the Automatic checkbox clears
- re-enabling Automatic reverts custom bindings and restores SDL-default mapping
- bindings persist across sessions in `config.ini`
- **Port 2 (Player 2) fully wired:** two-player games now work out of the box with no configuration required

### Bug Fixes

- **Nintendo Switch controller A/B/X/Y mapping:** SDL 2.28 changed the default to label-based mapping for Nintendo controllers, which mapped the physical South button as A (Nintendo label) rather than as the SNES B button (position-equivalent). silex now detects Nintendo Switch controllers and automatically corrects the mapping to SNES position-based layout. Xbox, DualSense, and all other controllers are unaffected.
- **Multi-monitor DPI window scaling:** dragging the silex window between monitors with different DPI scales no longer causes the window to grow exponentially. The resize handler now distinguishes compositor-driven DPI rescales from genuine user resizes and holds the intended window size.

---

## v0.3.2-alpha - 2026-03-29

Battery-backed SRAM persistence - saves now survive across emulator sessions.

### SRAM Persistence

- game saves are now written to disk and restored automatically across sessions
- `.srm` path is derived from the ROM path at load time; existing save data is loaded and validated on startup
- dirty tracking ensures only writes to battery-backed RAM (LoROM/HiROM SRAM, Super FX RAM, SA-1 BW-RAM) trigger a flush
- periodic background flush every 60 seconds by default (configurable, range 10–3600 s)
- flush skipped during rewind - only real forward-play frames trigger the periodic check
- forced flush on ROM swap and on application exit so no data is lost on clean shutdown

### Data Safety

- atomic write: SRAM is written to a temp file then renamed - the `.srm` file is never in a partially-written state
- rolling backup: the existing `.srm` is copied to `.srm.bak` before each overwrite, preserving the previous good save
- stale temp files from prior crashes are cleaned up automatically on the next flush

### Frontend

- toast notification `"SRAM saved"` appears after a background flush that wrote data to disk
- toast notification `"SRAM save failed - check disk"` appears on any flush failure
- flush interval is adjustable in Settings

### CLI

- added `--no-srm` flag: disables SRAM disk persistence for the session; intended for headless tooling and scripted runs

---

## v0.3.1-alpha - 2026-03-29

Post-processing shaders, controller scrub binding, CPU decimal mode correctness, and targeted PPU and bus timing fixes.

### Video / Shaders

- implemented a GLSL 3.3 post-process shader pipeline in the OpenGL renderer
- added four built-in shader presets selectable at runtime: Scanlines, CRT Simple, CRT Geom, and Phosphor Glow
- CRT Geom includes barrel distortion, aperture-grille shadow mask, phosphor vertical bleed, and vignette
- Phosphor Glow adds multi-tap horizontal and vertical bloom with a warm phosphor tone
- added a Shader Settings panel in the frontend UI with per-shader description and live switching
- shader selection is disabled and greyed out when a non-OpenGL renderer is active, with a clear notice
- shader preference persisted across sessions

### Input

- added Guide/Home button controller binding for scrub-pause and timeline navigation
- holding Guide/Home pauses emulation at the current frame
- pressing D-pad Left while Guide is held steps backward through rewind history
- pressing D-pad Right while Guide is held advances at fast-forward speed
- D-pad Left/Right are suppressed from normal joypad input while Guide is held to avoid ghost inputs

### CPU

- corrected ADC and SBC decimal mode (BCD) arithmetic for both 8-bit and 16-bit accumulator widths
- binary overflow flag now computed from the intermediate binary path independently of BCD result
- expanded CPU test suite with new BCD arithmetic coverage

### PPU

- added SETINI register (0x2133) write path, previously unhandled
- corrected frame parity bit in STAT register reads

### Memory Bus / Timing

- corrected H-IRQ compare calculation: HTIME compare is now offset by +4 dot-clocks from the register write value, matching hardware behavior

### Platform

- Vulkan renderer hidden from the macOS renderer selection list; Vulkan requires an external ICD that is not bundled with silex and producing an incompatible-driver error at launch on macOS

---

## v0.3.0-alpha - 2026-03-28

Frontend and rendering milestone. This cycle focused on making silex behave more like a complete standalone emulator while keeping the from-scratch core architecture intact.

### Frontend

- expanded the desktop shell into a more practical day-to-day emulator frontend
- added a first-run wizard for initial setup flow
- added native ROM loading with recent-ROM tracking
- added fullscreen flow and more complete in-app settings panels
- added save states, rewind, fast-forward, and scrub/seek controls
- added toast notifications and clearer in-app status feedback
- added bench/stat overlays for FPS, frame pacing, and emulation/upload/swap timing
- improved SDL controller discovery and active input handling

### Video / Rendering

- added multiple renderer paths with runtime selection: `OpenGL3`, `Software`, `Vulkan` (Linux/Windows), `D3D11` (Windows)
- added `Fast PPU` as an optional lower-cost rendering path
- added `Render Cache` as an optional tile/window caching path for extra CPU-side headroom
- added `Threaded Video` as an optional presentation worker for the `Software` renderer
- added macOS `HiDPI` control, off by default on Retina-class systems
- improved renderer startup and fallback behavior
- corrected a broad BG character-base regression that caused scrambled tile fetches in commercial software
- hardened `Threaded Video` by treating `VSync` as incompatible when that combination would stall or freeze presentation

### RetroAchievements

- integrated RetroAchievements login and session flow into the frontend
- added achievement state, score, game badge, and notification plumbing to the UI
- added frontend controls for Hardcore, Encore, Spectator, unofficial sets, and rich presence
- current practical support centered on standard/non-Hardcore play while save-state, rewind, and slow-motion workflows continue to evolve

### Tooling / Workflow

- improved headless screenshot and input-preset workflow for regression capture
- expanded local performance and scene-validation workflow around real commercial titles
- cleaned up macOS build layout so release/debug outputs are separated predictably
- stabilized release-build workflow and dependency reconfigure behavior on macOS

---

## v0.2.0-alpha - 2026-03-15

Audio milestone. SPC700 APU core and DSP voice engine integrated end-to-end; audio output confirmed correct in multiple commercial titles.

### APU / SPC700

- SPC700 CPU core implemented from scratch against the SPC700 reference manual
- full opcode table wired with hardware-accurate cycle counts
- timer subsystem implemented: all three hardware timers (T0/T1/T2) with prescaler and output counter behavior
- SPC I/O port handshake fully wired between the main CPU bus and the APU

### DSP

- DSP register model implemented
- BRR (Bit Rate Reduction) sample decoder implemented, including block header, filter coefficients, and all four filter modes
- eight-voice sample engine: key-on, key-off, ENDX, per-voice pitch accumulator, BRR lookahead and loop handling
- voice mixing and stereo output at 32 kHz
- ADSR and GAIN envelope paths implemented
- echo / reverb buffer path implemented
- master volume, per-voice volume, and pitch modulation path present

### Audio Pipeline

- SDL2 audio device opened at 32 kHz stereo S16
- ring buffer audio path from DSP output to SDL callback with volume scaling

### Timing Fixes

- corrected BRR block header bit ordering: end flag = bit 0, loop flag = bit 1 (was reversed, causing voices to die after four samples in most commercial titles - root cause of BGM silence across all tested ROMs)
- corrected SPC instruction tick unit scaling: instruction cycle counts were in a 2× internal scale and were passed directly to the timer and DSP accumulators, causing computation loops to consume 2× the expected timer ticks and producing audible BGM slowdown
- per-instruction CPU/SPC interleaving: APU now receives proportional ticks after each CPU instruction rather than once per scanline, eliminating the port-handshake stall that occurred during sound data uploads

### Validation Additions

- Super Mario World intro music: confirmed correct playback with no slowdown
- F-Zero Mute City 1 BGM: confirmed correct playback with no slowdown
- The Legend of Zelda: A Link to the Past audio: confirmed correct, no regression against existing video progress

### Timing Metrics (post-fix)

- T0 miss rate (F-Zero): 58% → 7.4%
- T0 miss rate (SMW): 1.3% → 0.06%
- DSP sample rate: stable at ~32 kHz across all tested titles

---

## v0.1.0-alpha - 2026-03-13

First versioned development snapshot.

### Project

- established the first formal versioned baseline for silex
- added compatibility tracking for validated software milestones
- refreshed project documentation around current scope, workflow, and roadmap

### Emulator Progress Snapshot

- CPU core brought to a real validation baseline with automated tests
- bus, cartridge, DMA, interrupt, and joypad paths established
- headless execution, tracing, and screenshot tooling available
- PPU path progressed from fallback output to meaningful commercial-game scene rendering
- sprite rendering, window masking, color-window logic, and multiple scene-composition fixes integrated
- commercial regression testing actively underway against `The Legend of Zelda: A Link to the Past` and `Donkey Kong Country`

### Validation Snapshot

- PeterLemon CPU validation suite remains the CPU baseline
- LttP has moved well beyond boot/title-only output into broader intro/demo rendering
- DKC has moved from black-screen intro failure into visible scene progression and sprite output

### Notes

This release marks the point where silex becomes a versioned emulator project with active compatibility tracking rather than an unversioned workbench snapshot.
