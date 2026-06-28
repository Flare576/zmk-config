# Sofle v2.0w — ZMK Config

ZMK firmware for the [Sofle v2.0w](https://github.com/GarrettFaucher/SofleKeyboard) — a wireless MX split 60% keyboard built on [nice!nano v2](https://nicekeyboards.com/nice-nano/) + [nice!view](https://nicekeyboards.com/nice-view/).

**Hardware:** nice!nano v2 · nice!view · Sofle v2.0w PCB · EC11 rotary encoders

---

## Build Targets

| Artifact | Keymap | Description |
|---|---|---|
| `sofle_left` / `sofle_right` | `sofle.keymap` | Josef Adamcik's original layout, ZMK port with OS-split design |
| `soflare_left` / `soflare_right` | `soflare.keymap` | Personal layout: mouse layer, FF14 game layer, ESDF layer, tap-dance |
| `sofle_test_left` / `sofle_test_right` | `sofle_test.keymap` | Hardware validation — each key outputs its row/position for diagnostics |
| `settings_reset` | — | Clears BT bonds on right half; flash when pairing goes wrong |

Builds run via GitHub Actions on push. Firmware artifacts appear in the workflow run.

---

## `sofle_test.keymap` - Testing Firmware

Simply maps a distinct keystroke to every key/encoder action for soldering validation.

## `soflare.keymap` - Personal Mapping

See the full explanation at my [Sofle](https://github.com/Flare576/sofle) top-level repo.

## `sofle.keymap` — Design Reference

This is a full ZMK port of [josefadamcik's QMK default keymap](https://github.com/qmk/qmk_firmware/tree/master/keyboards/sofle/keymaps/default), redesigned to solve problems the direct translation couldn't.

### The problem with a naive QMK → ZMK translation

The QMK default keymap handles Mac/Windows differences through a runtime toggle (`CG_TOGG`) and OS-adaptive custom keycodes (`KC_PRVWD`, `KC_NXTWD`, `KC_LSTRT`, `KC_LEND`). ZMK has no equivalent — it sends fixed keycodes at flash time, not runtime.

A straight port to ZMK produces a keyboard that:
- Has word-navigation hardcoded to `Alt+arrow` — correct for Mac, **broken on Windows/Linux** (should be `Ctrl+arrow`)
- Has line-boundary navigation hardcoded to `Home`/`End` — correct for Windows, **broken on Mac** (should be `Cmd+arrow`)
- Has clipboard shortcuts hardcoded to `Ctrl+Z/X/C/V` — correct for Windows, **broken on Mac** (should be `Cmd+Z/X/C/V`)
- Has no Bluetooth controls at all — unusable as a wireless keyboard

### The solution: OS-split layers

Instead of a toggle, the OS is a layer. Each combination of layout (QWERTY/Colemak) and OS (Mac/Windows) is a distinct base layer. A two-parameter macro switches the BT profile and base layer atomically — one keypress connects to your device *and* activates the right key behavior.

```
sm <BT profile> <layer>
```

### Layer map

| # | Name | Description |
|---|---|---|
| 0 | `M_QWERTY` | Mac + QWERTY — **default on flash** |
| 1 | `M_COLEMAK` | Mac + Colemak |
| 2 | `W_QWERTY` | Windows/Linux + QWERTY |
| 3 | `W_COLEMAK` | Windows/Linux + Colemak |
| 4 | `LOWER` | Symbols, F-keys — shared across all OS/layout combos |
| 5 | `RAISE_MAC` | Mac navigation + clipboard |
| 6 | `RAISE_WIN` | Windows/Linux navigation + clipboard |
| 7 | `ADJUST` | BT profile × OS × layout selection; media keys |

**ADJUST** activates when you hold both `LOWER` and `RAISE` simultaneously (either RAISE variant triggers it).

---

### Thumb cluster

The thumb cluster order differs between Mac and Windows because the primary modifier differs between operating systems.

| Mode | Outer → Inner (left thumb) | Rationale |
|---|---|---|
| Mac (`M_*`) | `LCTRL · LALT · LGUI` | Cmd (GUI) is inner — mirrors Mac keyboard ergonomics; matches QMK's CG_TOGG-on result |
| Windows (`W_*`) | `LGUI · LALT · LCTRL` | Ctrl is inner — identical to the QMK default layout |

Windows users switching from QMK: your thumb positions are unchanged.

Mac users switching from QMK who used `CG_TOGG`: the functional behavior (Cmd at inner thumb) is preserved — it's now baked into the layer instead of toggled.

---

### LOWER layer

Identical to the QMK default. No changes.

```
┌───────┬────┬────┬────┬────┬────┐                 ┌────┬────┬────┬────┬────┬─────┐
│ (trn) │ F1 │ F2 │ F3 │ F4 │ F5 │                 │ F6 │ F7 │ F8 │ F9 │F10 │ F11 │
├───────┼────┼────┼────┼────┼────┤                 ├────┼────┼────┼────┼────┼─────┤
│   `   │  1 │  2 │  3 │  4 │  5 │                 │  6 │  7 │  8 │  9 │  0 │ F12 │
├───────┼────┼────┼────┼────┼────┤                 ├────┼────┼────┼────┼────┼─────┤
│ (trn) │  ! │  @ │  # │  $ │  % │                 │  ^ │  & │  * │  ( │  ) │  |  │
├───────┼────┼────┼────┼────┼────┼────┐   ┌────┬───┴┬───┼────┼────┼────┼────┼─────┤
│ (trn) │  = │  - │  + │  { │  } │(trn│   │trn)│  [ │ ] │  ; │  : │  \ │(trn)    │
└───────┴────┴────┼────┼────┼────┼────┘   └────┴────┴───┼────┼────┴────┴────┴─────┘
                  │(trn│(trn│(trn│(trn│   │(trn│(trn│trn│(trn│
                  └────┴────┴────┴────┘   └────┴────┴───┴────┘
```

---

### RAISE layers — what changed and why

The physical key positions are identical between `RAISE_MAC` and `RAISE_WIN`. Only the keycodes sent differ.

| Physical key | QMK behavior | `RAISE_WIN` | `RAISE_MAC` |
|---|---|---|---|
| Word back | `Ctrl+←` or `Opt+←` (OS-adaptive) | `Ctrl+←` | `Opt+←` |
| Word forward | `Ctrl+→` or `Opt+→` (OS-adaptive) | `Ctrl+→` | `Opt+→` |
| Delete word | `Ctrl+Bspc` | `Ctrl+Bspc` | `Opt+Bspc` |
| Line start | `Home` or `Cmd+←` (OS-adaptive) | `Home` | `Cmd+←` |
| Line end | `End` or `Cmd+→` (OS-adaptive) | `End` | `Cmd+→` |
| Undo | `Ctrl+Z` only (Mac broken in QMK port) | `Ctrl+Z` | `Cmd+Z` |
| Cut | `Ctrl+X` only | `Ctrl+X` | `Cmd+X` |
| Copy | `Ctrl+C` only | `Ctrl+C` | `Cmd+C` |
| Paste | `Ctrl+V` only | `Ctrl+V` | `Cmd+V` |

The modifier cluster on the left side (A/S/D positions while RAISE is held) also differs:

| | `RAISE_WIN` | `RAISE_MAC` |
|---|---|---|
| Modifiers | `LALT · LCTRL · LSHFT` | `LALT · LGUI · LSHFT` |
| Example combo | `Shift+Ctrl+←` = select word | `Shift+Cmd+←` = select to line start |

---

### ADJUST layer — the BT profile × OS × layout grid

Hold `LOWER + RAISE` to reach ADJUST. The entire left side is a selection grid:

```
Columns = BT profile (0 through 4, one per paired device)
Rows    = OS + layout combination
```

```
╭─────────────┬──────┬──────┬──────┬──────┬──────╮
│             │  P0  │  P1  │  P2  │  P3  │  P4  │   Row = Win  QWERTY
├─────────────┼──────┼──────┼──────┼──────┼──────┤
│ [bootloader]│  P0  │  P1  │  P2  │  P3  │  P4  │   Row = Win  Colemak
├─────────────┼──────┼──────┼──────┼──────┼──────┤
│             │  P0  │  P1  │  P2  │  P3  │  P4  │   Row = Mac  QWERTY
├─────────────┼──────┼──────┼──────┼──────┼──────┤
│  [OUT_TOG]  │  P0  │  P1  │  P2  │  P3  │  P4  │   Row = Mac  Colemak
╰─────────────┴──────┴──────┴──────┴──────┴──────╯
                                             [BT_CLR] ← right encoder button
```

Press any cell to simultaneously select that BT profile and activate that OS+layout layer. Your typical workflow for each device you own:

- **Profile 0, Mac QWERTY row** → your Mac laptop
- **Profile 1, Win QWERTY row** → your Windows desktop
- **Profile 2, Mac QWERTY row** → your iPad or second Mac
- etc.

**Other ADJUST keys:**

| Key | Location | Action |
|---|---|---|
| `bootloader` | Row 2 (Win Colemak), far-left outer key | Flash mode for left half |
| `OUT_TOG` | Row 4 (Mac Colemak), far-left outer key | Toggle USB ↔ Bluetooth output |
| `BT_CLR` | Right encoder button (while in ADJUST) | Clear current BT profile pairing |
| Vol Dn / Mute / Vol Up | Right side, row 3 | Media volume |
| Prev / Play-Pause / Next | Right side, row 4 | Media transport |

---

### Muscle memory: QMK default → this keymap

**Nothing changes for Windows users on W_QWERTY / W_COLEMAK.** Every key sends the same code at the same physical position as the QMK default. The RAISE layer is now *more correct* (clipboard shortcuts previously only worked on Windows).

**Mac users on M_QWERTY / M_COLEMAK:** The letter rows, number row, LOWER, and RAISE physical positions are all identical. The only change is thumb cluster order — Cmd (GUI) is now at the inner thumb position (closer to spacebar), which matches standard Mac keyboard ergonomics. If you were using QMK's `CG_TOGG` to get this behavior, it's now the default and you get Cmd-correct RAISE shortcuts automatically.

**ADJUST:** The layout-switching paradigm changed. QMK's two-key toggle (W=QWERTY, E=Colemak) is replaced by the 4×5 grid. It's more powerful but requires learning once. The bootloader and media keys are at the same physical positions as in the QMK default.

---

## Hardware config (`sofle.conf`)

| Setting | Value | Why |
|---|---|---|
| `ZMK_SLEEP` | enabled, 15 min timeout | Battery life |
| `BT_CTLR_TX_PWR_PLUS_8` | +8 dBm | Max BT range |
| `LV_Z_MEM_POOL_SIZE` | 4608 | nice!view LVGL requirement |
| `EC11` + global thread trigger | enabled | Rotary encoders |
| `ZMK_POINTING` | enabled | Mouse layer (soflare) |

---

## Flashing

1. Put the target half into bootloader mode (double-tap reset, or use the `bootloader` key from ADJUST)
2. A USB drive appears — drag the `.uf2` artifact onto it
3. Repeat for the other half

Flash left half first. The right half is peripheral-only and needs no keymap.

---

## References

- [Sofle v2.0w hardware](https://github.com/GarrettFaucher/SofleKeyboard) — Garrett Faucher's wireless variant
- [Original Sofle by Josef Adamcik](https://github.com/josefadamcik/SofleKeyboard)
- [ZMK documentation](https://zmk.dev/docs)
- [nice!nano](https://nicekeyboards.com/nice-nano/) · [nice!view](https://nicekeyboards.com/nice-view/)
