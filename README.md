# silakka — ZMK config

Personal [ZMK](https://zmk.dev) firmware for a wireless **Lily58** split keyboard.
Dvorak base layout with positional home-row mods, an animated nice!view UI, and a
QWERTY fallback for gaming.

> The keyboard advertises over Bluetooth as **"Eladio"**.

## Hardware

| Part | Detail |
|------|--------|
| Controllers | 2× **nice!nano v2** (nRF52840, USB-C) |
| Shield | Lily58 split (58 keys) |
| Displays | 2× **nice!view** (one per half) |
| Connection | Bluetooth LE (split + host) |

The board target is `nice_nano@2//zmk` — the post–Zephyr-4.1 way of selecting
nice!nano **v2**. (v1 would be `nice_nano@1//zmk`; the old `nice_nano_v2` name no
longer exists. See the [Zephyr 4.1 blog post](https://zmk.dev/blog/2025/12/09/zephyr-4-1#zmk-board-variant).)

## Layers

| # | Name | Purpose |
|---|------|---------|
| 0 | **DVO** | Dvorak base, with home-row mods and number row |
| 1 | **QWE** | QWERTY fallback (no home-row mods — avoids gaming misfires) |
| 2 | **NAV** | F-keys, vim arrows, Home/End/PgUp/PgDn, tab-switching, undo/cut/copy/paste |
| 3 | **SYM** | Symbols, brackets/parens, math operators |
| 4 | **SYS** | Bluetooth profiles, bootloader, reset, ext-power toggle, media keys |

**Reaching layers**

- **NAV** — hold the left inner thumb (`TAB`)
- **SYM** — hold the right inner thumb (`BSPC`)
- **SYS** — press the two outermost thumb keys together (combo)
- **QWE** — toggle by pressing the two inner top-row keys together (combo)

## Home-row mods

The base layer uses **"timeless" home-row mods** (positional hold-tap, à la
[urob's config](https://github.com/urob/zmk-config)). Reading pinky → index, both
hands are **GUI · Alt · Ctrl · Shift** (mirror-symmetric):

```
Left:   A=GUI   O=Alt   E=Ctrl   U=Shift
Right:  S=GUI   N=Alt   T=Ctrl   H=Shift
```

Two behaviors, `hml` (left) and `hmr` (right), each use
`hold-trigger-key-positions` so a modifier only engages when a key on the
**opposite hand** (or a thumb) is pressed. This means same-hand letter rolls can
never misfire a modifier, while normal cross-hand chords stay instant.

Tuning: `tapping-term-ms = 280`, `quick-tap-ms = 175`, `require-prior-idle-ms = 150`,
`flavor = "balanced"`, `hold-trigger-on-release`. If mods feel slow to engage,
lower the tapping term; if you dislike home-row mods entirely, the usual
alternative is Callum-style one-shot mods on a layer.

## Combos

| Keys | Action |
|------|--------|
| Two inner top-row keys (`,` + `.` on Dvorak) | Toggle the QWERTY layer |
| Two outermost thumb keys | Momentary SYS (Bluetooth/System) layer |

## Display

The nice!view runs the [nice-view-gem](https://github.com/M165437/nice-view-gem)
custom status screen (animated, battery/WPM/layer widgets). It's pulled in via
`config/west.yml` and selected with the `nice_view_gem` shield in `build.yaml`.
Everything renders on-keyboard — no companion app required.

> **Note:** nice-view-gem must track the same ZMK version as your build. This repo
> uses ZMK `main` (Zephyr 4.1 / LVGL 9), so the module is pinned to its `main`
> branch. The bongo-cat module (`zmk-nice-oled`) is *not* yet LVGL-9 compatible.

## Building & flashing

Builds run automatically on every push via GitHub Actions
(`.github/workflows/build.yaml`).

1. Push your changes and wait for the **Build ZMK Firmware** workflow to finish.
2. Download the `firmware` artifact from the run. It contains:
   - `lily58_left … .uf2`
   - `lily58_right … .uf2`
   - `settings_reset … .uf2`
3. Put a half into bootloader mode (double-tap reset) — it mounts as a USB drive —
   and drag the matching `.uf2` onto it.
4. **After major changes** (board target, BLE settings, new pairing), flash
   `settings_reset` to *both* halves first to clear stale Bluetooth bonds, then
   flash the left/right firmware, then re-pair from your host.

## Config notes

Key settings in `config/lily58.conf`:

- **NKRO** HID, **max BLE TX power**, dedicated display work queue
- **Connection stability:** `CONFIG_ZMK_BLE_EXPERIMENTAL_CONN` (disables 2M PHY) +
  internal RC oscillator (`CONFIG_CLOCK_CONTROL_NRF_K32SRC_RC`) to avoid random
  disconnects from a flaky 32 kHz crystal
- **Split battery reporting** for both halves
- 30-minute deep-sleep timeout; ZMK Studio disabled

## Layout reference

```
0: DVORAK
,-----------------------------------------.                ,-----------------------------------------.
|  `  |  1  |  2  |  3  |  4  |  5  |                       |  6  |  7  |  8  |  9  |  0  | DEL |
|  =  |  P  |  Y  |  F  |  G  |  ,  |                       |  .  |  C  |  R  |  L  |  ;  |  -  |
|  \  |  A  |  O  |  E  |  U  |  I  |                       |  D  |  H  |  T  |  N  |  S  |  /  |
|  [  |  Q  |  J  |  K  |  X  |  '  |  -  |             |  - |  B  |  M  |  W  |  V  |  Z  |  ]  |
            | Esc | Tab | Spc | -  |             | -  | Ent | Bsp | Alt|
            \ Alt | NAV | Sft |                       | Ctl | SYM |    /
'-----------------------------------------'                '-----------------------------------------'
( A/O/E/U and H/T/N/S are home-row mods; thumb keys are mod-tap / layer-tap )
```

## Credits

- [ZMK Firmware](https://zmk.dev)
- [nice-view-gem](https://github.com/M165437/nice-view-gem) by M165437
- Home-row mod approach inspired by [urob/zmk-config](https://github.com/urob/zmk-config)
