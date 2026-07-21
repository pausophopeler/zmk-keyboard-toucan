# Toucan Keymap — Custom Key Reference

Quick reference for every non-standard key/behavior on the Toucan keymap (`config/toucan.keymap`). Plain letters, numbers, and unmodified punctuation are omitted — this covers only holds, taps, macros, and modifier chords.

## Custom Behaviors (used across layers)

| Behavior | What it does |
|---|---|
| HRM (Home Row Mods) | Tap = letter, Hold (250ms, balanced) = modifier |
| lp_cpy (Long-Press Copy/Paste) | Tap = letter, Hold (500ms) = ⌘+letter |
| hyper_meh (mod-morph) | Tap alone = Hyper (⌘⌥⌃⇧). Tap+Shift = Meh (⌥⌃⇧, no ⌘) |
| td_slash_caps (tap-dance, 200ms) | 1 tap = `/`. 2 taps = Caps Lock |

## BASE Layer

| Key | Binding | What it does |
|---|---|---|
| A / S / D / F | HRM | Tap=letter, Hold=Ctrl / Alt / Gui / Shift |
| J / K / L / `;` | HRM | Tap=letter, Hold=Shift / Gui / Alt / Ctrl |
| G | `lt G_LAYER G` | Tap=G, Hold=activates G Layer |
| Z | `lt NUM Z` | Tap=Z, Hold=activates NUM layer |
| C | lp_cpy | Tap=C, Hold=⌘C (copy) |
| V | lp_cpy | Tap=V, Hold=⌘V (paste) |
| Bottom-left pinky | hyper_meh | Tap=Hyper, Shift+Tap=Meh |
| `/` key | td_slash_caps | 1 tap=`/`, 2 taps=Caps Lock |
| Right thumb inner | `sk RSHFT` | Sticky Shift (one-shot) |
| Left thumb 2 | `mo NAV` | Hold for NAV layer |
| Right thumb 2 | `mo SYM` | Hold for SYM layer |
| Right thumb 3 | `mo ADJ` | Hold for ADJ layer |

## GRAPHITE Layer

Full alternate letter layout for typing practice, toggled on/off via combo. Same custom behaviors as BASE, different letters:

| Key | Binding | What it does |
|---|---|---|
| N / R / T / S | HRM | Ctrl / Alt / Gui / Shift |
| H / A / E / I | HRM | Shift / Gui / Alt / Ctrl |
| G | `lt G_LAYER G` | Hold = G Layer |
| Q | `lt NUM Q` | Hold = NUM layer |
| C / V | lp_cpy | Hold = ⌘C / ⌘V |
| Bottom-left pinky | hyper_meh | Tap=Hyper, Shift+Tap=Meh |
| Bottom-right pinky | td_slash_caps | 1 tap=`/`, 2 taps=Caps Lock |

## NAV Layer (hold left thumb)

| Key | Binding | What it does |
|---|---|---|
| Left/Down/Up/Right | Arrows | Standard arrow keys |
| Z, X, C, V | ⌘Z / ⌘X / ⌘C / ⌘V | Undo / Cut / Copy / Paste |
| B | ⌘← | Jump to line start (Mac) |
| N (right side) | ⌘→ | Jump to line end |
| M | ⌥← | Move one word left |
| `,` | ⌥→ | Move one word right |
| H | ⌃⌘N | Custom/app-specific shortcut |
| Right thumb 3 | Right Alt | AltGr / accented chars |

## SYM Layer (hold right thumb)

| Key | Binding | What it does |
|---|---|---|
| Top row | `! @ # $ % ^ & * ( )` | Shifted number-row symbols |
| Home row | `- = [ ] \ ` \` `` | Punctuation symbols |
| B (row 3) | ⌥→ | Move word right |
| V (row 3) | ⌃⌥⌘M | Custom/app-specific shortcut |
| N, M, `,` `.` `/` | `+ _ { } |` `~` | Shifted punctuation |

## ADJ Layer (right thumb 3)

### Notion macros

| Key | Macro | Result |
|---|---|---|
| Q | notion_callout | Opens `/callout` block |
| W | notion_italic_blue | `/blue` → Enter → ⌘A → ⌘I |
| E | notion_toggle | `> ` → creates toggle list block |
| R | notion_reapply_color | ⌘A → ⌘⇧H (re-apply last color) |
| T | notion_bold_red | `/red` → Enter → ⌘A → ⌘B |
| A | notion_bold_green | `/green` → Enter → ⌘A → ⌘B |
| S | notion_italic_orange | `/orange` → Enter → ⌘A → ⌘I |
| D | notion_highlight_yellow | Types `/yellow background` → Enter |
| F | notion_todo | `[] ` → creates to-do checkbox |
| G | notion_heading | `## ` → creates Heading 2 block |

### System/OS shortcuts

| Key | Binding | What it does |
|---|---|---|
| Tab (top-left) | ⌘⌥Esc | Force Quit Applications |
| Y | ⌃⌘Q | Lock screen |
| U | Voice Command | Dictation/voice control |
| I | ⌘⌥⌃⇧K (Hyper+K) | Custom shortcut (likely Keyboard Maestro) |
| O | ⌥Space | Alt app-launcher (Spotlight/Raycast/Alfred) |
| Vol Down/Mute/Vol Up/Bri Down/Bri Up | Media keys | Standard media controls |
| X | ⌃⇧Tab | Previous browser tab |
| C | ⌘⇧] | Next browser tab |
| V | ⌘\` | Cycle windows within current app |
| B | Right-Option+\` | App-specific / AltGr use |
| BT row | `bt BT_CLR` / `bt BT_CLR_ALL` / `bt BT_SEL 0-4` | Clear one/all BT profiles; select profile 0–4 |

## G Layer — "F-Nav" (hold G key)

Compressed spreadsheet/document navigation layer:

| Key cluster | Binding | What it does |
|---|---|---|
| Top-left | ⇧⌘← / ↑ / ↓ / ⇧⌘→ | Extend selection to line start/end, arrow up/down |
| Top-right | ⇧⌥← / ⇧⌥→ | Extend selection by word |
| Top-right (I, O) | ⌘⇧V / ⌘⌥⇧V | Paste & match style / paste without formatting (Mac) |
| Mid-left | Shift+Space / ⌃Num− / ⌃Num+ / Insert | Select row / delete row / insert row (Excel/Sheets) |
| Mid-right | ⌥← / ⌥→ | Move by word |
| Mid-right (K, L) | ⌃⇧V / ⌃⌥V | Paste plain / paste special (Windows) |
| Low-right | ⌘← / ⌘→ | Jump to line start/end |

## NUM Layer (hold Z)

Standard numpad layout (7/8/9, 4/5/6, 0/1/2/3, `.`, `+`, `-`, Page Up/Down) — no non-standard bindings.

## Orange Layer (combo: two keys on row 2)

`F1`–`F12` function keys only — no non-standard bindings.

## Combos (not tied to a single layer)

| Keys | Result |
|---|---|
| D + F (home row) | Esc |
| J + K (home row) | Esc |
| Two rightmost row-3 keys (BASE/GRAPHITE) | Momentary Orange (F-key) layer |
| N + M (row 3, BASE/GRAPHITE) | Toggle GRAPHITE layer |
