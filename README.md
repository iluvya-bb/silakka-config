# silakka — ZMK config

Personal [ZMK](https://zmk.dev) firmware for a wireless 54-key split keyboard.
Dvorak base layout with positional ("timeless") home-row mods, five layers, an
animated nice!view UI, and a QWERTY fallback for gaming.

> The keyboard advertises over Bluetooth as **"Eladio"**.

## Contents

- [Hardware](#hardware) · [Layers](#layers) · [Keymaps](#keymaps)
- [How it works](#how-it-works): [home-row mods](#home-row-mods) ·
  [thumb keys](#thumb-keys) · [combos](#combos) · [key positions](#key-positions)
- [Host integration (Hyprland)](#host-integration-hyprland) — which desktop
  shortcuts this board can and can't send, and why — plus tmux
- [Firmware config](#firmware-config) · [Building & flashing](#building--flashing)
- [Tuning & troubleshooting](#tuning--troubleshooting)

---

## Hardware

| Part | Detail |
|------|--------|
| Controllers | 2× **nice!nano v2** (nRF52840, USB-C) |
| Displays | 2× **nice!view** (one per half) |
| Connection | Bluetooth LE (split link + host link) |
| Keys | 54 (6×4 matrix + 3 thumbs, per half) |

**Board target.** `nice_nano@2//zmk` — the post–Zephyr-4.1 way of selecting
nice!nano **v2**. (v1 would be `nice_nano@1//zmk`; the old `nice_nano_v2` name no
longer exists. See the [Zephyr 4.1 blog post](https://zmk.dev/blog/2025/12/09/zephyr-4-1#zmk-board-variant).)

**Shield.** The build uses ZMK's stock `lily58_left` / `lily58_right` shields.
A Lily58 is a 58-key board: the same 6×4 matrix, plus one extra inner key on the
bottom row and a 4th (innermost) thumb key per half. Those four positions don't
exist here, so the keymap binds them to `&none` on **every** layer. They show up
as `▪` in the diagrams below. Everything else maps 1:1.

Files:

```
config/lily58.keymap   layers, combos, hold-tap behaviors
config/lily58.conf     Kconfig — BLE, display, power, HID
config/west.yml        ZMK + nice-view-gem module sources
build.yaml             what GitHub Actions builds
```

---

## Layers

| # | Define | On screen | Purpose | How to reach it |
|---|--------|-----------|---------|-----------------|
| 0 | `DVO` | `DVO` | Base layer, home-row mods, number row | default |
| 1 | `QWE` | `QWE` | Gaming / fallback, **no** home-row mods | combo: two inner top-row keys (toggle) |
| 2 | `NAV` | `NAV` | F-keys, arrows, paging, tab-switching, undo/cut/copy/paste, window moves | hold left inner thumb (`Tab`) |
| 3 | `SYM` | `SYM` | Shifted-number row, brackets, quotes, operators | hold right inner thumb (`Bspc`) |
| 4 | `BTL` | **`SYS`** | Bluetooth profiles, bootloader, reset, power, media | combo: two outer thumb keys (momentary) |

*On screen* is the layer's `display-name`, which is what the nice!view shows.
Note layer 4 is the one place these differ — the keymap calls it `BTL`, the
display says `SYS`.

> `sym_layer` also carries a stray `label = "SYM"`. That's the pre-`display-name`
> way of naming a layer and is inert in current ZMK — no other layer has one.
> Harmless, but it's dead weight if you're tidying up.

`QWE` is a **toggle** (`&tog`) — it stays on until you fire the combo again.
`NAV`, `SYM` and `BTL` are all **momentary** (`&mo`) — active only while held.

Legend for the diagrams: `▽` = transparent (falls through to the layer below),
`▪` = physical key not present on this board. Cells otherwise show the character
the key sends, except where a glyph would be unreadable inside the ASCII grid —
`PIPE` on the SYM layer is the literal `|` key.

---

## Keymaps

### 0 · DVO — Dvorak base

```
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|   `   |   1   |   2   |   3   |   4   |   5   |                     |   6   |   7   |   8   |   9   |   0   |  DEL  |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|   =   |   P   |   Y   |   F   |   G   |   ,   |                     |   .   |   C   |   R   |   L   |   ;   |   -   |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|   \   | A GUI | O ALT | E CTL | U SFT |   I   |                     |   D   | H SFT | T CTL | N ALT | S GUI |   /   |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+
|   [   |   Q   |   J   |   K   |   X   |   '   |   ▪   |     |   ▪   |   B   |   M   |   W   |   V   |   Z   |   ]   |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+

                +---------+---------+---------+---------+     +---------+---------+---------+---------+
                | Alt/Esc | NAV/Tab | Sft/Spc |    ▪    |     |    ▪    | Ctl/Ent | SYM/Bsp |  RAlt   |
                +---------+---------+---------+---------+     +---------+---------+---------+---------+
```

Home-row cells read `letter` + the modifier it produces **on hold**.
Thumb cells read `hold/tap`.

This is Dvorak with the punctuation shifted to make room for a programmer's outer
column: the standard `' , .` cluster moves inward, `'` lands on the bottom row and
`;` on the top row, freeing the outer columns for `= \ [` and `- / ]`.

### 1 · QWE — QWERTY (gaming)

```
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|   `   |   1   |   2   |   3   |   4   |   5   |                     |   6   |   7   |   8   |   9   |   0   |  DEL  |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|  Tab  |   Q   |   W   |   E   |   R   |   T   |                     |   Y   |   U   |   I   |   O   |   P   |   -   |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
| Ctrl  |   A   |   S   |   D   |   F   |   G   |                     |   H   |   J   |   K   |   L   |   ;   |   /   |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+
|   [   |   Z   |   X   |   C   |   V   |   B   |   ▪   |     |   ▪   |   N   |   M   |   ,   |   .   |   '   |   ]   |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+

                +---------+---------+---------+---------+     +---------+---------+---------+---------+
                |  LAlt   |  LShft  |  Space  |    ▪    |     |    ▪    |  Enter  |  Bspc   |  RAlt   |
                +---------+---------+---------+---------+     +---------+---------+---------+---------+
```

Every key here is a plain `&kp` — **no hold-taps at all**. That's deliberate:
hold-taps add latency and misfire under the fast, sustained, same-hand key-holding
that games demand. `Ctrl` moves to the home-row outer column and `Shift` sits on a
thumb, both as normal modifiers.

Note that `NAV` and `SYM` are **not reachable** from this layer — their thumb keys
are plain `Space`/`Bspc` here. Toggle back to `DVO` (same combo) to get them.

### 2 · NAV — navigation & F-keys

```
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|  F1   |  F2   |  F3   |  F4   |  F5   |  F6   |                     |  F7   |  F8   |  F9   |  F10  |  F11  |  F12  |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|   ▽   |NewTab |PrvTab |NxtTab |ClsTab |   ▽   |                     | Home  | PgDn  | PgUp  |  End  |Reopen |   ▽   |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
| Shift |  GUI  |  Alt  | Ctrl  | Shift |   ▽   |                     | Left  | Down  |  Up   | Right |   :   |UrlBar |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+
|   ▽   | Undo  |  Cut  | Copy  | Paste |   ▽   |   ▪   |     |   ▪   |Swap L |Swap D |Swap U |Swap R |   ?   |   ▽   |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+

                +---------+---------+---------+---------+     +---------+---------+---------+---------+
                |    ▽    |    ▽    |    ▽    |    ▪    |     |    ▪    |    ▽    |    ▽    |    ▽    |
                +---------+---------+---------+---------+     +---------+---------+---------+---------+
```

The layout is hand-split by role: **right hand navigates, left hand modifies**.

- Right home row is vim-style `←↓↑→`, with `Home`/`PgDn`/`PgUp`/`End` above it.
- Left home row becomes plain, *unconditional* modifiers (`GUI Alt Ctrl Shift`
  on `A O E U`, in the same order as the base layer) so you can `Shift`+arrow to
  select or `Ctrl`+arrow to jump words without the positional rules of the base
  layer getting in the way. `Shift` is **also** on the outer column, which is
  where it originally lived on this layer.

  > Every one of these must be an explicit `&kp`, never `&trans`. A transparent
  > position falls through to the base layer's `&hml`, and `require-prior-idle-ms`
  > will then force it to a tap mid-chord — that's what broke `Alt+Shift+Right`
  > until `U` was bound explicitly.
- Left bottom row is the clipboard cluster: `Ctrl+Z / X / C / V`.
- **Row 1 is the browser/tab cluster**, built around the two tab-cycle keys:

  | Key | Binding | Sends | Effect (Firefox/Zen) |
  |-----|---------|-------|----------------------|
  | `NewTab` | `&kp LC(T)` | `Ctrl+T` | New tab |
  | `PrvTab` | `&kp LC(LS(TAB))` | `Ctrl+Shift+Tab` | Previous tab |
  | `NxtTab` | `&kp LC(TAB)` | `Ctrl+Tab` | Next tab |
  | `ClsTab` | `&kp LC(W)` | `Ctrl+W` | Close tab |
  | `Reopen` | `&kp LC(LS(T))` | `Ctrl+Shift+T` | Reopen last closed tab |
  | `UrlBar` | `&kp LC(L)` | `Ctrl+L` | Focus the address bar |

  Cycle in the middle, create and destroy on either side of it. These are plain
  keycodes, so they fire in **any** application — `Ctrl+W` still closes a tab in
  the browser but deletes a word in a shell, and `Ctrl+L` clears a terminal. That
  is the intended behaviour, but worth knowing before you reach for them blind.

  Free NAV positions if you want more: 12, 17, 23, 29, 36, 41, 49. Move-tab-left
  and move-tab-right (`&kp LC(LS(PG_UP))` / `LC(LS(PG_DN))`) would sit naturally
  at 12 and 17, bracketing the cluster.
- **Right bottom row is window management**, sitting directly under the arrows —
  same finger, one row down, moves the *window* instead of the cursor. Each key
  emits the whole chord by itself:

  | Key | Binding | Sends | Hyprland |
  |-----|---------|-------|----------|
  | `Swap L` | `&kp LA(LS(LEFT))` | `Alt+Shift+←` | `swapwindow, l` |
  | `Swap D` | `&kp LA(LS(DOWN))` | `Alt+Shift+↓` | `swapwindow, d` |
  | `Swap U` | `&kp LA(LS(UP))` | `Alt+Shift+↑` | `swapwindow, u` |
  | `Swap R` | `&kp LA(LS(RIGHT))` | `Alt+Shift+→` | `swapwindow, r` |

  So moving a window is **NAV thumb + one finger**, instead of holding Alt and
  Shift and an arrow across two hands. Because the modifiers are baked into the
  keycode, nothing here depends on hold-tap timing.

### 3 · SYM — symbols

```
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|   ▽   |   ▽   |   ▽   |   ▽   |   ▽   |   ▽   |                     |   ▽   |   ▽   |   ▽   |   ▽   |   ▽   |   ▽   |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|   ▽   |   !   |   @   |   #   |   $   |   %   |                     |   ^   |   &   |   *   |   -   |   =   |   +   |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|   ▽   | PIPE  |   {   |   [   |   (   |   <   |                     |   >   |   )   |   ]   |   }   |   \   |   ▽   |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+
|   ▽   |   _   |   "   |   '   |   `   |   ~   |   ▪   |     |   ▪   |   ▽   |   :   |   ;   |   ?   |   /   |   ▽   |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+

                +---------+---------+---------+---------+     +---------+---------+---------+---------+
                |    ▽    |    ▽    |    ▽    |    ▪    |     |    ▪    |    ▽    |    ▽    |    ▽    |
                +---------+---------+---------+---------+     +---------+---------+---------+---------+
```

- Row 1 is the shifted number row in order (`! @ # $ %` → `^ & *`), with the
  right side trailing off into the math operators `- = +`.
- Row 2 is the bracket row, **opening on the left, closing on the right, mirrored**:
  `{ [ ( <` under the left fingers, `> ) ] }` under the right, so a pair is one
  alternating roll.
- Row 3 holds the quote/underscore family and the remaining punctuation.
- The number row is left transparent — digits still come from the base layer, so
  `SYM` + a number key types the digit, not a symbol.

**Character coverage.** Between DVO and SYM the board has a dedicated key for
**every printable ASCII character** — all 26 letters, all 10 digits and all 32
symbols. Verified by sweeping the keymap against the full printable set.

`~` was the one gap for a while: there was no `&kp TILDE` anywhere, and the
fallback of `Shift`+`` ` `` is awkward here because both backtick positions
(DVO@0, the top-left corner, and SYM@40) are on the left hand — so the left-hand
Shift won't fire, and you need the right-hand `H` or the thumb Shift plus a
stretch to the corner. A lot of hand for `~/`. It now lives at SYM position 41,
immediately right of the backtick, pairing them the way `-`/`_` and the brackets
already are.

A few characters are bound twice on purpose — `-` `=` `[` `]` `\` `;` `'` `/` are
on both DVO and SYM, so you don't have to leave the symbol layer mid-expression.
SYM still has free positions at 12, 24, 35, 36, 44 and 49.

### 4 · BTL — Bluetooth & system

```
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
| BOOT  |EP_TOG |   ▽   |   ▽   |   ▽   |   ▽   |                     |   ▽   |   ▽   |   ▽   |   ▽   |   ▽   |BT_CLR |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
| RESET | BT 0  | BT 1  | BT 2  | BT 3  | BT 4  |                     |   ▽   |   ▽   |   ▽   |   ▽   |   ▽   | RESET |
+-------+-------+-------+-------+-------+-------+                     +-------+-------+-------+-------+-------+-------+
|   ▽   |   ▽   |   ▽   |   ▽   |   ▽   |   ▽   |                     |   ▽   | Prev  | Vol-  | Vol+  | Next  |   ▽   |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+
|   ▽   |   ▽   |   ▽   |   ▽   |   ▽   |   ▽   |   ▪   |     |   ▪   |   ▽   |   ▽   | Mute  | Play  |   ▽   |   ▽   |
+-------+-------+-------+-------+-------+-------+-------+     +-------+-------+-------+-------+-------+-------+-------+

                +---------+---------+---------+---------+     +---------+---------+---------+---------+
                |    ▽    |    ▽    |    ▽    |    ▪    |     |    ▪    |    ▽    |    ▽    |    ▽    |
                +---------+---------+---------+---------+     +---------+---------+---------+---------+
```

| Key | Binding | Effect |
|-----|---------|--------|
| `BT 0`–`BT 4` | `&bt BT_SEL n` | Switch to Bluetooth profile *n* (5 hosts) |
| `BT_CLR` | `&bt BT_CLR` | Erase the bond on the **current** profile — do this before re-pairing a host |
| `RESET` | `&sys_reset` | Soft-reset. Left and right keys reset **their own half** |
| `BOOT` | `&bootloader` | Reboot the **left** half into the UF2 bootloader |
| `EP_TOG` | `&ext_power EP_TOG` | Cut/restore power to the display rail |
| Media | `&kp C_*` | Prev / Vol− / Vol+ / Next, and Mute / Play-Pause below |

Because this layer is reached by *holding* a combo, you keep both outer thumbs
down while pressing the target key.

Only the left half has a `&bootloader` key; put the right half into the bootloader
by double-tapping its physical reset button.

---

## How it works

### Home-row mods

The base layer uses **positional (a.k.a. "timeless") home-row mods**, in the style
of [urob's config](https://github.com/urob/zmk-config). Reading pinky → index,
both hands are **GUI · Alt · Ctrl · Shift**, mirror-symmetric:

```
Left:   A = GUI    O = Alt    E = Ctrl    U = Shift
Right:  S = GUI    N = Alt    T = Ctrl    H = Shift
```

Two hold-tap behaviors implement this — `hml` for the left hand, `hmr` for the
right — with identical timing and mirrored trigger sets:

| Property | Value | What it does |
|----------|-------|--------------|
| `flavor` | `balanced` | Hold resolves early if the *next* key is pressed **and released** while this one is held |
| `tapping-term-ms` | `280` | Hold this long with nothing else pressed → modifier |
| `quick-tap-ms` | `175` | Tap then immediately re-press → repeats the **letter**, so `eee` works |
| `require-prior-idle-ms` | `150` | If you typed anything in the last 150 ms, it can *only* be a tap. Kills mid-word misfires |
| `hold-trigger-key-positions` | opposite hand + thumbs | The positional rule (below) |
| `hold-trigger-on-release` | *(set)* | Lets you stack mods, e.g. hold `E`+`U` then hit a right-hand key for Ctrl+Shift |

**The positional rule.** `hml` lists every right-hand key plus all thumbs in
`hold-trigger-key-positions`; `hmr` lists every left-hand key plus the thumbs.
A modifier therefore only engages when the key that follows it is on the *other*
hand. Same-hand rolls — `oe`, `ue`, `ao` — physically cannot produce a modifier,
no matter how slowly you type them. Cross-hand chords (`Ctrl+C`, `GUI+Enter`) stay
instant, since they're always opposite-hand by construction.

The key groups come from three `#define`s at the top of the keymap, so if you ever
change the physical layout, those are what you update.

> **The tradeoff.** This is exactly what makes typing reliable, and exactly what
> makes some *application* shortcuts unreachable — `Alt+Q`, `Alt+1`, `Alt+F` and
> friends put the modifier and the target on the same hand, which the rule
> forbids. Use the thumb or the mirror mod instead; see
> [Host integration](#the-opposite-hand-rule-bites-host-shortcuts) for the full
> list and the workarounds.

### Thumb keys

| Position | Binding | Tap | Hold |
|----------|---------|-----|------|
| L outer | `&mt LALT ESC` | `Esc` | `Alt` |
| L middle | `&lt_th NAV TAB` | `Tab` | `NAV` layer |
| L inner | `&mt LSHFT SPACE` | `Space` | `Shift` |
| R inner | `&mt LCTRL RET` | `Enter` | `Ctrl` |
| R middle | `&lt_th SYM BSPC` | `Bspc` | `SYM` layer |
| R outer | `&kp RALT` | — | `Alt` (plain) |

The two layer keys use a custom `lt_th` behavior tuned for thumbs:
`flavor = "hold-preferred"`, `tapping-term-ms = 200`, `quick-tap-ms = 200`,
`require-prior-idle-ms = 125`. Hold-preferred makes the layer engage the instant
another key goes down, which is what you want for a layer you're about to type
into; `require-prior-idle-ms` keeps it from firing mid-word, and `quick-tap-ms`
means tap-then-hold auto-repeats `Tab`/`Bspc`.

The four mod-taps use ZMK's **stock `&mt`**, which is `hold-preferred` at 200 ms
with *no* `quick-tap-ms` and *no* `require-prior-idle-ms`. See
[Tuning](#tuning--troubleshooting) for what that implies.

### Combos

| Node | Keys | Positions | Binding | Effect |
|------|------|-----------|---------|--------|
| `combo_toggle_qwerty` | The two inner keys of the top letter row (`,`+`.` on Dvorak, `T`+`Y` on QWERTY) | `17 18` | `&tog QWE` | Toggle the QWERTY layer on/off |
| `combo_bt_layer` | The two outermost thumb keys | `50 57` | `&mo BTL` | Hold for the System layer |

Both use `timeout-ms = 50`, so the two keys must go down within 50 ms of each
other — fast enough that ordinary typing won't trigger them. Neither combo is
restricted with a `layers` property, so both are live on **every** layer. That's
required for the QWERTY toggle (you need it to turn the layer back off), and
harmless in practice for the rest.

### Key positions

Position numbers run left-to-right, top-to-bottom across the whole board — the
order bindings appear in the keymap. They're what the combos and the
`hold-trigger-key-positions` defines refer to:

```
  0  1  2  3  4  5                    6  7  8  9 10 11
 12 13 14 15 16 17                   18 19 20 21 22 23
 24 25 26 27 28 29                   30 31 32 33 34 35
 36 37 38 39 40 41 [42]         [43] 44 45 46 47 48 49
        50 51 52 [53]           [54] 55 56 57
```

Bracketed positions are the four that don't exist on this board.

```c
#define KEYS_L  0 1 2 3 4 5  12 13 14 15 16 17  24 25 26 27 28 29  36 37 38 39 40 41
#define KEYS_R  6 7 8 9 10 11  18 19 20 21 22 23  30 31 32 33 34 35  44 45 46 47 48 49
#define THUMBS  50 51 52 53 54 55 56 57
```

---

## Host integration (Hyprland)

The Dvorak remap happens **in firmware**, so the host stays on a plain `us`
layout and every Hyprland bind refers to the letter this keymap *sends*, not the
letter printed on the keycap. `$mainMod = ALT` throughout.

> Config lives in `~/.config/hypr/` (an ML4W dotfiles checkout). Binds are in
> `conf/keybindings/default.conf`, which is **ML4W-managed and overwritten on
> update** — put personal binds in `conf/custom.conf`, which is sourced last.

Auditing all 103 bind lines against this keymap: 4 are mouse binds, 61 work as
typed, **28 can't use the left-hand modifiers**, and **10 reference a key no
layer produces**.

### The opposite-hand rule bites host shortcuts

This is the single biggest gotcha on this board, and it is a direct consequence
of [positional home-row mods](#home-row-mods). `hml` only fires when the *next*
key is on the right hand or a thumb. So any shortcut whose target key is **also
on the left hand** cannot be produced with the left-hand modifiers — the mods
resolve as taps and you get the letters typed into the window instead.

| Bind | Target key | Action |
|------|-----------|--------|
| `Alt+1` … `Alt+5` | `1`–`5` @ 1–5 | Switch to workspace 1–5 |
| `Alt+Shift+1` … `5` | ″ | Move window to workspace 1–5 |
| `Alt+Ctrl+1` … `5` | ″ | Move *all* windows to workspace 1–5 |
| `Alt+Q` · `Alt+Shift+Q` · `Alt+Ctrl+Q` | `Q` @ 37 | Kill window · quit app · wlogout |
| `Alt+E` · `Alt+Ctrl+E` | `E` @ 27 | File manager · emoji picker |
| `Alt+F` | `F` @ 15 | Fullscreen |
| `Alt+J` · `Alt+K` | @ 38, 39 | Move focus up · down |
| `Alt+G` · `Super+Alt+G` | `G` @ 16 | Toggle group · game mode |
| `Alt+Ctrl+K` | `K` @ 39 | Show keybindings |
| `Alt+Shift+A` · `Alt+Shift+I` | @ 25, 29 | Toggle animations · sleep mode |

Note the asymmetry in the workspace binds: `Alt+6` … `Alt+0` are fine, because
6–0 sit on the right hand. Only **1–5** are affected.

**Two workarounds, both already on the board — no firmware change needed:**

1. **Use the left outer thumb for Alt.** Position 50 is `&mt LALT ESC`, a plain
   `&mt` with no `hold-trigger-key-positions`, so the opposite-hand rule doesn't
   apply to it at all. Thumb + `1` just works. This single habit covers all 28.
2. **Use the right-hand mirror mods** — `N`=Alt, `H`=Shift, `T`=Ctrl, `S`=GUI.
   The target is left-hand, so `hmr`'s `KEYS_L` trigger set fires correctly.
   This is the better option for `Super+Alt+G`: hold `S`+`N`, tap `G`.

### Keys no layer produces

| Bind | Note |
|------|------|
| `Alt+PRINT` → screenshot | No Print Screen key in any layer. The `Alt+Shift+S` alias does the same thing and works. |
| `XF86MonBrightnessUp` / `Down` | Not in the keymap |
| `XF86AudioMicMute` | Not in the keymap |
| `XF86AudioPause` | No dedicated Pause; BTL's `C_PP` sends play-*pause*, which Hyprland sees as `XF86AudioPlay` |
| `XF86Calculator` · `XF86Lock` · `XF86Tools` | Not in the keymap |
| `code:238` / `code:237` (kbd backlight) | Not in the keymap |

Most of these are ML4W's laptop defaults and irrelevant on a desktop. The
remaining audio keys — volume, mute, next, prev, play — **do** work: they come
from the [BTL layer's](#4--btl--bluetooth--system) consumer codes, which the host
resolves to the matching `XF86Audio*` keysyms.

### Other gotchas

**`Alt+Tab` can't auto-repeat.** `binde = ALT,Tab,cyclenext` is meant to repeat
while held, but on DVO the only Tab is the *tap* of the NAV thumb — holding that
key gives you the NAV layer, not a held Tab. You can tap-cycle one window at a
time, but never hold to scroll through them. A dedicated `&kp TAB` on NAV or SYM
would restore it. (QWE is unaffected; it has a real Tab at position 12.)

**The second keyboard layout is unreachable.** `conf/keyboard.conf` sets
`kb_layout = us,mn` with `kb_options = grp:caps_toggle`, so switching to the
Mongolian layout requires Caps Lock — and there is no Caps Lock in any layer.
Either add `&kp CAPS` (the BTL layer has free positions) or change `kb_options`
to a chord this board can actually send.

### tmux

`~/.tmux.conf`, prefix `C-a`.

**Press the prefix with the right inner thumb, not the left home row.** `Ctrl` on
the left home row is `E`@27 and `A` is @25 — same hand, so the positional rule
refuses it and you get `ea` typed into the pane. Right-hand `Ctrl` (`T`@32) works
positionally, but `hmr` carries `require-prior-idle-ms = 150`, so firing the
prefix immediately after typing forces a tap and yields a literal `t`. The thumb's
`&mt` has neither an idle guard nor a positional rule, so it is the only source
that is reliable every time — which matters for the most-pressed chord you own.

**Pane navigation is `d h t n`, not `h j k l`.** Those are the physical keys where
QWERTY's `hjkl` sit, so the finger pattern is vim's even though the letters
differ. The literal letters scatter across both hands on Dvorak:

| vim key | Dvorak position | | Dvorak key | Position |
|---------|----------------|---|-----------|----------|
| `h` | @31 right home, index | | `d` | @30 |
| `j` | @38 **left bottom** | | `h` | @31 |
| `k` | @39 **left bottom** | | `t` | @32 |
| `l` | @21 right top, ring | | `n` | @33 |

Resizing follows the same keys shifted (`D H T N`). That also makes the modifier
consistent: all four are right-hand, so all four take the **left** Shift. Under
`H J K L`, `H`/`L` wanted left Shift while `J`/`K` wanted right Shift or the thumb.

> `d`, `t` and `n` shadow tmux defaults — `detach-client`, `clock-mode` and
> `next-window`. Detach and next-window are re-homed to `prefix C-d` and
> `prefix C-n`; `prefix p` is still `previous-window`. Note that `source-file`
> never *un*binds, so after editing pane binds a running server keeps the old
> ones until you unbind them by hand or restart the server.

**`C-b` is a second prefix for QWERTY.** Six of the eight nav letters are
unambiguous, but `h` is not — Dvorak needs it for *down*, QWERTY for *left* — so
the two sets cannot share one key table:

| Direction | Dvorak sends | QWERTY sends |
|-----------|--------------|--------------|
| left | `d` | `h` |
| down | `h` | `j` |
| up | `t` | `k` |
| right | `n` | `l` |

`C-a` therefore keeps the Dvorak set, and `C-b` switches into a separate `qwerty`
key table holding canonical `hjkl` (plus `HJKL` resize and the essentials — `c`,
splits, `d`, `n`, `p`, `z`, `r`). Use it when the board is on its QWERTY layer, or
on any normal keyboard.

Note that tmux's `prefix2` option cannot do this: a second prefix key still enters
the *same* `prefix` table, so it can't carry different bindings. A genuine second
prefix needs a root binding into a custom table:

```tmux
bind -n C-b switch-client -T qwerty
bind -T qwerty h select-pane -L
```

> The `-n` makes `C-b` global, so vim's page-up no longer reaches the pane.
> `C-b C-b` sends a literal `C-b` to get it back. Changing the prefix key is a
> one-line edit to the `bind -n` line.

**`bind |` needs the SYM layer** — `|` exists only at SYM@25, so a horizontal
split is prefix → SYM thumb + key. `\` is a plain base-layer key at DVO@24 and is
bound as an alias for the same split.

Window navigation is `S-Left`/`S-Right` with no prefix. Those work only because
NAV binds `Shift` explicitly; before that fix they failed the same way
`Alt+Shift+<arrow>` did.

### Window management

Moving a window is `Alt+Shift+<arrow>` → `swapwindow`. Pressed literally that's a
four-key chord across a layer, so the [NAV layer's bottom row](#2--nav--navigation--f-keys)
has dedicated keys that emit the whole chord as a single keycode
(`&kp LA(LS(RIGHT))` and friends), reducing it to NAV thumb + one finger.

> Those four keys **hardcode `$mainMod = ALT` into the firmware**. If you ever
> switch `$mainMod` to `SUPER` — as many Hyprland configs do — they silently stop
> matching, and the keymap has to change to `LG(LS(...))`. This coupling is not
> visible from the Hyprland side, so it's worth remembering.

---

## Firmware config

Annotated tour of `config/lily58.conf`:

**Identity & display**

- `CONFIG_ZMK_KEYBOARD_NAME="Eladio"` — the BLE advertised name.
- `CONFIG_ZMK_DISPLAY=y`, `..._BLANK_ON_IDLE=n` — screens stay lit until deep sleep.
- `..._WORK_QUEUE_DEDICATED=y` — the display gets its own thread, so redraws don't
  stutter while typing.
- `..._STATUS_SCREEN_CUSTOM=y` — required to hand the screen over to nice-view-gem.
- `CONFIG_SSD1306=n` — disables the old OLED driver so the nice!view SPI driver wins.
- `CONFIG_ZMK_EXT_POWER=y` — lets the board gate power to the display rail
  (and is what `&ext_power EP_TOG` drives).

**Bluetooth stability**

- `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y` — maximum radio TX power (+8 dBm).
- `CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y` — the newer connection/pairing path.
- `CONFIG_CLOCK_CONTROL_NRF_K32SRC_RC=y` — run the 32 kHz clock off the nRF52840's
  **internal RC oscillator** instead of an external crystal. This is the fix for
  random unprompted disconnects on nice!nano clones with a flaky or missing
  crystal. Costs a little idle current; drop it if your disconnects are gone.

**Battery**

- `CONFIG_ZMK_BATTERY_REPORTING=y` — master switch.
- `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y` — the central pulls the
  peripheral's level over the split link so both gauges draw on the nice!view.
- Do **not** enable `..._BATTERY_LEVEL_PROXY`. It republishes the peripheral as a
  *second* GATT Battery Service, which BlueZ rejects (*"More than one BATT service
  exists for this device"*); the HID profile then fails to attach and the host
  drops the connection right after pairing. This was found the hard way — see
  commit `84a292c`.

**Power & input**

- `CONFIG_ZMK_SLEEP=y` + `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000` — deep sleep
  after 30 minutes idle.
- `CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=1` / `..._RELEASE_MS=7` — eager debounce:
  presses register almost immediately, releases are damped to avoid chatter.
- `CONFIG_ZMK_HID_REPORT_TYPE_NKRO=y` — full n-key rollover.
- `CONFIG_ZMK_STUDIO=n` — ZMK Studio is off; the keymap here is the only source of
  truth, which avoids stale settings-partition state fighting the firmware.

### Display module

The nice!view runs [nice-view-gem](https://github.com/M165437/nice-view-gem) — an
animated status screen with battery, WPM and layer widgets, rendered entirely
on-keyboard. It's pulled in as a west module in `config/west.yml` and selected by
adding the `nice_view_gem` shield in `build.yaml`.

> nice-view-gem must track the same ZMK version as the build. This repo builds ZMK
> `main` (Zephyr 4.1 / LVGL 9), so the module is pinned to its `main` branch too.
> The bongo-cat module (`zmk-nice-oled`) is *not* LVGL-9 compatible and was
> replaced for exactly that reason (commit `3dd899c`).

---

## Building & flashing

Builds run automatically on every push via GitHub Actions
(`.github/workflows/build.yaml`, which just calls ZMK's reusable
`build-user-config.yml`). `build.yaml` declares three targets:

```yaml
- board: nice_nano@2//zmk
  shield: lily58_left nice_view_adapter nice_view_gem
- board: nice_nano@2//zmk
  shield: lily58_right nice_view_adapter nice_view_gem
- board: nice_nano@2//zmk
  shield: settings_reset
```

1. Push, wait for the **Build ZMK Firmware** workflow to finish.
2. Download the `firmware` artifact. It contains `lily58_left … .uf2`,
   `lily58_right … .uf2` and `settings_reset … .uf2`.
3. Double-tap the reset button on a half — it mounts as a USB drive — and drag
   the matching `.uf2` onto it.
4. **After major changes** (board target, BLE settings, new pairing): flash
   `settings_reset` to **both** halves first to clear stale bonds and split
   pairing, then flash left and right, then re-pair from the host.

Flashing order matters after a settings reset: bring up the left (central) half
first, let the halves re-link, then pair with the host.

---

## Tuning & troubleshooting

**Home-row mods feel slow / don't trigger.** Lower `tapping-term-ms` (280 → ~220)
on `hml`/`hmr`. If they trigger when you don't want them, raise
`require-prior-idle-ms` instead — that's the setting that suppresses mid-word
misfires. If you want off home-row mods entirely, the usual alternative is
Callum-style one-shot mods parked on a layer.

**Thumb mod-taps can misfire on a roll.** `&mt` is `hold-preferred`, so pressing
another key *while* a mod-tap thumb is still down resolves it to the modifier
immediately. Overlapping `Space` with the next letter therefore yields
`Shift`+letter, and typing quickly after `Enter` can yield `Ctrl`+letter. If you
hit this, define a thumb mod-tap alongside `lt_th` and swap the four `&mt` uses
to it:

```c
mt_th: mod_tap_thumb {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";     // only the timer promotes it to a hold
    tapping-term-ms = <200>;
    quick-tap-ms = <200>;         // tap-then-hold repeats Space/Enter
    require-prior-idle-ms = <125>;
    bindings = <&kp>, <&kp>;
};
```

**A desktop shortcut types letters into the window instead of firing.** The
modifier and the target key are on the same hand, so the positional rule refused
to promote the hold. Use the left outer thumb for Alt, or the mirror modifier on
the right hand. Full list in
[Host integration](#the-opposite-hand-rule-bites-host-shortcuts).

**A shortcut does nothing at all, silently.** Either the key doesn't exist in any
layer (`PRINT`, `CAPS`, the `XF86` laptop keys — see
[Keys no layer produces](#keys-no-layer-produces)), or the dispatcher is a no-op:
`swapwindow` in particular does nothing on a floating window, or when there's no
tiled window in that direction. `Alt+<arrow>` (`movefocus`) is the quick way to
tell the two apart — if focus moves, the chord is reaching Hyprland fine.

**A modifier on the NAV layer doesn't hold.** Every modifier on that layer must
be an explicit `&kp`. A `&trans` falls through to the base layer's `&hml`, whose
`require-prior-idle-ms` then forces a tap mid-chord.

**Random disconnects.** Keep `CONFIG_CLOCK_CONTROL_NRF_K32SRC_RC=y`. If they
persist, flash `settings_reset` to both halves and re-pair.

**Host won't accept the keyboard / no HID after pairing.** Almost always the
double Battery Service problem — verify `..._BATTERY_LEVEL_PROXY` is *not* set.

**Build suddenly breaks.** `config/west.yml` pins both `zmk` and `nice-view-gem` to
`main`, so an upstream change can break a build with no local change. Pin both to
a known-good SHA or tag if you want reproducible builds.

**Battery life.** Max TX power, the RC oscillator, and a display that never blanks
all cost current. In rough order of savings: set
`CONFIG_ZMK_DISPLAY_BLANK_ON_IDLE=y`, drop to a lower `CONFIG_BT_CTLR_TX_PWR_*`,
shorten `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT`.

---

## Credits

- [ZMK Firmware](https://zmk.dev)
- [nice-view-gem](https://github.com/M165437/nice-view-gem) by M165437
- Home-row mod approach inspired by [urob/zmk-config](https://github.com/urob/zmk-config)
