# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Hardware

- **Board**: seeeduino_xiao_ble (both halves)
- **Form factor**: 42-key split Corne-style (beekeeb Toucan), 6×3 + 3 thumb keys per side
- **Left half**: nice_view_gem display, ZMK Studio enabled
- **Right half**: rgbled_adapter, Azoteq TPS43 trackpad (via `zmk_driver_azoteq` + `zmk-input-zoom` from beekeeb, I2C) — Toucan2 hardware revision
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
| `boards/shields/toucan/toucan_left.overlay` | Left-half pin assignments, SPI for display, enables trackpad listener + zoom mapper |
| `boards/shields/toucan/toucan_right.overlay` | Right-half pin assignments, Azoteq TPS43 I2C device definition |
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

## Trackpad (Azoteq TPS43, Toucan2 hardware) architecture

The trackpad lives physically on the right half but is used by the left (central) half via ZMK's input-split mechanism. This is the Toucan2 hardware revision — the original Toucan used a Cirque Pinnacle trackpad over SPI; Toucan2 replaces it with an Azoteq TPS43 over I2C (see `zmk_driver_azoteq`, pulled in via `config/west.yml`).

1. **Right half** (`toucan_right.overlay`): defines `tps43_trackpad` on `&i2c0` (`azoteq,tps43`, addr `0x74`), with gesture support (`zoom`, `scroll`, `two-finger-tap`, `single-tap`, `press-and-hold`, `three-finger-swipe`), `sensitivity = <100>`, `switch-xy`, `invert-scroll-y`.
2. **`toucan.dtsi`**: defines `trackpad_split` (the split proxy), `trackpad_listener` (the input pipeline), plus two input-processor nodes: `zip_zoom_mapper` (pinch gesture → ⌘−/⌘+) and `swipe_button_mapper` (4-direction swipe → ⌃⌘-arrow / Mission Control style shortcuts). The listener is **disabled** by default; the left overlay enables it.
3. **Left half** (`toucan_left.overlay`): sets `&trackpad_listener { status = "okay"; }` and `&zip_zoom_mapper { status = "okay"; }`.

Both `zip_zoom_mapper` and `swipe_button_mapper` bind Mac shortcuts (`LG(...)`/`LC(LG(...))`) by default; there's a `TOUCAN_WIN_MODE` `#define` at the top of `toucan.dtsi` (commented out) to switch them to Windows bindings instead.

Upstream `zmk-keyboard-toucan2` also ships an `is_touching_processor` that maps `&mo 4` while the trackpad is touched — deliberately **not** wired in here, since layer 4 is this keymap's ADJ layer (Notion macros), not a trackpad-specific layer, and enabling it would pop up ADJ any time a finger rests on the pad.

### Tuning pointer and scroll speed

All tuning is in `boards/shields/toucan/toucan.dtsi` inside `trackpad_listener`:

```c
trackpad_listener: trackpad_listener {
    input-processors = <&zip_xy_scaler 100 100>, <&zip_scroll_scaler 1 20>, <&zip_zoom_mapper>, <&swipe_button_mapper>;
    scroller {
        layers = <2 3>;                             // NAV, SYM — scroll active while either is held
        input-processors = <
            &zip_xy_to_scroll_mapper
            &zip_scroll_scaler 1 20                 // scroll speed: 1/20 (lower = slower)
            &zip_scroll_transform INPUT_TRANSFORM_X_INVERT
        >;
    };
};
```

- **Pointer speed**: change `zip_xy_scaler <num> <den>` — larger ratio = faster cursor. (Values above are upstream's TPS43 defaults, not yet retuned from the old Pinnacle's `250 100`/`1 7` — the two sensors have different native sensitivity, so treat this as a starting point.)
- **Scroll speed**: change `zip_scroll_scaler <num> <den>` in both places (main list and `scroller`) — e.g. `1 10` is faster than `1 20`.
- **Hardware sensitivity**: set in `toucan_right.overlay` on the `tps43_trackpad` node — `sensitivity`/`scroll-sensitivity` (0–100), `scroll-angle`, `filter-settings`, `hold-time`.
- **Scroll-active layers**: `scroller { layers = <...>; }` takes numeric layer indices, not the `NAV`/`SYM` `#define` names — those are only in scope inside `config/toucan.keymap`, not `toucan.dtsi`. Cross-check against the layer table below if layers are renumbered again.

## Display (nice_view_gem)

Custom shield in `boards/shields/nice_view_gem/`. Widgets: battery, layer name, output/BLE profile, sleep indicator. Font assets (QuinqueFive) are pre-compiled C files. Modify `custom_status_screen.c` and the `widgets/` files to change what the display shows.

## ZMK Studio

Enabled on the left half only (see `build.yaml` snippet `studio-rpc-usb-uart` and `cmake-args: -DCONFIG_ZMK_STUDIO=y`). Connect the left half via USB to use ZMK Studio for live keymap editing without reflashing.
