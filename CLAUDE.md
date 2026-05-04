# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

ZMK firmware configuration for the **Eyelash Sofle** split ergonomic keyboard (nice_nano_v2 MCU). ZMK firmware uses a Zephyr RTOS devicetree-based configuration system — keymaps are defined in DTS syntax (`.keymap` files), not code.

## Build & Deploy

**Builds happen exclusively via GitHub Actions** — there is no local build toolchain. Flash the keyboard by downloading the `.uf2` artifacts from a workflow run.

- **Firmware build**: Triggered automatically on push (excluding `keymap-drawer/`), or manually via `workflow_dispatch`. Produces left/right half `.uf2` files plus a settings-reset image.
- **Keymap visualization**: Triggered on changes to `config/`, auto-commits SVG output to `keymap-drawer/` with a `[Draw]` prefixed commit message.

The `settings_reset` build target (`nice_nano_v2` + `settings_reset` shield) is used to clear the keyboard's flash when re-pairing Bluetooth.

## Key Files

| File | Purpose |
|---|---|
| `config/eyelash_sofle.keymap` | All keymap layers and custom behaviors (DTS syntax) |
| `config/eyelash_sofle.conf` | Kconfig settings (RGB, BLE, power, mouse) |
| `config/west.yml` | ZMK version pin (`v0.3.0`) and module dependencies |
| `build.yaml` | Defines which board/shield combos to build |
| `keymap_drawer.config.yaml` | Styling and keycode mappings for SVG generation |
| `boards/arm/eyelash_sofle/` | Custom board definition files |

## Keymap Architecture

The keymap (`config/eyelash_sofle.keymap`) has 8 layers:

| Layer | Name | Purpose |
|---|---|---|
| 0 | Dvorak | Primary layout with mod-morph number/symbol remapping |
| 1 | Base | QWERTY fallback |
| 2 | NAV&Fn | Navigation, function keys |
| 3 | CON | Bluetooth, mouse, RGB, audio controls |
| 4 | Game | Gaming layer |
| 5–7 | CAD | CAD application layers (split across halves + symbols) |

### Custom Behaviors Defined in the Keymap

- **`hml` / `hmr`**: Home row mods (left/right) — hold-tap with positional hold preference
- **`ltthumb`**: Layer-tap optimized for thumb keys
- **`dvorak_*` mod-morphs**: Modifier-activated symbol switching for the Dvorak layer (e.g., `dvorak_1` produces `1` normally, `!` with shift)
- **`mouse_scroll_left`**: Rotary encoder binding for horizontal scroll
- **`change_lang`**: Macro that sends `Cmd+Space` for input language switching

### Mouse / Pointing

Mouse support (`CONFIG_ZMK_POINTING`) is enabled. Move velocity: `ZMK_POINTING_DEFAULT_MOVE_VAL 1200`. Mouse move (`mmv`) and scroll (`msc`) with acceleration are configured in Layer 3.

## ZMK DTS Syntax Notes

- Behaviors are defined under `/ { behaviors { ... }; };`
- Keymaps live under `/ { keymap { compatible = "zmk,keymap"; ... }; };`
- Config knobs (brightness, power, BLE) go in the `.conf` file, not the keymap
- Module dependencies (e.g., `nice-view-battery`) are declared in `config/west.yml`
