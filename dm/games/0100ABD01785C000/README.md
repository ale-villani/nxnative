# Portal 2 (`0100ABD01785C000`)

Update 1.0.3 (`v196608`) over the base game, SDK 13.3.0. Valve's Source engine as NVIDIA Lightspeed Studios ported it to the Switch: `main` is a launcher with SDL, and the engine consists of 21 NRO modules in the RomFS (`nro/*.nro`, registered by `.nrr/*.nrr`) loaded dynamically through `nn::ro`. Renders with NVN via `materialsystem.nro`. Modules `main`, `subsdk0` (GLSLC), and `sdk`; `subsdk1` is NVIDIA's OpenGL driver and is never loaded.

**Current:** plays on the Mac at 60 fps: opening sequence, 3D background title scene, chapter selection, full controller input, audio output, and engine threads (`NXVoice_Encoder`, `NXVoice_WorkerThread`, `IOJob0..2`, `main thread`) running at 60 Hz without aborts or crashes.

## What it needs

| Setting | Where | Why |
|---|---|---|
| `modules main subsdk0 sdk` | [`game.tsv`](game.tsv) | `main` imports from no subsdk; the NROs call GLSLC (`subsdk0`) for runtime shader compilation |
| 4 GB layout: `mem_reserve 0x100000000`, `mem_alias 0x10000000`, `mem_stack 0x10000000`, `app_pool 0xCD500000`, `sys_resource 0x0` | [`game.tsv`](game.tsv) | no system resource, so no virtual address memory; the game sets a heap in the 4 GB layout |
| `pad_layout full` | [`game.tsv`](game.tsv) | first-person controls: left stick movement, right stick look |
| `shader_groups /game.zip/=game` | [`game.tsv`](game.tsv) | the shipped programs come from `game.zip`; runtime GLSLC captures go to `other` |
| `hle_audren.c`: `AcquireAudioDeviceNotificationForInput` | `dm/host/hle_audren.c` | Portal 2 initializes audio input for co-op voice chat; the host binds the system event as never signaled, mirroring output notification |

## Shaders

- 1,186 shaders: precompiled GLSLC programs extracted from `game.zip` (`nxcontent/nvsb/*.nvsb`) plus 9 captured at runtime via `glslcCompile`.
- All 1,186 shaders translate and compile cleanly to Metal libraries (`.metallib`).
- Offline pipeline archive built for Apple Silicon GPUs (`applegpu_g13g`, 28.8 MB).

## Files

- The game assets reside in `game.zip` (7.18 GB) alongside the NROs in RomFS.
- `romfs_marker` is `game.zip`.
