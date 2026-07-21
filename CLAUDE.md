# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Hardware

- **Board**: seeeduino_xiao_ble (both halves)
- **Form factor**: 42-key split Corne-style (beekeeb Toucan), 6×3 + 3 thumb keys per side
- **Left half**: nice_view_gem display, Cirque Pinnacle trackpad (via `cirque-input-module` from geeksville's `toucan` branch), ZMK Studio enabled
- **Right half**: rgbled_adapter only (no display)
- **ZMK version**: v0.3 (pinned in `config/west.yml`)

## Building firmware

There is no local build — all firmware is built by GitHub Actions on every push. To trigger a build, push a commit. Artifacts appear under the Actions tab as downloadable `.uf2` files.

Build matrix is defined in `build.yaml`:
- `toucan_left` + `rgbled_adapter` + `nice_view_gem` with ZMK Studio snippet
- `toucan_right` + `rgbled_adapter`
- `settings_reset` (factory reset firmware)

## Key files

| File | Purpose |
|------|---------|
| `config/toucan.keymap` | **Main keymap — edit this** |
| `boards/shields/toucan/toucan.dtsi` | Hardware layout, matrix transform, trackpad input pipeline |
| `boards/shields/toucan/toucan_left.overlay` | Left-half pin assignments, SPI for display, enables glidepoint listener |
| `boards/shields/toucan/toucan_right.overlay` | Right-half pin assignments, Cirque Pinnacle SPI device definition |
| `boards/shields/toucan/toucan_left.conf` | Left Kconfig (pointing, sleep, BLE battery proxy) |
| `boards/shields/toucan/toucan_right.conf` | Right Kconfig (pointing, sleep) |
| `config/west.yml` | ZMK version pin + external module dependencies |

## Keymap layer structure

Defined in `config/toucan.keymap`:

| Layer | Index | How to activate |
|-------|-------|-----------------|
| BASE     | 0 | Default |
| GRAPHITE | 1 | Toggled on/off via combo (`N`+`M` on BASE/GRAPHITE, positions 30+31) |
| NAV      | 2 | Hold left thumb `mo NAV` |
| SYM      | 3 | Hold right thumb `mo SYM` |
| ADJ      | 4 | Dedicated right-thumb key `mo ADJ` (no longer a NAV+SYM combo) |
| G Layer  | 5 | Hold `G` key (`lt G_LAYER G` on home row) |
| NUM      | 6 | Hold `Z` key (`lt NUM Z` on BASE) / `Q` key (GRAPHITE) |
| Orange   | 7 | Combo on BASE/GRAPHITE (positions 34+35, the two rightmost row-3 keys) |

GRAPHITE is a full alternate letter layout for typing practice, added after the original NAV/SYM/ADJ numbering — layer indices shifted accordingly, so don't assume `mo 1`/`mo 2` refer to NAV/SYM anymore.

### Custom behaviors

- **`lp_cpy`** (on `C`/`V`, BASE & GRAPHITE): tap = bare key, hold (500ms, tap-preferred) = `LG(C)`/`LG(V)` (⌘C/⌘V).
- **`hrm`** (home row mods, balanced flavor, 250ms term): tap = letter, hold = modifier. BASE uses A/S/D/F → Ctrl/Alt/Gui/Shift and J/K/L/`;` → Shift/Gui/Alt/Ctrl; GRAPHITE remaps the same modifiers onto N/R/T/S and H/A/E/I.
- **`hyper_meh`** (mod-morph, bottom-left pinky, BASE & GRAPHITE): tap alone = Hyper (⌘⌥⌃⇧), tap while holding Shift = Meh (⌥⌃⇧, no ⌘).
- **`td_slash_caps`** (tap-dance, bottom-right pinky, 200ms window, BASE & GRAPHITE): 1 tap = `/`, 2 taps = Caps Lock.
- **`notion_*` macros** (ADJ layer, top two rows): type Notion slash-commands/markdown shortcuts and apply text color/bold/italic — e.g. `notion_callout`, `notion_todo`, `notion_bold_red`, `notion_highlight_yellow`. See macro definitions at the top of `config/toucan.keymap`.

### Combos

Defined in `config/toucan.keymap`:
- `D`+`F` and `J`+`K` (home row, 50ms) → `Esc`
- `B`+`,`/`<` (row 3, BASE/GRAPHITE only) → toggles GRAPHITE layer
- Two rightmost row-3 keys (BASE/GRAPHITE only) → momentary Orange (F-key) layer

## Trackpad (Cirque Pinnacle) architecture

The trackpad lives physically on the right half but is used by the left (central) half via ZMK's input-split mechanism:

1. **Right half** (`toucan_right.overlay`): defines the `glidepoint` SPI device (`cirque,pinnacle`), `sensitivity = "2x"`, `x-invert` enabled.
2. **`toucan.dtsi`**: defines `glidepoint_split` (the split proxy) and `glidepoint_listener` (the input pipeline). The listener is **disabled** by default; the left overlay enables it.
3. **Left half** (`toucan_left.overlay`): sets `&glidepoint_listener { status = "okay"; }`.

### Tuning pointer and scroll speed

All tuning is in `boards/shields/toucan/toucan.dtsi` inside `glidepoint_listener`:

```c
glidepoint_listener: glidepoint_listener {
    input-processors = <&zip_xy_scaler 250 100>;   // pointer speed: numerator/denominator (= 2.5×)
    scroller {
        layers = <1 2>;                             // scroll active on layer indices 1 and 2
        input-processors = <
            &zip_xy_to_scroll_mapper
            &zip_scroll_scaler 1 7                  // scroll speed: 1/7 (lower = slower)
            &zip_scroll_transform INPUT_TRANSFORM_X_INVERT
        >;
    };
};
```

- **Pointer speed**: change `zip_xy_scaler <num> <den>` — larger ratio = faster cursor.
- **Scroll speed**: change `zip_scroll_scaler <num> <den>` — e.g. `1 3` is faster than `1 7`.
- **Hardware sensitivity**: set in `toucan_right.overlay` on the `glidepoint` node — `sensitivity = "1x"`, `"2x"`, or `"4x"`.

> ⚠️ **Layer-number drift**: `layers = <1 2>` was written when NAV=1, SYM=2. After the GRAPHITE layer was inserted, index 1 is now GRAPHITE and index 2 is NAV — so scrolling currently activates on GRAPHITE+NAV, not NAV+SYM as originally intended. If you want scroll back on NAV+SYM, change this to `layers = <NAV SYM>` (or `<2 3>`).

## Display (nice_view_gem)

Custom shield in `boards/shields/nice_view_gem/`. Widgets: battery, layer name, output/BLE profile, sleep indicator. Font assets (QuinqueFive) are pre-compiled C files. Modify `custom_status_screen.c` and the `widgets/` files to change what the display shows.

## ZMK Studio

Enabled on the left half only (see `build.yaml` snippet `studio-rpc-usb-uart` and `cmake-args: -DCONFIG_ZMK_STUDIO=y`). Connect the left half via USB to use ZMK Studio for live keymap editing without reflashing.
