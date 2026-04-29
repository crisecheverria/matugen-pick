# matugen-pick

A 25-line bash wrapper that turns picking a wallpaper into one command on
macOS: browse a folder of images with [fzf](https://github.com/junegunn/fzf)
(kitty image previews), set the pick as your desktop wallpaper via
[desktoppr](https://github.com/scriptingosx/desktoppr), then hand off to
[matugen](https://github.com/InioX/matugen) which prompts for a source color
and regenerates Material You themes for any apps you've templated.

## Demo

<video src="https://github.com/crisecheverria/matugen-pick/raw/main/demo.mp4" controls width="100%"></video>

If the player above does not load, [download or play demo.mp4 directly](demo.mp4).

## Requirements

- macOS (uses `desktoppr` for wallpaper, and the kitty image protocol for previews)
- [kitty](https://sw.kovidgoyal.net/kitty/) — for the `kitten icat` image preview
- [fzf](https://github.com/junegunn/fzf)
- [matugen](https://github.com/InioX/matugen)
- [desktoppr](https://github.com/scriptingosx/desktoppr)

For Neovim integration:

- [matugen.nvim](https://github.com/daedlock/matugen.nvim) — reads the
  `colors.json` matugen produces and rebuilds the colorscheme live whenever
  the file changes.

```sh
brew install fzf desktoppr
cargo install matugen   # or: brew install matugen
```

## Install

```sh
install -m 755 matugen-pick ~/.local/bin/
```

Make sure `~/.local/bin` is on your `PATH`.

## Usage

```sh
matugen-pick                     # browses ~/Documents/backgrounds
matugen-pick ~/Pictures/walls    # browse a different folder
```

`fzf` opens with a thumbnail preview pane. Hit `Enter` on a file:

1. `desktoppr` sets it as your macOS desktop wallpaper.
2. `matugen image` prompts you to pick the source color extracted from the
   image, then runs every template in your matugen config.

## Setting up matugen targets (nvim, kitty, …)

The wrapper itself doesn't know what colors you want recoloured — that's
matugen's job, driven by `~/.config/matugen/config.toml` and the templates
it points at. On macOS the matugen config lives at
`~/Library/Application Support/com.InioX.matugen/`; symlinking that to
`~/.config/matugen/` keeps everything tidy:

```sh
mkdir -p ~/Library/Application\ Support
ln -s ~/.config/matugen "$HOME/Library/Application Support/com.InioX.matugen"
```

A minimal `~/.config/matugen/config.toml` that drives both nvim
(via [matugen.nvim](https://github.com/daedlock/matugen.nvim)) and kitty:

```toml
[config]

[templates.nvim]
input_path = "~/.config/matugen/templates/colors.json"
output_path = "~/.config/matugen/colors.json"

[templates.kitty]
input_path = "~/.config/matugen/templates/kitty.conf"
output_path = "~/.config/kitty/matugen-theme.conf"
post_hook = "for s in /tmp/kitty-*; do kitty @ --to \"unix:$s\" load-config; done"
```

The `post_hook` glob handles kitty's auto-suffixed listen sockets — set
`allow_remote_control socket-only` and `listen_on unix:/tmp/kitty` in
`kitty.conf`, and add `include matugen-theme.conf` after your base theme
include so matugen overrides the parts it knows about.

`templates/colors.json` (read by matugen.nvim) and `templates/kitty.conf`
are plain Tera templates — see the
[matugen template docs](https://github.com/InioX/matugen) for the full
list of available color tokens. The
[matugen.nvim README](https://github.com/daedlock/matugen.nvim) lists the
flat MD3 token names it expects in the JSON.

## License

MIT
