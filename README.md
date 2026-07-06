# kitty

## Overview

Configuration for the [kitty](https://sw.kovidgoyal.net/kitty/) GPU-accelerated terminal emulator — the primary terminal across the Imperator ricing. Provides the amber CRT ANSI palette, a frosted-glass background, and a slanted powerline tab bar, and doubles as the palette reference other terminal-adjacent tools (newsboat, calcurse-open.sh) map their 16-color output onto.

## Design Philosophy

- **Theme as a swappable include, not inline color directives.** `kitty.conf` never hardcodes a color; it includes `current-theme.conf` inside a `BEGIN_KITTY_THEME`/`END_KITTY_THEME` marker block, kitty's own convention for `+kitten themes` compatibility — switching themes never means editing the main config.
- **Compositor-delegated blur.** kitty does not implement blur itself on Wayland; `background_blur` is only a *hint* forwarded to the compositor. The frosted-glass look is a joint responsibility of `background_opacity` (kitty) and compositor-side blur (niri).
- **Legacy-safe glyph rendering.** `text_composition_strategy legacy` is set explicitly rather than left at kitty's newer default, because some Wayland compositors mis-render ligatures/glyphs under the newer strategy.

## Key Features

- 16 ANSI colors remapped to the Imperator palette (`Imperator.conf`), with bright variants distinct from their base (e.g. `color1`/`color9` = Plasma Red base/bright).
- Powerline-slanted, left-aligned tab bar; active tab uses full-intensity Golden Signal (`#FFD700`) on near-black, inactive tabs recede to muted amber.
- `shell_integration no-cursor` — disables kitty's shell-integration cursor override so the shell/Vim's own cursor-shape control is authoritative.
- `sync_to_monitor yes` + tuned `repaint_delay`/`input_delay` for low input latency without tearing.
- Font ligature feature flags (`+ss01 +ss02`) enabled specifically, not full ligature mode.

## Configuration Breakdown

| File | Responsibility | Why it exists |
|---|---|---|
| `kitty.conf` | Fonts, cursor, scrollback, mouse, performance tuning, bell, window/tab-bar styling, keybindings, theme include | Main entry point kitty reads directly |
| `Imperator.conf` | Canonical theme definition — background/foreground/cursor/selection/border/tab colors + full 16-color ANSI table | Source of truth for the palette; consumed by `kitty +kitten themes` to regenerate `current-theme.conf` |
| `current-theme.conf` | Active theme, generated/copied from `Imperator.conf` | Decouples "what theme is defined" from "what theme is currently loaded", matching kitty's own theme-switching mechanism |

## Dependencies

- `kitty` ≥ 0.35 (tested on 0.47.1)
- `JetBrainsMonoNL Nerd Font Mono` — primary font, ligatures + glyphs
- A Wayland compositor with blur support (niri, KDE) for the frosted-glass effect to render as intended — without it, the terminal is simply semi-transparent over whatever is behind it

## Usage

Deployed as the kitty config directory. `kitty.conf`'s `include current-theme.conf` line requires `current-theme.conf` to exist at that relative path (`themes/Imperator.conf` copied or symlinked to `current-theme.conf` in the parent directory) — if the whole `kitty/` directory is deployed as a unit (copy or symlink), this resolves automatically.

## Customization

- **Palette**: edit `Imperator.conf` directly, or run `kitty +kitten themes --reload-in=all Imperator` after placing it under `themes/` to regenerate `current-theme.conf` through kitty's own tooling instead of hand-editing.
- **Tab bar style**: `tab_bar_style`, `tab_powerline_style`, and the `*_tab_*` color/font directives in `kitty.conf`.
- **Keybindings**: the `Keybindings` fold in `kitty.conf` — tabs, splits, clipboard, font size, fullscreen.
- **Frosted glass intensity**: `background_opacity` (transparency amount) and `background_blur` (blur hint magnitude) in the `Background` fold.

## Performance Considerations

- `repaint_delay 10` / `input_delay 3` trade a small, deliberate amount of batching for reduced GPU wakeups versus repainting on every single event.
- `background_blur 64` costs compositor-side GPU time, not kitty-side — the actual cost is paid by niri's renderer, not the terminal process.
- `scrollback_lines 10000` is a memory/CPU tradeoff versus kitty's default; large scrollback buffers increase memory held per window but this value is well within normal desktop-RAM budgets.
- `sync_to_monitor yes` avoids tearing at the cost of being bound to the display's refresh rate rather than rendering as fast as possible.

## Notes

- The frosted-glass effect **requires both** a transparent/blurred wallpaper behind the window *and* compositor-side blur configured (niri's `config.kdl`, or KDE window-decoration blur) — enabling only `background_opacity` without compositor blur produces plain transparency, not frost.
- `disable_ligatures never` combined with only two feature flags (`+ss01 +ss02`) means ligatures are allowed but limited to those two OpenType feature sets — not every ligature the font defines is necessarily active.
- This directory is a git submodule (`Impr-Kitty`) with its own remote — changes here should be committed/pushed independently of the parent `Imperator-dotfiles` repo.
