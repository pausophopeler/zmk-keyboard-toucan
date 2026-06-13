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
| BASE  | 0     | Default |
| NAV   | 1     | Hold left thumb `mo 1` |
| SYM   | 2     | Hold right thumb `mo 2` |
| ADJ   | 3     | NAV + SYM simultaneously (conditional layer) |
| G Layer | 4   | Hold `G` key (`lt 4 G` on home row) |

The `lp_cpy` behavior on `C` and `V` fires `LG(C)` / `LG(V)` on hold and the bare key on tap (500ms tapping-term, tap-preferred).

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
        layers = <1 2>;                             // scroll active on NAV and SYM layers
        input-processors = <
            &zip_xy_to_scroll_mapper
            &zip_scroll_scaler 1 5                  // scroll speed: 1/5 (lower = slower)
            &zip_scroll_transform INPUT_TRANSFORM_X_INVERT
        >;
    };
};
```

- **Pointer speed**: change `zip_xy_scaler <num> <den>` — larger ratio = faster cursor.
- **Scroll speed**: change `zip_scroll_scaler <num> <den>` — e.g. `1 3` is faster than `1 5`.
- **Hardware sensitivity**: set in `toucan_right.overlay` on the `glidepoint` node — `sensitivity = "1x"`, `"2x"`, or `"4x"`.

## Display (nice_view_gem)

Custom shield in `boards/shields/nice_view_gem/`. Widgets: battery, layer name, output/BLE profile, sleep indicator. Font assets (QuinqueFive) are pre-compiled C files. Modify `custom_status_screen.c` and the `widgets/` files to change what the display shows.

## ZMK Studio

Enabled on the left half only (see `build.yaml` snippet `studio-rpc-usb-uart` and `cmake-args: -DCONFIG_ZMK_STUDIO=y`). Connect the left half via USB to use ZMK Studio for live keymap editing without reflashing.
