`
# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.
``

Repository purpose
- This repo is the official ZMK user configuration for the Corne-ish Zen V2 split keyboard. It defines boards/variants to build, the keymap and behaviors, and minimal firmware config, while delegating the build system to upstream ZMK.

Key files
- .github/workflows/build.yml: delegates CI builds to zmkfirmware/zmk’s reusable workflow.
- build.yaml: declares which boards/variants are built by CI (v2 left/right, plus a left-with-studio variant; v1 boards are present but commented out).
- config/west.yml: west manifest pinning ZMK to main and pointing self to config/.
- config/corneish_zen.keymap: layers, behaviors, and physical layout for the keyboard.
- config/corneish_zen.conf: additional ZMK settings (e.g., idle sleep timeout).

Common commands
CI builds (recommended)
- Trigger a manual build of the reusable workflow:
  - gh workflow run "Build ZMK firmware"
- Inspect recent runs and download artifacts:
  - gh run list --workflow "Build ZMK firmware" --limit 5
  - gh run view <run-id>
  - gh run download <run-id> --dir ./artifacts
- Artifacts: a firmware.zip containing two .uf2 files for left/right and, if enabled, a separate artifact for the Studio-enabled left variant (see build.yaml).

Local builds (advanced)
- This repo includes a west manifest; you can build locally via west:
  - west init -l config
  - west update
  - west build -s zmk/app -b corneish_zen_v2_left -d build/left
  - west build -s zmk/app -b corneish_zen_v2_right -d build/right
- Notes:
  - The manifest tracks zmk@main and sets self: path: config, so -l config ties this repo’s config into the workspace.
  - To target v1 PCBs, uncomment the v1 entries in build.yaml and adjust local board targets accordingly (corneish_zen_v1_left/right).

Flashing
- Double-tap reset on each half to enter bootloader, then drag-and-drop the corresponding .uf2 onto the mounted volume.
- If you only changed the keymap (config/corneish_zen.keymap), flash the left .uf2 only; if you changed config (config/corneish_zen.conf), flash both halves.

High-level architecture
- Build orchestration
  - CI: .github/workflows/build.yml invokes zmkfirmware/zmk’s build-user-config workflow, which reads build.yaml and produces UF2 artifacts per board/variant.
  - Local: west (Zephyr’s meta-tool) assembles a workspace using config/west.yml, pulling ZMK sources and building zmk/app with this repo as the ZMK_CONFIG via the manifest’s self path.
- Targets and variants (build.yaml)
  - corneish_zen_v2_left
  - corneish_zen_v2_right
  - corneish_zen_v2_left_with_studio: enables CONFIG_ZMK_STUDIO and a USB UART RPC snippet; publishes under a distinct artifact name.
  - corneish_zen_v1_left/right are present but commented for older PCBs.
- Keymap and behaviors (config/corneish_zen.keymap)
  - Physical layout: chosen foostan Corne 6-column layout.
  - Custom behavior: balanced_homerow_mods (hold-tap, tap-preferred, 240ms tapping-term) applied to A/S/T and N/E/I positions to provide home-row mods without nested statements; declared under /behaviors.
  - Layers:
    - default_layer ("CLMK"): primary typing layer (Colemak-like), with MO(1) lower and MO(2) raise.
    - lower_layer ("NUMBER"): numbers, Bluetooth controls (bt bt_sel 0–4, bt bt_clr), arrows, Studio unlock.
    - raise_layer ("SYMBOL"): punctuation/symbols including shifted variants and navigation symbols.
- Firmware config (config/corneish_zen.conf)
  - Sets CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=3600000 (1 hour), impacting power management on both halves.

Operational tips specific to this repo
- Prefer CI builds unless you are iterating on ZMK itself; the upstream workflow reliably produces the correct UF2 artifacts as firmware.zip.
- To enable or disable studio/rpc builds, adjust the include entries in build.yaml. Artifact naming for Studio is set via artifact-name.
- For PCB version changes (v1 vs v2), toggle the appropriate board entries in build.yaml and align local west build targets if building locally.

Important notes from README (condensed)
- This configuration targets V2 PCBs (R3 group buy). For V1 PCBs, uncomment the v1 boards in build.yaml.
- After CI completes, download firmware.zip from the workflow’s Artifacts section and flash the appropriate .uf2 to each half. Only reflash the left if you changed only the keymap; reflash both if you changed config.
- For broader ZMK development, consult the official ZMK docs and repo referenced in README.
